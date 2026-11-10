# VittaCare

> Mais organização para quem cuida. Mais facilidade para quem precisa de cuidado.

Plataforma web SaaS para gestão de clínicas e ambulatórios, focada no **agendamento de consultas** para pacientes, profissionais de saúde, recepcionistas e administradores.

## Sobre o projeto

O VittaCare centraliza marcação, remarcação e cancelamento de consultas, disponibilidade dos profissionais, registro de comparecimento e relatórios administrativos, com controle de acesso por perfil e isolamento lógico dos dados de cada clínica (multi-tenant).

**Perfis de usuário:** Paciente, Profissional de saúde, Recepcionista e Administrador.

**Fora do escopo desta versão:** prontuário eletrônico completo, prescrição, diagnóstico automatizado, teleconsulta, pagamentos e gestão financeira.

## Tecnologias

| Camada | Tecnologia |
| --- | --- |
| Front-end | React |
| Back-end | Java 21 + Spring Boot (monólito modular em camadas) |
| Banco de dados | PostgreSQL (Amazon RDS Multi-AZ em produção) |
| Persistência | Spring Data JPA / Hibernate |
| Segurança | Spring Security + JWT |
| API | REST + JSON sobre HTTPS, documentada com OpenAPI/Swagger |
| Infraestrutura | Docker, AWS (2x EC2 atrás de ALB, S3, CloudWatch, CloudFront) |
| Testes | JUnit 5, Mockito, Testcontainers, Jest, React Testing Library, Playwright, k6 |

## Estrutura do repositório

```
vittacare/
├── README.md
├── .env.example
├── .gitignore
├── CODING_STANDARDS.md
├── GIT_WORKFLOW.md
├── SPRINT_1.md
├── docs/
│   ├── api/openapi.yaml
│   ├── blocos/            # documentação em PDF (blocos 1 a 9)
│   └── diagramas/         # arquivos .drawio editáveis
├── backend/
└── frontend/
```

## Como rodar

### Pré-requisitos

- Git
- Docker e Docker Compose (caminho recomendado)
- Para rodar sem Docker: Java 21 (LTS), Node.js (LTS) e PostgreSQL

### Passo a passo (Docker)

```bash
# 1. Clonar o repositório
git clone https://github.com/<organizacao>/vittacare.git
cd vittacare

# 2. Criar o arquivo de variáveis de ambiente
cp .env.example .env
# Edite o .env: troque POSTGRES_PASSWORD, JWT_SECRET e SEED_DEV_PASSWORD

# 3. Subir banco, API e front-end
docker compose up --build
```

Na primeira execução, as migrations criam as tabelas e o seed de desenvolvimento cria uma clínica de teste e um usuário fictício para cada perfil, com a senha definida em `SEED_DEV_PASSWORD`.

### Endereços locais

| Serviço | Endereço |
| --- | --- |
| Front-end | http://localhost:3000 |
| API | http://localhost:8080/api/v1 |
| Swagger UI | http://localhost:8080/swagger-ui/index.html |
| Health check | http://localhost:8080/actuator/health |
| PostgreSQL | localhost:5432 |

### Rodar sem Docker

```bash
# Banco: crie o banco e o usuário conforme o .env (POSTGRES_HOST=localhost)

# Back-end (exemplo com Maven; use o equivalente se o projeto usar Gradle)
cd backend
./mvnw spring-boot:run

# Front-end
cd frontend
npm install
npm run dev
```

## Testes

```bash
cd backend && ./mvnw test      # JUnit 5 + Testcontainers (exige Docker)
cd frontend && npm test        # Jest + React Testing Library
```

Metas de cobertura e estratégia completa estão no Bloco 9 (qualidade e processo).

## Variáveis de ambiente

O `.env.example` é o modelo versionado. O `.env` real **nunca** vai para o Git. Em staging e produção, os segredos vêm do gerenciador de segredos da AWS.

| Variável | Descrição |
| --- | --- |
| `SPRING_PROFILES_ACTIVE` | Perfil da aplicação (dev, staging, prod) |
| `SERVER_PORT` | Porta da API (8080) |
| `APP_CORS_ALLOWED_ORIGINS` | Origem do front-end permitida |
| `POSTGRES_HOST` / `POSTGRES_PORT` / `POSTGRES_DB` | Endereço e nome do banco |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` | Credenciais do banco |
| `JWT_SECRET` | Chave de assinatura do JWT (mín. 32 caracteres, aleatória, por ambiente) |
| `JWT_EXPIRATION_MINUTES` | Validade do token de acesso |
| `PASSWORD_RESET_TOKEN_MINUTES` | Validade do token de redefinição de senha |
| `SEED_DEV_PASSWORD` | Senha dos usuários fictícios do seed de desenvolvimento |

A lista completa está no `.env.example`.

## Problemas comuns

- **Porta ocupada (3000, 8080 ou 5432):** altere a porta no `.env` ou encerre o processo que a usa.
- **API não conecta ao banco no Docker:** dentro do Compose, `POSTGRES_HOST` deve ser o nome do serviço do banco (por exemplo, `db`), não `localhost`.
- **401 em todas as chamadas:** o token expirou ou o `JWT_SECRET` mudou; faça login novamente.

## Documentação

Toda a documentação do projeto está em `docs/blocos/`:

1. Concepção e escopo
2. Requisitos
3. Arquitetura (C4 e ADRs)
4. Diagramas UML
5. Banco de dados
7. Planejamento e estimativas
8. Documentação para desenvolvedores
9. Qualidade e processo

O contrato da API está em `docs/api/openapi.yaml`. Padrões de código, fluxo Git (Conventional Commits) e o detalhamento da Sprint 1 ficam em `CODING_STANDARDS.md`, `GIT_WORKFLOW.md` e `SPRINT_1.md`.

## Segurança e privacidade

- HTTPS com TLS 1.2+ e senhas com Argon2id ou bcrypt.
- O `clinica_id` vale sempre a clínica do token; o que vem do navegador não é prova de autorização.
- Não versione segredos e não registre CPF, e-mail ou telefone em logs (LGPD).

## Autores

Gabriela Dias e João Batista.
