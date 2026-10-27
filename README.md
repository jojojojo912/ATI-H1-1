# API de autenticação com JWT

Exercício didático individual em ASP.NET Core Web API. A API usa credenciais fixas no código, sem banco de dados, e protege a rota de perfil com JWT.

## Requisitos

- .NET 10 SDK
- Visual Studio 2022 atualizado ou Visual Studio Code
- Postman

> A validação de build depende da restauração do pacote `Microsoft.AspNetCore.Authentication.JwtBearer` pelo NuGet. Neste ambiente, o acesso ao `api.nuget.org` foi bloqueado e a solicitação de liberação foi recusada; restaure os pacotes localmente com `dotnet restore` antes de executar.

## Executar

Abra `Atividade 1 API.csproj` no Visual Studio e execute o projeto (`F5`). Ou, no terminal, na pasta do projeto:

```powershell
dotnet restore
dotnet run --urls http://localhost:5000
```

Credenciais didáticas: usuário `aluno`, senha `senha123`. A chave de assinatura fica em `appsettings.json` apenas para facilitar a execução local. Não use essas credenciais ou chave em produção.

## Rotas

### `POST /api/auth/login`

Corpo JSON:

```json
{"usuario":"aluno","senha":"senha123"}
```

Com credenciais corretas, retorna `200 OK` com `accessToken`, `tokenType` e `expiresIn`. Credenciais incorretas retornam `401 Unauthorized`.

### `GET /api/usuario/perfil`

Requer `Authorization: Bearer <token>`. Retorna uma mensagem e o nome do usuário autenticado. Sem token, token inválido ou expirado, retorna `401 Unauthorized`.

## Testes no Postman

1. Importe `postman/Autenticacao JWT.postman_collection.json` (Import > File).
2. Inicie a API em `http://localhost:5000`. A coleção define `baseUrl` como essa URL; ajuste a variável se necessário.
3. Execute as quatro requisições da coleção. A primeira salva o token na variável `jwtToken`, usada automaticamente pelo quarto teste. Cada requisição tem uma asserção que aparece na aba **Test Results**.

Os resultados esperados são: login válido `200` e token; login inválido `401`; perfil sem token `401`; perfil com token válido `200` e mensagem contendo `aluno`. A coleção registra as verificações executáveis; para evidência visual da execução, capture a aba **Test Results** de cada requisição no Postman após executá-las.

## Respostas teóricas

### Exercício 1 — Autenticação e autorização

Autenticação confirma quem está fazendo a solicitação. Por exemplo, ao enviar usuário e senha corretos no login, a API confirma que a pessoa é a conta `aluno`. Autorização verifica o que uma identidade já confirmada pode fazer. Por exemplo, um usuário autenticado sem a permissão de administrador não pode acessar uma rota de administração.

### Exercício 2 — Funcionamento do JWT

O cliente envia usuário e senha ao endpoint de login. A API valida as credenciais e, se estiverem corretas, assina e devolve um JWT com dados necessários para identificar o usuário e metadados como emissor, audiência e validade. Em chamadas posteriores, o cliente envia o token no cabeçalho `Authorization` no formato `Bearer <token>`. `Bearer` indica que a requisição apresenta um token portador como credencial. A API verifica assinatura e validade antes de liberar uma rota protegida. O payload de um JWT assinado é codificado, mas não criptografado por padrão; por isso senhas, chaves e outros segredos não devem ser colocados nele. A assinatura detecta alterações, não esconde o conteúdo.

### Exercício 3 — HTTP 401 e 403

`401 Unauthorized` indica que a requisição não trouxe credenciais válidas para autenticar a identidade, como uma chamada à rota protegida sem token ou com token expirado. `403 Forbidden` indica que a identidade foi autenticada, mas não tem permissão para aquela operação; por exemplo, um usuário autenticado sem papel de administrador tentando acessar uma rota restrita a administradores.
