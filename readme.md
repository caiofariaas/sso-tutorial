# Tutorial de Single Sign-On (Bosch - SSO) com Spring Boot e React

Este projeto é um tutorial de implementação de Single Sign-On (SSO) usando Spring Boot e React. O objetivo é criar uma aplicação web que autentica usuários através de um provedor de identidade, no caso, o Azure Active Directory (Azure AD) que é o provedor utilizado pela Bosch!

---

## OAuth2 e OpenID

Para este projeto, utilizaremos alguns conceitos de OAuth2 e OpenID. é necessário que saiba mais sobre esses padrões de autenticação e autorização para que continue com o projeto, para isso, recomendamos que veja os seguintes vídeos: 

- [OAuth2.0 e OpenID](https://www.youtube.com/watch?v=68azMcqPpyo)
- [Auth Server + OpenID](https://www.youtube.com/watch?v=hgLKOPHfuis)

---

# Portal Azure 

Primeiro vamos fazer o registro do nosso aplicativo no [Portal da Azure](https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)

- Após acessar o link, devemos clicar no botão "+ Novo registro", localizado no canto superior esquerdo da tela.

- Após a tela de registrar um aplicativo aparecer, deve-se colocar as seguintes informações:
<img src="https://raw.githubusercontent.com/caiofariaas/sso-tutorial/master/assets/novo_aplicativo.png" alt="Diagrama" width="600"/>

Com o aplicativo criado, temos que configurar algumas coisas para que o acesso seja concedido!

- O próximo passo é entrar no seu aplicativo e navegar até a aba "proprietários". No canto superior esquerdo você pode localizar o botão "+ Adicionar proprietários", clique nele e, na aba que abriu, há uma barra de pesquisa, nela adicione algum instrutor como proprietário.  
⚠️ **Lembre-se sempre de pedir autorização pro instrutor em questão**

A próxima parte é **opcional**, serve apenas em casos de permissões diferentes para os usuários.

- Para adicionar essas funções você deve navegar até "funções de aplicativo". Dentro de "funções de aplicativo" há o botão "+ Criar uma função de aplicativo", ele fica localizado no mesmo lugar dos demais. Na aba que abriu, você deve preencher com as seguintes informações:  
PS: O "Nome de exibição" e "Valor" são coisas que você pode alterar para o nome que desejar!
<img src="https://raw.githubusercontent.com/caiofariaas/sso-tutorial/master/assets/adm.png" alt="Diagrama" width="600"/>   

- Após preencher as informações, devemos adicionar um usuário como admin (ou com o nome da role que você acabou de criar)
Para isso, devemos ir até a barra de pesquisa que fica localizada no topo de página e procurar por "Aplicativos Empresariais". Após acessar essa página, procuramos nosso aplicativo pela barra de pesquisa menor que fica acima da lista de aplicativos. Assim que localizar, clique no seu aplicativo e vá até a aba "Usuários e grupos", dentro dela clique no botão "+ Adicionar um usuário ou um grupo". Assim que a página abrir, primeiro devemos selecionar um usuário para atribuir aquela função, então clicamos em "Nenhum Selecionado", logo abaixo de "Usuários e grupos" e procuramos pelo usuário desejado (Por exemplo, da ETS). Após selecionarmos o usuário, selecionamos a função dele, clicamos novamente em "Nenhum selecionado", abaixo de "Selecionar uma função" e selecionamos a função criada anteriormente, no caso exemplificado, "Admin".

Agora voltamos para uma parte **obrigatória**, ainda dentro de "Aplicativos Empresariais".

- Navegue até a aba "Proprietários" e clique em "+ Adicionar". Após isso, procure na barra de pesquisa dentro da aba aberta o mesmo instrutor que você adicionou anteriormente.

- Após o passo anterior ser concluído, voltamos para "Registros de Aplicativo", podemos seguir os mesmos passos para achar "Aplicativos Empresariais", e achamos nosso aplicativo novamente, pela barra de pesquisa. Após clicar em seu aplicativo, vá até a aba "Autenticação" e clique em "+ Adicionar uma plataforma". Após abrir a página de autenticação, clique no botão "+ Adicionar uma plataforma". O próximo passo após a aba abrir é selecionar "Web", após isso você deve adicionar uma URI de redirecionamento, por exemplo `http://localhost:3000/callback`, lembre-se de que essa URI vai ser utilizada durante o desenvolvimento da aplicação. Após adicionar a URI, fique na mesma página de "Autenticação" e role até encontrar as seguintes caixas de seleção: "Tokens de acesso" e "Tokens de ID", selecione ambas as caixas e salve novamente.

- O penúltimo passo é adicionar um client secret! Para isso, vá até a aba "Certificados e segredos", clique no botão "+ Novo segredo do cliente". Após a aba abrir, adicione na descrição o que quiser e selecione o tempo desejado de expiração. Após adicionar a secret você verá ela na lista de segredos, localize o atributo "Valor" dentro dela e copie esse valor. É importante ressaltar que esse valor não será mostrado novamente, portanto guarde esse valor em algum local. Se por acaso você perder o valor, você pode deletar a chave e fazer outra, sem problemas.

- Finalmente, o último passo. Para concluir toda a configuração, você deve ir até a aba "Permissões de APIs" e, dentro dela, localize a parte "Permissões configuradas". Dentro de "Permissões configuradas", se tiver alguma permissão, tire ela e deixe sem nada. Agora vamos adicionar as permissões necessárias, clique em "+ Adicionar uma permissão". Após a aba abrir selecione Microsoft Graph e selecione em seguida "Permissões delegadas".  Role um pouco para baixo e localize "Permissões de OpenId", dentro dessa parte selecione "email", "openid" e "profile" e adicione essas permissões.


Para prosseguir com o projeto é importante saber onde conseguir algumas informações:
- Acesse visão geral, dentro de seu aplicativo. Na parte de cima você verá o título "Fundamentos", é dentro dessa parte que as informações estarão.
- O **client id** é o código logo após "ID do aplicativo (cliente)"
- O **tenant id** é o código logo após "ID do diretório (locatário)"

Você usará esses códigos para "completar" alguns links. Para pegar esses links você deve acessar em "Visão geral" a parte de "Pontos de extremidade", que fica acima de "Fundamentos" que vimos anteriormente. Dentro de "Pontos de extremidade" você deve pegar os seguintes links:
- Ponto de extremidade de autorização OAuth 2.0 (v2)
- Ponto de extremidade do token OAuth 2.0 (v2)

Tem alguns links que não ficam dentro da azure, passaremos eles pra vocês apenas completarem com a informação necessária.
- ISSUER URI: https://login.microsoftonline.com/{tenant_id}/v2.0
- JWK SET URI: https://login.microsoftonline.com/{tenant_id}/discovery/v2.0/keys

---

### Siga para a pasta "Backend"
