# Gateway
Este app servirá como um intermediário entre o Frontend e o Backend (Secured Service).
Ele atua como um ponto de entrada único para as requisições dos clientes (Frontend).
O gateway é a peça que orquestra a autenticação,
autorização e comunicação segura entre o cliente (frontend) e os serviços backend.
Ele assegura que apenas usuários autenticados e autorizados possam acessar recursos protegidos
e simplifica a interação com os serviços backend!

## Primeiro Passo
- Crie um Controller para fornecer informações sobre o usuário logado!
```Java
@RestController
public class ProfileController {

    @GetMapping("/userinfo")
    @CrossOrigin("http://localhost:3000")
    public Map<String, Object> userInfo(@AuthenticationPrincipal OidcUser oidcUser, @RegisteredOAuth2AuthorizedClient OAuth2AuthorizedClient authorizedClient) {
        Map<String, Object> attributesMap = new HashMap<>(oidcUser.getAttributes());
        attributesMap.put("id_token", oidcUser.getIdToken().getTokenValue());
        attributesMap.put("access_token", authorizedClient.getAccessToken().getTokenValue());
        attributesMap.put("client_name", authorizedClient.getClientRegistration().getClientId());
        attributesMap.put("user_attributes", oidcUser.getAttributes());
        return attributesMap;
    }
}
```
O método userInfo define um endpoint que retorna informações sobre o usuário autenticado.
Esse endpoint é acessível apenas para clientes autorizados e fornece detalhes sobre a sessão do usuário.

- Este parâmetro fornece acesso ao cliente OAuth2 autorizado, que inclui o access_token usado para autenticação.

```Java
@RegisteredOAuth2AuthorizedClient OAuth2AuthorizedClient authorizedClient
```

- Este parâmetro é utilizado para acessar as informações do usuário autenticado no contexto de OAuth2/OpenID.
O OidcUser contém detalhes sobre o usuário, como atributos e tokens.

```Java
@AuthenticationPrincipal OidcUser oidcUser
```

## Segundo Passo

- Alterar `application.properties` para `application.yaml` e seguir com a seguinte configuração

```yaml
server:
  port: 8000

spring:
  application:
    name: gateway

  # Configuração do cliente OAuth2 para se registrar com o provedor de identidade Azure.

  security:
    oauth2:
      client:
        registration:
          azure:
            client-id: ${CLIENT_ID_AUZURE}
            client-secret: ${CLIENT_SECRET_AZURE}
            scope: openid, profile, email

            redirect-uri: http://localhost:3000/callback  # Necessário ser a mesma cadastrada na Azure
            provider: azure

        # Configuração do provedor de identidade Azure.

        provider:
          azure:
            issuer-uri: ${ISSUER__URI_AZURE} # Mesmo presente no Secured-Service
            authorization-uri: ${AUTH_URI_AZURE}
            token-uri: ${TOKEN_URI_AZURE}
            jwk-set-uri: ${JWK_SET_URI_AZURE}

  cloud:
    gateway:

      # Define as rotas do gateway.

      routes:
        - id: resource
          uri: http://localhost:9000 # URI para onde a requisição será roteada (URI do Secured-Service)
          predicates: # A rota será usada para requisições com o caminho /resource.
            - Path=/resource

          # Filtros aplicados às requisições que passam pela rota.
        
          filters: 

            - name: TokenRelay # Um filtro que repassa o token OAuth2 para o serviço de backend.
              
            # Adiciona cabeçalhos de resposta para permitir CORS

            - AddResponseHeader=Access-Control-Allow-Origin, http://localhost:3000
            - AddResponseHeader=Access-Control-Allow-Methods, GET, POST, PUT, DELETE, OPTIONS
            - AddResponseHeader=Access-Control-Allow-Headers, Authorization, Content-Type

# Logs mais detalhados para acompanhar o processo pelo terminal!

logging:
  level:
    org.springframework.security: DEBUG
    org.springframework.cloud.gateway.filter.factory.TokenRelayGatewayFilterFactory: DEBUG
    org.springframework.cloud.gateway: DEBUG
    org.springframework.security.oauth2: DEBUG
```
## Terceiro passo

- Configurar o nosso `SecurityConfig.java`

```Java

@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {

  // Injeta o repositório de registro de clientes OAuth2, utilizado para gerenciar registros de clientes OAuth2.

  @Autowired
  private ReactiveClientRegistrationRepository clientRegistrationRepository;

  // Injeta o URI do JWK Set (JSON Web Key Set) presente no `application.yaml`.

  @Value("${spring.security.oauth2.client.provider.azure.jwk-set-uri}")
  private String jwkSetUri;

  @Bean
  SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
    http.csrf(ServerHttpSecurity.CsrfSpec::disable).cors(ServerHttpSecurity.CorsSpec::disable);

    // - Permite acesso público ao caminho "/login" e requer autenticação para qualquer outro caminho.

    http.authorizeExchange(conf -> conf
                    .pathMatchers("/login").permitAll()
                    .anyExchange().authenticated())

            // - Define o resolvedor de requisições de autorização.
            // - Redireciona para "/profile" após a autenticação bem-sucedida.

            .oauth2Login(conf -> conf
                    .authorizationRequestResolver(authorizationRequestResolver(clientRegistrationRepository))
                    .authenticationSuccessHandler(new RedirectServerAuthenticationSuccessHandler("http://localhost:3000/profile")))

            // - Define o decodificador JWT para validar tokens de acesso.

            .oauth2ResourceServer(conf -> conf
                    .jwt(jwt -> jwt.jwtDecoder(jwtDecoder())));
    return http.build();
  }

  // Cria um bean para o decodificador de JWTs usando o URI do JWK SET que configuramos acima!.

  @Bean
  public ReactiveJwtDecoder jwtDecoder() {
    return NimbusReactiveJwtDecoder.withJwkSetUri(jwkSetUri).build();
  }

  // - Configura o resolvedor para usar PKCE (Proof Key for Code Exchange) para maior segurança.

  private ServerOAuth2AuthorizationRequestResolver authorizationRequestResolver(ReactiveClientRegistrationRepository clientRegistrationRepository) {
    var authorizationRequestResolver = new DefaultServerOAuth2AuthorizationRequestResolver(clientRegistrationRepository);
    authorizationRequestResolver.setAuthorizationRequestCustomizer(OAuth2AuthorizationRequestCustomizers.withPkce());
    return authorizationRequestResolver;
  }
}
```
---
### Siga para o Frontend
