# Definindo um *client* no VTEX IO

## Introdução
Já foi visto como criar um _resolver_ GraphQL; agora o desenvolvimento da funcionalidade será continuado. Comumente, a implementação de uma funcionalidade em uma *app* requer a **comunicação com outros clientes**, sejam externos ou internos (outras *apps* VTEX). 

Para realizar comunicação com outros clientes, é necessário **criar um *client***. Um *client* é uma entidade que extende uma definição genérica exportada [`node-vtex-api`](https://github.com/vtex/node-vtex-api). Seu propósito é facilitar e fornecer auditoria de requisições com outras APIs.

Você pode ver um exemplo de um *client* criado para se comunicar com um serviço externo na app [`service-example`](https://github.com/vtex-apps/service-example/blob/ffd7a86f928f9931a9353215eebb764cb3150695/node/clients/status.ts).

Além de **External *Clients***, como este do exemplo, você pode criar: 
- **[App Clients](https://github.com/vtex/node-vtex-api/blob/d273aac28702a5bad6ebac4df9ddee69aba61350/src/clients/apps/AppClient.ts)**, para comunicação HTTP com outras apps dentro da conta VTEX IO;
- **[App GraphQL Clients](https://github.com/vtex/node-vtex-api/blob/d273aac28702a5bad6ebac4df9ddee69aba61350/src/clients/apps/AppGraphQLClient.ts)**, para comunicação através de GraphQL com outras apps também do IO; 
- **[Infra Clients](https://github.com/vtex/node-vtex-api/blob/d273aac28702a5bad6ebac4df9ddee69aba61350/src/clients/infra/InfraClient.ts)** para comunicação com serviços de Infra do VTEX IO.

Após a criação de um *client*, é necessário **adicioná-lo na exportação do Service**. Depois disso, os *clients* estarão disponíveis para acesso na assinatura da função de requisição dos *handlers* através de: 
```
ctx.clients.{{your_client}}
```

## Atividade

### Criando o *client*
1. Crie um arquivo em `node/clients` chamado `giphy.ts`.
2. A partir do [*client* de exemplo](https://github.com/vtex-apps/service-example/blob/ffd7a86f928f9931a9353215eebb764cb3150695/node/clients/status.ts), crie um `GiphyClient` que se comunica com a API do Giphy na URL https://api.giphy.com/v1/gifs/
3. O *client* precisa ter apenas um método chamado `translate` que aceita um `term: string` e retornará uma URL de GIF. Este método deverá chamar o _endpoint_ [translate](https://developers.giphy.com/docs/api/endpoint#translate) da API.  
**OBS.:** Use a `api_key` `dp2scGnUcDee5yLRI1qJMTRTAAJey9Tl` para testar seu _client_.
4. Após criar (e exportar) o *client* em `giphy.ts`, em `node/clients/index.ts` importe `Giphy from './giphy'` e adicione na classe `Clients`:
    ```
    public get giphy() {
        return this.getOrSet('giphy', Giphy)
    }
    ```

### Alterando o *resolver*

1. Agora, voltando ao *resolver*, é possível utilizar `ctx.giphy.translate` para finalizar a implementação da funcionalidade. Retorne a chamada deste método, informando o termo passado como parâmetro para o _resolver_. Para isso, volte ao arquivo `giphy.ts` e modifique a função, que irá utilizar o método `translateGif`. Dessa forma, define-se o *resolver*:
    ```ts
    // node/resolvers/giphy.ts
    export const gif = async (
    _: any,
    { term }: { term: string },
    { clients: { giphy }}: Context
    ) => giphy.translateGif(term)
    ```
    É possível ver que a estrutura é bem semelhante ao esqueleto de *resolver* mostrado anteriormente, há `parent`, `args` e, por fim, `context`. Como já foi adicionado o *resolver* ao arquivo `node/resolver/index.ts`, não é preciso alterá-lo novamente.

6. Adicione no arquivo `manifest.json` uma *policy* para acessar URL externa:
    ```json
    {
        ...
        "policies": [
            {
            "name": "outbound-access",
            "attrs": {
                "host": "api.giphy.com",
                "path": "*"
            }
            }
        ],
        ...
    }
    ```

7. Teste no _GraphiQL_ sua modificação

    Para testar o resultado, basta fazer uma query no GraphiQL e colocar no campo de *query variables* a string que você deseja utilizar para pesquisar um GIF. O resultado esperado é que você receberá uma resposta com o URL de um GIF.

    No nosso exemplo, definimos uma *query* da seguinte forma:
    ```graphql
    query buscaGif ($term: String) {
        gif(term:$term)
    }
    ```
    Além disso, no campo de *query variables*, definimos `term` como *cats*:
    ```json
    {
        "term": "cats"
    }
    ```
    E o nosso resultado foi:
    ```json
    {
        "data": {
            "gif": "https://media2.giphy.com/media/3o72EX5QZ9N9d51dqo/giphy.gif?cid=96678fa42d14d68f9c3ebdfaff64b84de51f012598e0a2e9&rid=giphy.gif"
        }
    }
    ```
    <img src="https://media2.giphy.com/media/3o72EX5QZ9N9d51dqo/giphy.gif?cid=96678fa42d14d68f9c3ebdfaff64b84de51f012598e0a2e9&rid=giphy.gif" width="600" height="320"/>


