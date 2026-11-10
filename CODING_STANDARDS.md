Contrato de API — VittaCare
Bloco 8 — Documentação para Desenvolvimento

O contrato define como o aplicativo mobile e o sistema web poderão se comunicar com o backend.

Especificações previstas
Item	                    | Definição
Nome	                    | VittaCare API
Versão	                  | 1.0.0
Padrão	                  | OpenAPI 3.0
Arquitetura	              | REST
Formato de dados	        | JSON
Autenticação	            | JWT Bearer Token
Banco de dados previsto	  | PostgreSQL

Principais endpoints
Método	Endpoint	Funcionalidade
POST	/auth/login	Autenticar usuário
POST	/pacientes	Cadastrar paciente
GET	/pacientes/{id}	Consultar paciente
GET	/profissionais	Listar profissionais
GET	/especialidades	Listar especialidades
GET	/disponibilidades	Consultar horários livres
POST	/agendamentos	Agendar consulta
PATCH	/agendamentos/{id}	Remarcar consulta
PATCH	/agendamentos/{id}/cancelamento	Cancelar consulta
GET	/agendamentos	Consultar agendamentos
GET	/relatorios/consultas	Consultar relatórios

Exemplo de agendamento
POST
/v1/agendamentos

Corpo da requisição — JSON

{
  "pacienteId": 1,
  "profissionalId": 5,
  "data": "2026-11-15",
  "hora": "14:00"
}
  

Resposta de sucesso

{
  "id": 10,
  "pacienteId": 1,
  "profissionalId": 5,
  "data": "2026-11-15",
  "hora": "14:00",
  "status": "AGENDADO"
}
  



Observação: este contrato é uma proposta de documentação técnica para o desenvolvimento futuro do VittaCare. Os endpoints, os dados e o endereço ilustrado na imagem são exemplos, não serviços já implementados.
