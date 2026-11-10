# VittaCare — Sprint 1: Fundação do projeto e autenticação básica

Bloco 8 — Documentação para Desenvolvimento

Esta sprint segue o planejamento do Bloco 7 (Sprint 1: "Fundação do projeto, estrutura React/Spring Boot, banco, pipeline inicial e autenticação básica") e a decisão do Bloco 9 de configurar o pipeline nesta sprint.

## 1. Visão geral

| Item | Definição |
|---|---|
| Sprint | 01 de 08 |
| Duração | 2 semanas (10 dias úteis) |
| Capacidade planejada | 20 Story Points (velocidade de referência, a validar após as Sprints 1 e 2) |
| Equipe | 2 devs back-end, 1 dev front-end, 1 QA (apoio contínuo); PO, UX/UI e DevOps em apoio parcial |
| Metodologia | Scrum (Planning, Daily, Review, Retrospectiva) |
| Objetivo da sprint | Deixar o projeto executável em qualquer máquina e entregar o primeiro fluxo ponta a ponta: login de um usuário com JWT |
| Prioridade | Alta (todas as sprints seguintes dependem dela) |

## 2. Backlog da sprint

| ID | Item | Referência | SP |
|---|---|---|---|
| S1-01 | Estrutura do repositório e dos projetos (Spring Boot + React) | ADR-001, ADR-002, ADR-003 | 3 |
| S1-02 | Ambiente local com Docker Compose (PostgreSQL + API + front) e `.env.example` | ADR-008 | 3 |
| S1-03 | Migrations iniciais do banco (identidade e acesso) e dados de teste | Bloco 5, RNF008, RNF014 | 3 |
| S1-04 | Pipeline de CI inicial (build, lint, testes, cobertura, segurança) | Bloco 9 §3.3 e §10 | 3 |
| S1-05 | Spring Security + JWT (filtro, claims de perfil e clínica, 401/403) | RF001, RNF009–RNF011, ADR-006 | 3 |
| S1-06 | US002 — Login com e-mail e senha (endpoint + tela React) | RF001, RNF015–RNF017 | 3 |
| S1-07 | Swagger/OpenAPI base publicado e contrato inicial da API | ADR-010 | 2 |
| | **Total** | | **20** |

## 3. Critérios de aceite

### S1-01 — Estrutura do repositório e dos projetos
- Repositório com `backend/`, `frontend/` e `docs/`, e pacotes por módulo conforme o Guia de Padrões de Código.
- `GET /actuator/health` (ou equivalente) responde `200` com a API rodando.
- O front React abre a página inicial e fala com a API pela URL configurada em variável de ambiente.
- Versões de Java, Spring Boot e Node confirmadas e registradas no README.
- Branches `main` e `develop` criadas e protegidas, com modelo de PR configurado.

### S1-02 — Ambiente local com Docker
- `docker compose up` sobe PostgreSQL, API e front sem passos manuais além de copiar `.env.example` para `.env`.
- Nenhum segredo versionado; `.env` está no `.gitignore`.
- README descreve o caminho do `git clone` até o localhost funcionando.

### S1-03 — Migrations iniciais
- Migrations versionadas (ferramenta de migrations a registrar em ADR; proposta: Flyway) criam as tabelas `clinica`, `usuario`, `papel`, `permissao`, `usuario_papel` e `papel_permissao`, exatamente como o DDL do Bloco 5, incluindo FKs compostas por `clinica_id`.
- Seed de **desenvolvimento** com 1 clínica de teste, os 4 papéis (`PACIENTE`, `PROFISSIONAL`, `RECEPCIONISTA`, `ADMINISTRADOR`) e 1 usuário por perfil. Dados fictícios, com senha de desenvolvimento vinda do `.env`, nunca do repositório.
- Migration roda do zero em banco vazio (teste com Testcontainers).
- Teste de integração confirma que `usuario.email` é único por clínica e que a FK composta impede referência entre clínicas.

### S1-04 — Pipeline de CI inicial
- A cada PR para a `develop`, o pipeline executa: lint, build, testes unitários, testes de integração, cobertura (JaCoCo e Jest) e verificação de dependências/segredos.
- PR com check vermelho não pode ser integrado.
- Relatório de cobertura disponível no PR.

### S1-05 — Spring Security + JWT
- **Given** um token válido, **When** acessa endpoint protegido, **Then** a requisição passa com o perfil e a `clinicaId` do token em `ContextoClinica`.
- **Given** token ausente, expirado ou com assinatura inválida, **When** acessa endpoint protegido, **Then** resposta `401` no formato `Erro`.
- **Given** perfil sem permissão para o endpoint, **When** acessa, **Then** resposta `403`.
- A chave de assinatura e a validade do token vêm de variáveis de ambiente.
- Senhas verificadas com Argon2id ou bcrypt (RNF011).
- Comunicação preparada para HTTPS/TLS 1.2+ nos ambientes publicados (RNF009).

### S1-06 — US002: Login
> Como paciente, quero entrar na plataforma utilizando meu e-mail e senha, para acessar minhas funcionalidades pessoais. (3 SP, Bloco 2)

- **Given** o usuário possui conta `ATIVO`, **When** informa credenciais válidas, **Then** o sistema responde `200` com token JWT e perfil, e a interface exibe as funcionalidades do perfil.
- **Given** senha incorreta, e-mail inexistente ou conta `INATIVO`/`BLOQUEADO`, **When** tenta entrar, **Then** `401` com mensagem genérica, sem revelar se o e-mail existe.
- A tela de login funciona por teclado e tem rótulos textuais (RNF017) e abre em Chrome, Firefox e Edge (RNF016).
- Testes: unitários do serviço de autenticação, integração do endpoint e um E2E do fluxo de login.

### S1-07 — OpenAPI base
- Swagger UI acessível no ambiente local, exibindo ao menos `POST /auth/login`.
- `docs/openapi.yaml` versionado no repositório e igual ao publicado pela API.

## 4. Plano sugerido por dia

| Dias | Foco |
|---|---|
| 1–2 | S1-01 e S1-02 (estrutura e ambiente local) |
| 3–4 | S1-03 e S1-04 (banco e pipeline) |
| 5–6 | S1-05 (segurança e JWT) |
| 7–8 | S1-06 (login: back-end e tela) |
| 9 | S1-07 e correções |
| 10 | Revisão, demonstração (Sprint Review) e Retrospectiva |

## 5. Definition of Ready e Definition of Done

- **DoR:** atendido para todos os itens acima: critérios verificáveis, vínculo com RF/RNF ou ADR, dependências identificadas, estimativa em SP e dados de teste não reais.
- **DoD:** valem os critérios do Bloco 9 §6. Resumo: critérios de aceite atendidos, código revisado e integrado na `develop`, testes passando, pipeline verde, documentação, API e variáveis de ambiente atualizadas, sem segredos no repositório e demonstração ao PO.

## 6. Dependências e riscos

- Depende de: decisões do Bloco 3 (stack), Bloco 5 (DDL) e Bloco 9 (ferramentas de teste).
- R06 (velocidade menor que 20 SP): calcular a velocidade real ao fim da sprint e replanejar com o PO se for necessário.
- R05 (sobrecarga de pessoas-chave): revisão de código cruzada e documentação das decisões.
- R10 (defeitos em fluxos críticos): testes automatizados desde a primeira sprint.

## 7. Fora do escopo desta sprint

- Recuperação de senha e papéis/permissões completos: Sprint 2.
- Cadastros de paciente, profissional e especialidade: Sprint 3.
- Pesquisa, disponibilidade e **agendamento de consulta (UC01)**: Sprints 4 a 6.

## 8. Demonstração (Sprint Review)

1. `git clone` e `docker compose up` em máquina limpa.
2. Login com um usuário de teste e visualização do perfil.
3. Tentativa com senha errada (401) e com token expirado.
4. Swagger UI com o contrato de login.
5. Pipeline rodando em um PR e o relatório de cobertura.
