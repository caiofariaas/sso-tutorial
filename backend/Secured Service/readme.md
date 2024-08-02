# Secured Service

Este app servirá como um Backend Comum, ele será a nossa API

## Primeiro Passo
- Crie um Controller para fornecer um "Recurso Protegido"

```Java
@RestController
public class ResourceController {

    @GetMapping("/resource")
    @CrossOrigin(origins = "http://localhost:3000")
    public String getResourceString(@AuthenticationPrincipal Jwt jwt) {
        String userName = jwt.getClaimAsString("name");

        return "Recurso Protegido Acessado por: " + userName;
    }
    
}
```

- Habilita o CORS para permitir que a aplicação frontend faça requisições para este endpoint.

```Java
  @CrossOrigin(origins = "http://localhost:3000")
  ```
- Injeta o JWT autenticado atual no método. O JWT é utilizado para obter informações sobre o usuário autenticado.

```Java
  @AuthenticationPrincipal Jwt jwt
  ```

## Segundo Passo

- Application.properties
  
```Properties
spring.application.name=secured-service
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://login.microsoftonline.com/{tenant-id}/v2.0
server.port = 9000
```
Utilizamos o issuer-uri para verificar se o Token recebido na requisição realmente foi emitido pelo provedor confiável, e támbem para processar e validar esta informação.
---
### Siga para Gateway
