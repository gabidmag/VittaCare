
VittaCare — Guia de Padrões de Código
Bloco 8 — Documentação para Desenvolvimento

1. Tecnologias de referência
Camada	Tecnologia
Backend	Java + Spring Boot
Frontend web	React + TypeScript
Aplicativo mobile	Flutter + Dart
Banco de dados	PostgreSQL

2. Convenções de nomenclatura
Elemento	Padrão	Exemplo
Classes	PascalCase	PacienteService
Métodos	camelCase	agendarConsulta()
Variáveis	camelCase	dataConsulta
Constantes	UPPER_SNAKE_CASE	TEMPO_LIMITE
Pacotes Java	Letras minúsculas	com.vittacare.service

3. Exemplo de código Java

@Service
public class AgendamentoService {

    private final AgendamentoRepository repository;

    public AgendamentoService(
            AgendamentoRepository repository) {
        this.repository = repository;
    }

    public Agendamento agendarConsulta(
            AgendamentoDTO dados) {

        // Validar disponibilidade e conflitos
        // antes de salvar o agendamento.

        return repository.save(dados.toEntity());
    }
}

4. Boas práticas adotadas
- Utilizar nomes claros e descritivos.
- Separar responsabilidades entre Controller, Service e Repository.
- Aplicar os princípios SOLID.
- Evitar duplicação de código.
- Validar dados recebidos pela API.
- Tratar exceções de forma padronizada.
- Proteger dados pessoais conforme a LGPD.
- Escrever testes unitários e de integração.
- Documentar endpoints com OpenAPI 3.0.
  
5. Padrão de commits
Prefixo	Finalidade
feat:	Nova funcionalidade
fix:	Correção de erro
docs:	Documentação
refactor:	Refatoração
test:	Testes
chore:	Manutenção
