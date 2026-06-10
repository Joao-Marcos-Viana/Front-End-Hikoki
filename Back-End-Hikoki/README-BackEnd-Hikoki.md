# Back-End Hikoki — VDM TSEA

API REST do sistema de Visualização de Desenhos de Máquinas (VDM), desenvolvida em Java com Spring Boot. Responsável por autenticação, gerenciamento de usuários, peças/desenhos técnicos, documentos de clientes e armazenamento de arquivos CAD.

---

## Tecnologias

| Tecnologia | Versão |
|---|---|
| Java | 17 |
| Spring Boot | 3.2.5 |
| Spring Security | — |
| Spring Data JPA | — |
| PostgreSQL | — |
| Lombok | — |
| Maven | — |

---

## Pré-requisitos

- Java 17+
- Maven 3.8+
- PostgreSQL rodando localmente na porta `5432`

---

## Configuração do Banco de Dados

Crie um banco de dados no PostgreSQL chamado `hikokiDb`:

```sql
CREATE DATABASE hikokiDb;
```

As credenciais padrão configuradas em `src/main/resources/application.properties` são:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/hikokiDb
spring.datasource.username=postgres
spring.datasource.password=1234

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
server.port=8085
```

> Ajuste `username` e `password` conforme o seu ambiente local. O Hibernate criará as tabelas automaticamente na primeira execução (`ddl-auto=update`).

---

## Como Executar

```bash
# Clone ou extraia o projeto
cd Back-End-Hikoki

# Execute com Maven Wrapper (não precisa ter o Maven instalado globalmente)
./mvnw spring-boot:run

# Windows
mvnw.cmd spring-boot:run
```

A API estará disponível em: **http://localhost:8085**

---

## Estrutura do Projeto

```
Back-End-Hikoki/
├── src/main/java/com/testeSistemas/hikoki/
│   ├── Controller/
│   │   ├── DrawingIntegrationController.java  ← endpoints principais (login, desenhos, upload)
│   │   ├── PecaController.java                ← CRUD de peças
│   │   ├── UserController.java                ← CRUD de usuários
│   │   └── DocClienteController.java          ← CRUD de clientes
│   ├── Entity/
│   │   ├── PecaEntity.java     ← tabela: pecas
│   │   ├── UserEntity.java     ← tabela: users
│   │   └── DocCliente.java     ← tabela: docCliente
│   ├── Repository/
│   │   ├── PecaRepository.java
│   │   ├── UserRepository.java
│   │   └── DocClienteRepository.java
│   ├── Service/
│   │   ├── PecaService.java
│   │   ├── UserService.java
│   │   └── DocClienteService.java
│   ├── Security/
│   │   └── SecurityConfig.java  ← configuração do Spring Security + BCrypt
│   └── HikokiApplication.java   ← ponto de entrada da aplicação
├── src/main/resources/
│   └── application.properties
└── pom.xml
```

---

## Endpoints da API

### Autenticação

| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/api/login` | Autenticação por perfil (funcionário / supervisor) |

**Body do login:**
```json
{
  "perfil": "supervisor",
  "matricula": "admin",
  "senha": "admin1234"
}
```

**Credenciais padrão:**

| Perfil | Matrícula | Senha | Acesso |
|---|---|---|---|
| Supervisor | `admin` | `admin1234` | Todos os setores + gerenciamento |
| Funcionário | `00123` | `1234` | Apenas setor Montagem (visualização) |

---

### Desenhos / Peças

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/api/setores` | Lista todos os setores disponíveis |
| GET | `/api/desenhos` | Lista todos os desenhos (agrupados por nome e versão) |
| GET | `/api/desenhos?setor={setor}` | Filtra desenhos por setor |
| POST | `/api/desenhos/upload` | Faz upload de nova versão de desenho (multipart/form-data) |
| GET | `/api/arquivo/{filename}` | Serve o arquivo CAD/imagem para o front-end |
| DELETE | `/api/desenhos/{id}` | Exclui um desenho e todas as suas versões |

**Parâmetros do upload (`multipart/form-data`):**
- `nome` — nome do desenho
- `setor` — setor ao qual pertence
- `revisao` — número da revisão (ex: `1.0`, `2.0`)
- `descricao` *(opcional)* — descrição da revisão
- `autor` *(opcional, padrão: "Supervisor")* — autor da revisão
- `arquivo` — arquivo binário (PNG, JPG, SVG, PDF, DWG, DXF)

---

### Peças (CRUD direto)

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/peca` | Lista todas as peças |
| GET | `/peca/{id}` | Busca peça por ID |
| POST | `/peca/novaPeca` | Cria nova peça |
| PUT | `/peca/{id}` | Atualiza peça |
| DELETE | `/peca/{id}` | Remove peça |

---

### Usuários

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/users` | Lista todos os usuários |
| POST | `/users/novoUser` | Cria novo usuário |
| PUT | `/users/{id}` | Atualiza usuário |
| DELETE | `/users/{id}` | Remove usuário |

> As senhas dos usuários são armazenadas com hash BCrypt.

---

### Clientes (DocCliente)

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/docCliente` | Lista todos os clientes |
| POST | `/docCliente/novoDocCliente` | Cadastra novo cliente |
| DELETE | `/docCliente/{id}` | Remove cliente |

---

## Armazenamento de Arquivos

Os arquivos enviados via upload são salvos fisicamente na pasta `uploads/` dentro do diretório de execução da aplicação. O nome do arquivo gerado segue o padrão:

```
{nomePeca}_rev{revisao}.{extensao}
```

Exemplo: `Suporte_Lateral_rev1-0.pdf`

---

## Segurança

- O Spring Security está configurado com todas as rotas abertas (`permitAll`) para facilitar o desenvolvimento e uso em rede interna.
- As senhas dos usuários cadastrados no banco são encriptadas com **BCryptPasswordEncoder**.
- CSRF está desabilitado.
- CORS liberado para todas as origens (`*`), permitindo acesso pelo front-end local.

> Para produção, recomenda-se restringir as origens do CORS e implementar autenticação baseada em token (JWT).

---

## Modelos de Dados

### PecaEntity (`tabela: pecas`)
| Campo | Tipo | Descrição |
|---|---|---|
| idPeca | Integer | Identificador (PK) |
| nomePeca | String | Nome do desenho |
| dataCriacao | Date | Data do upload |
| versao | Double | Número da revisão |
| urlDoc | String | Nome do arquivo físico salvo |
| setor | String | Setor responsável |
| descricao | String | Descrição da revisão |
| autor | String | Autor da revisão |
| cliente | DocCliente | Relação com cliente (FK) |

### UserEntity (`tabela: users`)
| Campo | Tipo | Descrição |
|---|---|---|
| idUser | Integer | Identificador / matrícula |
| key | String | Senha (hash BCrypt) |
| nomeUser | String | Nome completo |
| setor | String | Setor do funcionário |

### DocCliente (`tabela: docCliente`)
| Campo | Tipo | Descrição |
|---|---|---|
| id | Integer | Identificador (PK) |
| nomeCliente | String | Nome do cliente |
| cnpjCliente | String | CNPJ |
| emailCliente | String | E-mail |
| telefoneCliente | String | Telefone |
| observacaoCliente | String | Observações gerais |
