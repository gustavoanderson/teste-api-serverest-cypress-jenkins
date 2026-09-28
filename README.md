<div align="center">

# 🔁 ServeRest · Testes de API com Cypress e Pipeline Jenkins

**Automatizei os testes de usuários, produtos e login de uma API REST, com validação de contrato e execução contínua num pipeline Jenkins**

![Cypress](https://img.shields.io/badge/Cypress-13-17202C?style=for-the-badge&logo=cypress&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-Pipeline-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Joi](https://img.shields.io/badge/Joi-contrato-0969da?style=for-the-badge)
![JavaScript](https://img.shields.io/badge/JavaScript-ES6-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Cenários](https://img.shields.io/badge/cen%C3%A1rios-13-2ea44f?style=for-the-badge)
![CRUD](https://img.shields.io/badge/GET%20%C2%B7%20POST%20%C2%B7%20PUT%20%C2%B7%20DELETE-4%2F4-2ea44f?style=for-the-badge)

🇧🇷 [Português](#-português) · 🇺🇸 [English](#-english)

</div>

---

## 🇧🇷 Português

### 🎯 Objetivo

Depois de explorar a API ServeRest no [Postman](https://github.com/gustavoanderson/postman-serverest), levei os testes para código e dei o passo seguinte: **fazer a suíte rodar sozinha num pipeline**, sem ninguém precisar abrir o Cypress na mão.

O projeto teve três frentes:

1. **Testes funcionais** do CRUD de usuários e de produtos, e do login
2. **Validação de contrato** da resposta com Joi, para detectar mudança na estrutura da API
3. **Integração contínua** com Jenkins: clonar, instalar, subir a API e rodar os testes

### 🧭 Estratégia

#### Pipeline

Configurei um `Jenkinsfile` declarativo com três estágios. O ponto-chave foi o `start-server-and-test`: ele sobe a ServeRest, espera a API responder e só então dispara o Cypress, sem nenhum passo manual.

```mermaid
flowchart LR
    A["📥 Clonar<br/>repositório"] --> B["📦 Instalar<br/>dependências<br/><code>npm install</code>"]
    B --> C["🚀 Subir ServeRest<br/><code>start-server-and-test</code>"]
    C --> D{"API respondendo<br/>em :3000?"}
    D -- Sim --> E["🧪 Rodar Cypress<br/>headless"]
    E --> F(["📊 Resultado<br/>no Jenkins"])

    style A fill:#57606a,color:#fff,stroke:#57606a
    style B fill:#0969da,color:#fff,stroke:#0969da
    style C fill:#8250df,color:#fff,stroke:#8250df
    style E fill:#1a7f37,color:#fff,stroke:#1a7f37
    style F fill:#D24939,color:#fff,stroke:#D24939
```

Ajustei o pipeline para rodar num **agente Windows**, trocando os comandos `sh` por `bat`.

#### Testes independentes

Nos cenários de edição e exclusão, crio o registro dentro do próprio teste com comandos customizados, pego o `_id` que a API devolve e opero em cima dele:

```mermaid
sequenceDiagram
    participant T as Teste
    participant API as ServeRest
    T->>API: POST /login
    API-->>T: authorization (token)
    T->>API: POST /produtos + token (nome aleatório)
    API-->>T: 201 · _id
    T->>API: DELETE /produtos/{_id} + token
    API-->>T: 200 · Registro excluído com sucesso
```

### 🧪 Cobertura

```mermaid
%%{init: {"themeVariables": {"pieOpacity": "1", "pieStrokeColor": "#ffffff", "pieStrokeWidth": "2px", "pieOuterStrokeColor": "#8c959f", "pieSectionTextColor": "#ffffff", "pieSectionTextSize": "15px", "pieTitleTextColor": "#57606a", "pieLegendTextColor": "#57606a", "pie1": "#1a7f37", "pie2": "#0969da", "pie3": "#8250df", "pie4": "#bf3989", "pie5": "#9a6700", "pie6": "#cf222e", "pie7": "#1b7c83", "pie8": "#57606a"}}}%%
pie showData
    title Cenários por funcionalidade (13 no total)
    "Produtos" : 6
    "Usuários" : 6
    "Login" : 1
```

| Funcionalidade | Cenário | Método | Tipo |
|---|---|:-:|:-:|
| 👤 Usuários | Validar contrato da listagem | `GET` | 📐 Contrato |
| 👤 Usuários | Listar usuários | `GET` | ✅ Positivo |
| 👤 Usuários | Cadastrar usuário | `POST` | ✅ Positivo |
| 👤 Usuários | Cadastrar com e-mail já usado | `POST` | ⚠️ Negativo |
| 👤 Usuários | Editar usuário criado no teste | `PUT` | ✅ Positivo |
| 👤 Usuários | Excluir usuário criado no teste | `DELETE` | ✅ Positivo |
| 📦 Produtos | Listar produtos **em menos de 20 ms** | `GET` | ⏱️ Performance |
| 📦 Produtos | Cadastrar produto | `POST` | ✅ Positivo |
| 📦 Produtos | Cadastrar produto com nome repetido | `POST` | ⚠️ Negativo |
| 📦 Produtos | Editar produto existente | `PUT` | ✅ Positivo |
| 📦 Produtos | Editar produto criado no teste | `PUT` | ✅ Positivo |
| 📦 Produtos | Excluir produto criado no teste | `DELETE` | ✅ Positivo |
| 🔐 Login | Login com sucesso | `POST` | ✅ Positivo |

```mermaid
%%{init: {"themeVariables": {"pieOpacity": "1", "pieStrokeColor": "#ffffff", "pieStrokeWidth": "2px", "pieOuterStrokeColor": "#8c959f", "pieSectionTextColor": "#ffffff", "pieSectionTextSize": "15px", "pieTitleTextColor": "#57606a", "pieLegendTextColor": "#57606a", "pie1": "#1a7f37", "pie2": "#0969da", "pie3": "#8250df", "pie4": "#bf3989", "pie5": "#9a6700", "pie6": "#cf222e", "pie7": "#1b7c83", "pie8": "#57606a"}}}%%
pie showData
    title Tipos de teste
    "Funcional positivo" : 9
    "Negativo" : 2
    "Contrato" : 1
    "Performance" : 1
```

### 🧰 Técnicas que apliquei

| Técnica | Implementação |
|---|---|
| Validação de contrato | Schema **Joi** aplicado à resposta com `validateAsync` |
| Asserção de performance | `expect(response.duration).to.be.lessThan(20)` |
| Autenticação reutilizável | `cy.token()` no `before` da suíte de produtos |
| Comandos customizados | `cy.token()`, `cy.cadastrarProduto()` e `cy.cadastrarUsuario()` |
| Massa aleatória | Nomes e e-mails com `Math.random()` para não colidir |
| Testes negativos | `failOnStatusCode: false` para validar o erro 400 |
| Orquestração de ambiente | `start-server-and-test` sobe a API antes da suíte |
| CI | `Jenkinsfile` declarativo com 3 estágios |

### 📈 Resultados

- **Automatizei 13 cenários** cobrindo os quatro métodos HTTP nos recursos de usuários e produtos, além do login.
- **Coloquei a suíte num pipeline Jenkins**, que clona o repositório, instala as dependências, sobe a API e roda os testes de ponta a ponta.
- **Fui além do status code:** a suíte valida o contrato da resposta com Joi e o tempo de resposta da listagem.
- **Protegi regras de negócio:** e-mail duplicado e produto com nome repetido precisam ser recusados, e os testes negativos garantem isso.

### 🚀 Onde esse trabalho se aplica

- **Qualidade contínua:** com o pipeline, os testes rodam a cada alteração, sem depender de alguém lembrar de executar.
- **Proteção contra quebra de integração:** a validação de contrato avisa quando a API muda a estrutura da resposta e o front deixaria de funcionar.
- **SLA de performance:** a asserção de tempo de resposta vira um alarme simples de degradação.
- **Portável para outras ferramentas de CI:** o mesmo fluxo (instalar → subir API → testar) se aplica a GitHub Actions, GitLab CI ou Azure DevOps.

### ▶️ Como executar

**Pré-requisito:** Node.js.

```bash
npm install

# sobe a ServeRest e roda os testes em sequência (mesmo comando do Jenkins)
npm run cy:run-ci

# ou, em dois terminais
npm start          # ServeRest em http://localhost:3000
npx cypress open   # modo interativo
```

**No Jenkins:** crie um job do tipo *Pipeline* apontando para este repositório. O `Jenkinsfile` na raiz define os estágios.

### 📁 Estrutura

```
├── Jenkinsfile                    # pipeline declarativo
├── cypress/
│   ├── contracts/
│   │   └── produtos.contract.js   # schema Joi
│   ├── e2e/
│   │   ├── exercicio-api.cy.js    # usuários
│   │   ├── login.cy.js
│   │   └── produtos.cy.js
│   └── support/commands.js        # token · cadastrarProduto · cadastrarUsuario
└── package.json                   # scripts cy:run-ci e start
```

---

## 🇺🇸 English

### 🎯 Goal

After exploring the ServeRest API in [Postman](https://github.com/gustavoanderson/postman-serverest), I moved the tests into code and took the next step: **making the suite run on its own in a pipeline**.

The project had three fronts: **functional tests** for users, products and login; **contract validation** with Joi; and **continuous integration** with Jenkins.

### 🧭 Strategy

```mermaid
flowchart LR
    A["📥 Clone<br/>repository"] --> B["📦 Install<br/>dependencies<br/><code>npm install</code>"]
    B --> C["🚀 Start ServeRest<br/><code>start-server-and-test</code>"]
    C --> D{"API up<br/>on :3000?"}
    D -- Yes --> E["🧪 Run Cypress<br/>headless"]
    E --> F(["📊 Result<br/>in Jenkins"])

    style A fill:#57606a,color:#fff,stroke:#57606a
    style B fill:#0969da,color:#fff,stroke:#0969da
    style C fill:#8250df,color:#fff,stroke:#8250df
    style E fill:#1a7f37,color:#fff,stroke:#1a7f37
    style F fill:#D24939,color:#fff,stroke:#D24939
```

I wrote a declarative `Jenkinsfile` with three stages. `start-server-and-test` starts ServeRest, waits for the API and only then runs Cypress. I also adapted the pipeline to a **Windows agent** (`bat` instead of `sh`).

For edit and delete scenarios, I create the record inside the test through custom commands, take the returned `_id` and operate on it, so tests don't depend on each other.

### 🧪 Coverage

```mermaid
%%{init: {"themeVariables": {"pieOpacity": "1", "pieStrokeColor": "#ffffff", "pieStrokeWidth": "2px", "pieOuterStrokeColor": "#8c959f", "pieSectionTextColor": "#ffffff", "pieSectionTextSize": "15px", "pieTitleTextColor": "#57606a", "pieLegendTextColor": "#57606a", "pie1": "#1a7f37", "pie2": "#0969da", "pie3": "#8250df", "pie4": "#bf3989", "pie5": "#9a6700", "pie6": "#cf222e", "pie7": "#1b7c83", "pie8": "#57606a"}}}%%
pie showData
    title Scenarios per feature (13 total)
    "Products" : 6
    "Users" : 6
    "Login" : 1
```

```mermaid
%%{init: {"themeVariables": {"pieOpacity": "1", "pieStrokeColor": "#ffffff", "pieStrokeWidth": "2px", "pieOuterStrokeColor": "#8c959f", "pieSectionTextColor": "#ffffff", "pieSectionTextSize": "15px", "pieTitleTextColor": "#57606a", "pieLegendTextColor": "#57606a", "pie1": "#1a7f37", "pie2": "#0969da", "pie3": "#8250df", "pie4": "#bf3989", "pie5": "#9a6700", "pie6": "#cf222e", "pie7": "#1b7c83", "pie8": "#57606a"}}}%%
pie showData
    title Test types
    "Positive functional" : 9
    "Negative" : 2
    "Contract" : 1
    "Performance" : 1
```

### 📈 Results

- **I automated 13 scenarios** covering all four HTTP methods on users and products, plus login.
- **I put the suite in a Jenkins pipeline** that clones, installs, starts the API and runs the tests end to end.
- **I went beyond status codes:** the suite validates the response contract with Joi and the listing response time (< 20 ms).
- **I protected business rules:** duplicate emails and duplicate product names must be rejected.

### 🚀 Where this applies

- **Continuous quality:** tests run on every change, without anyone having to remember.
- **Integration safety:** contract validation flags when the API changes its response structure.
- **Performance SLA:** the response-time assertion works as a simple degradation alarm.
- **Portable CI:** the same flow (install → start API → test) fits GitHub Actions, GitLab CI or Azure DevOps.

### ▶️ How to run

```bash
npm install
npm run cy:run-ci   # starts ServeRest and runs the tests (same as Jenkins)
```

---

<div align="center">

Feito por **Gustavo Anderson** · QA Engineer
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/gustavo-anderson)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/gustavoanderson)

</div>
