# VittaCare — Git Workflow

Bloco 8 — Documentação para Desenvolvimento

## 1. Objetivo

Padronizar versionamento e colaboração da equipe. Ferramentas: **Git** (controle de versão) e **GitHub** (hospedagem, Pull Requests e revisão), conforme o ADR-011.

## 2. Estratégia de branches

| Branch | Finalidade | Origem | Destino |
|---|---|---|---|
| `main` | Versões estáveis, prontas para implantação (tags) | — | — |
| `develop` | Integração das funcionalidades revisadas | `main` | `main` (release) |
| `feature/*` | Nova funcionalidade ou tarefa | `develop` | `develop` |
| `fix/*` | Correção de defeito encontrado no desenvolvimento | `develop` | `develop` |
| `docs/*` | Documentação técnica | `develop` | `develop` |
| `hotfix/*` | Correção urgente em produção | `main` | `main` **e** `develop` |

Nomes em minúsculas, com hífen e sem acento. Exemplos: `feature/autenticacao-jwt`, `feature/agendamento-consulta`, `fix/validacao-horario`, `docs/contrato-api`, `hotfix/falha-autenticacao`.

`main` e `develop` são **protegidas**: sem push direto, só por Pull Request com revisão aprovada e checks do pipeline passando.

## 3. Padrão de commits (Conventional Commits)

Formato: `tipo(escopo): descrição no imperativo, em português`. O escopo é opcional e é o módulo afetado.

| Tipo | Uso |
|---|---|
| `feat` | Nova funcionalidade |
| `fix` | Correção de erro |
| `docs` | Documentação |
| `refactor` | Melhoria interna sem mudar comportamento |
| `test` | Criação ou ajuste de testes |
| `chore` | Manutenção, configuração, dependências |
| `style` | Formatação, sem mudar comportamento |

Exemplos:

```
feat(autenticacao): adicionar login com JWT
fix(agendamentos): corrigir conflito de horários concorrentes
docs(api): atualizar contrato OpenAPI
test(agendamentos): adicionar teste de concorrência
chore(ci): configurar pipeline inicial
```

## 4. Processo de desenvolvimento

1. Atualizar a `develop` local: `git checkout develop && git pull`.
2. Criar a branch de trabalho: `git checkout -b feature/nome-da-tarefa`.
3. Desenvolver seguindo o Guia de Padrões de Código. Fazer commits pequenos e frequentes.
4. Rodar localmente testes, lint e análise estática antes de enviar.
5. Publicar a branch (`git push -u origin feature/nome-da-tarefa`) e abrir um Pull Request para a `develop`.
6. Corrigir os comentários da revisão e aguardar a aprovação.
7. Integrar (squash merge) e apagar a branch.

## 5. Pull Requests

Todo PR deve ter:

- título objetivo (`feat(autenticacao): login com JWT`);
- descrição do que mudou e por quê;
- vínculo com a tarefa/história (ex.: `US002`, `RF001`, `S1-05`);
- evidência de testes executados;
- impactos em API, banco ou variáveis de ambiente;
- pelo menos **1 revisor** (não pode ser o autor).

Modelo de PR (`.github/pull_request_template.md`):

```markdown
## Descrição
<!-- O que foi feito e por quê -->

## Tarefa / História
<!-- Ex.: US002 · RF001 · S1-05 -->

## Como testar
<!-- Passos e evidências -->

## Checklist
- [ ] Segue o Guia de Padrões de Código
- [ ] Testes criados/atualizados e passando
- [ ] Contrato OpenAPI atualizado (se a API mudou)
- [ ] Migrations versionadas (se o banco mudou)
- [ ] Sem segredos ou dados pessoais no código ou nos logs
- [ ] Filtros por clinica_id respeitados
```

**Critérios de aprovação:** checks do pipeline verdes (build, lint, testes, cobertura e segurança), código conforme os padrões, entrada validada, documentação atualizada e aprovação de um revisor. Exceção de cobertura só com justificativa no PR e tarefa para corrigir a lacuna (Bloco 9 §4).

## 6. Revisão de código

Critérios do revisor:

1. Legibilidade e organização.
2. Conformidade com o Guia de Padrões de Código.
3. Ausência de duplicação desnecessária.
4. Tratamento adequado de exceções.
5. Segurança: autorização, isolamento por `clinica_id`, ausência de segredos e de dados pessoais em log.
6. Cobertura de testes, inclusive cenários negativos e de concorrência.
7. Documentação e contrato de API atualizados.

## 7. Versionamento e releases

- Versionamento semântico (`MAJOR.MINOR.PATCH`).
- Ao fim de cada sprint estável, a `develop` é integrada à `main` por PR e recebe uma tag (ex.: `v0.1.0` ao fim da Sprint 1).
- Hotfix: branch a partir da `main`, PR para a `main`, nova tag `PATCH` e merge de volta na `develop`.

## 8. Relação com a Definition of Done (Bloco 9)

Uma tarefa só é "Done" quando o PR foi revisado e integrado na `develop` e os checks do pipeline passaram. A branch "principal de integração" citada no DoD é a `develop`. A `main` recebe apenas versões estáveis.

## 9. Critérios de conclusão do Git Workflow

O workflow está adequado quando há estratégia documentada de branches, padrão de commits, regras de Pull Request, revisão, integração e versionamento, e as branches `main` e `develop` estão protegidas no GitHub.
