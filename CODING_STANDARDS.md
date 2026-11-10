# VittaCare — Guia de Padrões de Código

Bloco 8 — Documentação para Desenvolvimento

Este guia vale para todo o código do VittaCare. Em caso de dúvida, as decisões de arquitetura estão nos ADRs do Bloco 3 e o modelo de dados no Bloco 5.

## 1. Tecnologias de referência

| Camada | Tecnologia | Origem |
|---|---|---|
| Back-end | Java + Spring Boot (monólito modular) | ADR-001, ADR-003 |
| Persistência | Spring Data JPA / Hibernate | ADR-009 |
| Segurança | Spring Security + JWT | ADR-006 |
| Front-end web | React (JavaScript/JSX), SPA | ADR-002 |
| Banco de dados | PostgreSQL (Amazon RDS Multi-AZ) | ADR-004 |
| API | REST + JSON sobre HTTPS, OpenAPI 3.0 | ADR-005, ADR-010 |
| Containers | Docker | ADR-008 |
| Testes | JUnit 5, Mockito, Spring Boot Test + Testcontainers, Jest, React Testing Library, Playwright | Bloco 9 §3.3 |

Versões de Java, Spring Boot e Node: Java 21 (LTS) e Spring Boot 3.x, a confirmar na tarefa S1-01 da Sprint 1 e registrar no README.

## 2. Estrutura do back-end (monólito modular)

O código é organizado por módulo de domínio, e dentro de cada módulo por camada. Um módulo **nunca** acessa o `Repository` de outro módulo: ele chama o `Service` (interface) do outro módulo.

```
com.vittacare
├── autenticacao        (controller, service, security)
├── usuarios            (controller, service, repository, domain, dto)
├── clinicas
├── pacientes
├── profissionais
├── especialidades
├── disponibilidade
├── agendamentos
├── atendimentos
├── notificacoes
├── relatorios          (relatórios e auditoria)
└── compartilhado       (exceções, ContextoClinica, configuração)
```

## 3. Convenções de nomenclatura

| Elemento | Padrão | Exemplo |
|---|---|---|
| Classes | PascalCase | `AgendamentoService` |
| Métodos e variáveis | camelCase | `agendarConsulta()`, `inicioEm` |
| Constantes | UPPER_SNAKE_CASE | `LIMITE_TENTATIVAS_LOGIN` |
| Pacotes Java | minúsculas, sem acento | `com.vittacare.agendamentos` |
| DTOs | sufixo `RequestDTO` / `ResponseDTO` | `AgendamentoRequestDTO` |
| Exceções | sufixo `Exception` | `ConflitoAgendamentoException` |
| Tabelas e colunas | snake_case, singular (igual ao Bloco 5) | `agendamento.disponibilidade_id` |
| Endpoints REST | substantivos no plural, minúsculas | `/api/v1/agendamentos` |
| Componentes React | PascalCase | `AgendaProfissional.jsx` |
| Funções e hooks React | camelCase (hooks com `use`) | `useAgendamentos` |
| Testes | `deve<Resultado>Quando<Condicao>` | `deveLancarConflitoQuandoHorarioJaReservado` |

Nomes de domínio ficam em português, como no glossário do Bloco 9. Nomes técnicos (Controller, Service, Repository) seguem o padrão do Spring.

## 4. Responsabilidade das camadas

- **Controller:** recebe a requisição, valida a entrada (`@Valid`), converte DTO e chama o Service. Não contém regra de negócio.
- **Service:** concentra regras de negócio, transações (`@Transactional`), permissões e contexto da clínica.
- **Repository:** apenas acesso a dados (Spring Data JPA). Toda consulta filtra por `clinica_id`.
- **DTO:** a API nunca expõe entidades JPA nem o `senha_hash`.

## 5. Regras obrigatórias do domínio

1. **Isolamento por clínica (RF025, RNF014):** o `clinicaId` vem **sempre** do token JWT, via `ContextoClinica`. Um `clinicaId` recebido no corpo ou na URL nunca é prova de autorização.
2. **Autorização no back-end (RNF010):** todo endpoint protegido valida assinatura, expiração, perfil e clínica. Ocultar um botão no React não é segurança.
3. **Agendamento (RF018, RNF008):** a criação roda em transação. O índice único parcial `uq_agendamento_disponibilidade_ativa` é a última barreira: a violação vira `ConflitoAgendamentoException` (HTTP 409).
4. **Bloqueio de horário (RF026):** verificar agendamentos ativos (`AGENDADO` ou `CONFIRMADO`) e gravar o bloqueio na mesma transação. Se houver agendamento, a resposta é HTTP 409.
5. **Senhas (RNF011):** apenas hash Argon2id ou bcrypt. Nunca texto puro, nunca em log.
6. **Segredos:** nada de senha, chave JWT ou credencial no repositório. Use variáveis de ambiente (`.env.example` como modelo).
7. **LGPD (RNF012):** não registrar dados pessoais (CPF, e-mail, telefone) em logs. Campos pessoais seguem o dicionário de dados do Bloco 5.
8. **Auditoria (RNF013):** operações administrativas críticas gravam `registro_auditoria` (usuário, clínica, ação, entidade, data/hora).
9. **Datas:** `OffsetDateTime` no Java e `TIMESTAMPTZ` no banco. A API trafega ISO-8601.

## 6. Exemplo (serviço de agendamento, alinhado ao C4 do Bloco 3)

```java
@Service
public class AgendamentoService {

    private final AgendamentoRepository agendamentoRepository;
    private final DisponibilidadeRepository disponibilidadeRepository;
    private final NotificacaoService notificacaoService;

    public AgendamentoService(AgendamentoRepository agendamentoRepository,
                              DisponibilidadeRepository disponibilidadeRepository,
                              NotificacaoService notificacaoService) {
        this.agendamentoRepository = agendamentoRepository;
        this.disponibilidadeRepository = disponibilidadeRepository;
        this.notificacaoService = notificacaoService;
    }

    @Transactional
    public AgendamentoResponseDTO criar(AgendamentoRequestDTO req, ContextoClinica ctx) {
        validarContexto(ctx);

        Disponibilidade disponibilidade = disponibilidadeRepository
                .findByClinicaIdAndId(ctx.clinicaId(), req.disponibilidadeId())
                .orElseThrow(() -> new RecursoNaoEncontradoException("Disponibilidade não encontrada"));

        boolean ocupada = disponibilidade.getStatus() != StatusDisponibilidade.DISPONIVEL
                || agendamentoRepository.existeAtivoPorDisponibilidade(
                        ctx.clinicaId(), disponibilidade.getId());
        if (ocupada) {
            throw new ConflitoAgendamentoException("Horário indisponível");
        }

        Agendamento agendamento = Agendamento.novo(req, disponibilidade, ctx);

        try {
            Agendamento salvo = agendamentoRepository.save(agendamento);
            disponibilidade.marcarComoAgendada();
            notificacaoService.enviarConfirmacao(salvo); // falha de envio não desfaz o agendamento (R08)
            return AgendamentoResponseDTO.de(salvo);
        } catch (DataIntegrityViolationException e) {
            throw tratarConflito(e); // índice único parcial: segunda requisição concorrente falha
        }
    }
}
```

## 7. Tratamento padronizado de erros

Um `@RestControllerAdvice` converte exceções em resposta JSON com o formato `Erro` do contrato (`timestamp`, `status`, `codigo`, `mensagem`, `detalhes`).

| Exceção | HTTP |
|---|---|
| Validação de entrada (`MethodArgumentNotValidException`) | 400 |
| Credenciais inválidas, token ausente ou expirado | 401 |
| Perfil sem permissão ou recurso de outra clínica | 403 |
| `RecursoNaoEncontradoException` | 404 |
| `ConflitoAgendamentoException`, `BloqueioNaoPermitidoException` | 409 |
| Erro inesperado | 500 (sem detalhes internos na resposta) |

## 8. Testes e cobertura (metas do Bloco 9)

- Back-end: linhas ≥ 80% e branches ≥ 70%.
- Módulos críticos (autenticação, autorização, agendamento, tenancy): ≥ 90% de linhas nas regras de domínio.
- Front-end: ≥ 70%.
- Todo defeito corrigido ganha um teste de regressão.
- Obrigatórios: teste de acesso cruzado entre clínicas e teste de concorrência no agendamento.
- Cobertura medida no pipeline (JaCoCo e Jest). Não reduzir a meta só para o pipeline passar.

## 9. Boas práticas

- Nomes claros e descritivos. Sem abreviações obscuras.
- Aplicar os princípios SOLID e evitar duplicação.
- Injeção de dependência por construtor (sem `@Autowired` em campo).
- Validar toda entrada da API (Bean Validation) e limitar tamanhos.
- Métodos pequenos, com uma única responsabilidade.
- Comentar o "porquê", não o "o quê".
- Documentar endpoints no OpenAPI no mesmo PR que altera a API.
- Migrations versionadas e testadas. Nunca alterar uma migration já aplicada.
- No front-end: componentes pequenos e reutilizáveis, estados de carregamento, erro, vazio e sucesso, navegação por teclado e texto alternativo (RNF017).
