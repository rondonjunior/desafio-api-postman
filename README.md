# Desafio de Automação de Testes de API

![Testes de API](https://github.com/rondonjunior/desafio-api-postman/actions/workflows/testes-api.yml/badge.svg)

Testes automatizados da API de usuários do [ServeRest](https://serverest.dev), cobrindo todos os endpoints de usuários, a autenticação via token JWT e as principais regras de negócio. Os testes rodam automaticamente a cada commit no GitHub Actions e geram um relatório HTML disponibilizado como artefato.

**Resultado atual:** 25 casos de teste e 63 verificações automatizadas, todas passando.

---

## Tecnologias

| Ferramenta | Para que foi usada |
|---|---|
| **Postman** | Criação das requisições, scripts e testes |
| **Newman** | Execução da collection pelo terminal e na pipeline |
| **newman-reporter-htmlextra** | Geração do relatório HTML |
| **GitHub Actions** | Pipeline de CI que roda os testes a cada push e pull request |
| **Node.js** | Ambiente para rodar o Newman |

---

## Estrutura do projeto

```
desafio-api-postman/
├── .github/workflows/
│   └── testes-api.yml                            # Pipeline do GitHub Actions
├── postman/
│   ├── serverest-usuarios.postman_collection.json   # Requisições e testes
│   └── serverest.postman_environment.json           # URL base da API
├── .gitignore
├── package.json                                  # Dependências e comando de teste
└── README.md
```

A collection está organizada em 6 pastas, que seguem a ordem natural de uso da API:

1. **Cadastrar Usuário (POST)**
2. **Autenticação (Login JWT)**
3. **Listar Usuários (GET)**
4. **Buscar Usuário por ID (GET)**
5. **Atualizar Usuário (PUT)**
6. **Excluir Usuário (DELETE)**

O usuário criado no início é usado nas etapas seguintes e excluído no final, deixando a API limpa após cada execução.

---

## Como rodar os testes

### Pré-requisitos

- [Node.js](https://nodejs.org) versão 20 ou superior
- [Git](https://git-scm.com)

### Passo a passo

```bash
# 1. Clonar o repositório
git clone https://github.com/rondonjunior/desafio-api-postman.git

# 2. Entrar na pasta
cd desafio-api-postman

# 3. Instalar as dependências
npm install

# 4. Rodar os testes
npm test
```

Ao final, o relatório fica disponível em `reports/relatorio-api.html`. É só abrir no navegador.

### Rodando pelo Postman

Também é possível importar os arquivos da pasta `postman/` no Postman (botão **Import**), selecionar o environment **ServeRest** e usar o **Run collection**.

---

## Pipeline de CI

A pipeline roda automaticamente a cada **push** e **pull request**, e também pode ser executada manualmente pela aba **Actions**. As etapas são:

1. Baixa o código do repositório
2. Instala o Node.js
3. Instala as dependências
4. Roda os testes com Newman
5. Publica o relatório HTML como **artefato** (mesmo se algum teste falhar)

Para ver o relatório: aba **Actions** → clicar em uma execução → seção **Artifacts** → baixar `relatorio-testes-api`.

---

## Casos de teste cobertos

### 01 - Cadastrar Usuário (POST /usuarios)

| ID | Cenário | Status esperado |
|---|---|---|
| CT01 | Cadastrar usuário administrador com sucesso | 201 |
| CT02 | Não cadastrar com email já utilizado | 400 |
| CT03 | Não cadastrar sem o campo `nome` | 400 |
| CT04 | Não cadastrar sem o campo `email` | 400 |
| CT05 | Não cadastrar sem o campo `password` | 400 |
| CT06 | Não cadastrar sem o campo `administrador` | 400 |
| CT07 | Não cadastrar com email em formato inválido | 400 |

### 02 - Autenticação JWT (POST /login)

| ID | Cenário | Status esperado |
|---|---|---|
| CT08 | Login com credenciais válidas, validando o conteúdo do token | 200 |
| CT09 | Não logar com senha incorreta | 401 |
| CT10 | Não logar com email não cadastrado | 401 |
| CT11 | Não logar sem email e senha | 400 |
| CT12 | Acessar rota protegida com token válido | 201 |
| CT13 | Não acessar rota protegida sem token | 401 |

### 03 - Listar Usuários (GET /usuarios)

| ID | Cenário | Status esperado |
|---|---|---|
| CT14 | Listar todos os usuários, validando o contrato (JSON Schema) | 200 |
| CT15 | Filtrar usuário pelo email | 200 |
| CT16 | Retornar lista vazia para email inexistente | 200 |

### 04 - Buscar Usuário por ID (GET /usuarios/{id})

| ID | Cenário | Status esperado |
|---|---|---|
| CT17 | Buscar usuário pelo ID, validando contrato e todos os dados | 200 |
| CT18 | Não encontrar usuário com ID inexistente | 400 |

### 05 - Atualizar Usuário (PUT /usuarios/{id})

| ID | Cenário | Status esperado |
|---|---|---|
| CT19 | Atualizar os dados e confirmar a alteração com nova consulta | 200 |
| CT20 | Criar usuário ao atualizar ID inexistente (comportamento upsert) | 201 |
| CT21 | Não atualizar para email já utilizado por outro usuário | 400 |
| CT22 | Não atualizar sem campo obrigatório | 400 |

### 06 - Excluir Usuário (DELETE /usuarios/{id})

| ID | Cenário | Status esperado |
|---|---|---|
| CT23 | Informar que nenhum registro foi excluído para ID inexistente | 200 |
| CT24 | Não excluir usuário com carrinho cadastrado (regra de negócio) | 400 |
| CT25 | Excluir o usuário e confirmar que ele não existe mais | 200 |

### Resumo da cobertura

| Endpoint | Sucesso | Erros de validação | Regra de negócio | Contrato |
|---|:-:|:-:|:-:|:-:|
| GET /usuarios | ✅ | n/a | ✅ filtros | ✅ |
| POST /usuarios | ✅ | ✅ | ✅ email único | ✅ |
| GET /usuarios/{id} | ✅ | ✅ | n/a | ✅ |
| PUT /usuarios/{id} | ✅ | ✅ | ✅ upsert e email único | n/a |
| DELETE /usuarios/{id} | ✅ | ✅ | ✅ bloqueio com carrinho | n/a |
| POST /login (JWT) | ✅ | ✅ | ✅ rota protegida | ✅ |

---

## Decisões técnicas

**Dados dinâmicos.** O email do usuário é gerado a cada execução com base na data e hora, então os testes nunca conflitam com dados antigos e podem rodar quantas vezes for preciso.

**Autenticação JWT.** No ServeRest, as rotas de `/usuarios` são públicas. Para validar o requisito de autenticação de forma real, o token foi testado no login (formato, conteúdo do payload e validade) e em uma rota que exige token (`POST /produtos`), cobrindo acesso com e sem token.

**Validação além do status.** Nos cenários de sucesso, a alteração e a exclusão são confirmadas com uma nova consulta à API, em vez de confiar apenas na mensagem de retorno.

**Limpeza dos dados.** Tudo que os testes criam (usuários, produtos e carrinhos) é excluído ao final, deixando a API como estava.

**Limite de requisições.** A API aceita no máximo 100 requisições por minuto. A suíte completa faz 33 requisições em cerca de 10 segundos, bem abaixo do limite.

**Pontos de atenção encontrados na API.** Alguns comportamentos fogem do padrão REST e estão documentados nos testes: buscar um ID inexistente retorna `400` em vez de `404` (CT18), excluir um ID inexistente retorna `200` (CT23) e atualizar um ID inexistente cria um novo registro (CT20).

---

## Autor

**Rondon Júnior** · QA