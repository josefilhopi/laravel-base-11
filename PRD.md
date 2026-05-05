# PRD: Sistema de Consultorio Offline (AcheMed Sync)

## 1. Visao Geral do Produto

Sistema desktop/local para medicos, focado em alta disponibilidade.

O software sincroniza dados da API ACHEMED para um banco SQLite local, permitindo que o atendimento continue mesmo sem internet. A interface deve priorizar usabilidade rapida, design moderno e indicadores de status em tempo real.

## 2. Pilares Tecnologicos

- **Backend:** Laravel 11
- **Frontend:** Blade + Alpine.js + Tailwind CSS
- **Banco local:** SQLite
- **Integracao:** API ACHEMED via REST
- **Design:** moderno, com cores azul e cinza, botoes arredondados (`rounded-full` ou `rounded-xl`)

## 3. Requisitos Funcionais

### 3.1. Autenticacao e Configuracao de Identidade

**Identificacao do profissional**

- O sistema deve buscar no `.env` ou em arquivo de configuracao as variaveis `PROFISSIONAL_CPF` e `PROFISSIONAL_NOME`.
- Se as variaveis nao existirem, o sistema deve redirecionar para uma tela de Primeiro Acesso.
- A tela de Primeiro Acesso deve solicitar CPF e Nome.
- Os dados informados devem ser persistidos para proximas execucoes.

**Conexao API**

- As URLs da API devem ser configuradas via `.env`.
- Devem existir configuracoes para Auth, Sync e HealthCheck.

### 3.2. Sincronizacao e Observer de Conexao

**Carga de dados**

Rotas especificas devem popular o SQLite com:

- `schedulings`
- `requisitions`
- `customers`
- `procedures`
- `users`
- `agreements`

**API Observer**

- Um componente Alpine.js deve realizar um ping na rota de check da API a cada 30 a 60 segundos.

**Sinalizacao visual**

- Uma esfera discreta deve aparecer no header.
- Verde: API online / sincronizado.
- Vermelho: API offline / modo offline ativo.

### 3.3. Dashboard (Home)

**Resumo estatistico**

Cards rapidos com:

- Total de pacientes do dia
- Atendidos
- Pendentes
- Consultas particulares, Plano Economico, pagas e pendentes

**Lista de pacientes proximos**

- Exibir uma visao resumida dos proximos 3 atendimentos.

### 3.4. Fila de Atendimento (Consultorio)

**Visualizacao em card**

Cada paciente deve aparecer em um card contendo:

- Avatar padrao ou foto, se houver
- Nome
- Horario da consulta
- Convenio
- Badge de status: Aguardando, Em Atendimento ou Finalizado
- Status financeiro para Plano Economico: Pago ou Pendente

### 3.5. Prontuario Eletronico (Area do Medico)

Ao selecionar um paciente, deve abrir o prontuario com:

**Header do paciente**

- Nome
- CPF
- Idade
- Data de nascimento
- Plano de saude
- Matricula

**Editor Rich Text**

- Sugestao: Trix ou CKEditor.
- Deve suportar negrito, paragrafo, centralizacao e tamanhos de fonte.

**Secoes de registro**

- Anamnese
- Solicitacao de exames + busca de CID
- Prescricoes, incluindo receituario comum e especial
- Atestados

## 4. Requisitos Nao Funcionais

**Performance**

- A busca na lista de pacientes deve ser instantanea.
- Preferencia por Alpine.js filtrando dados ja carregados a partir do SQLite.

**Offline First**

- O sistema deve abrir e permitir navegacao mesmo se a rota da API falhar no middleware de check.

**UX/UI**

- Paleta principal:
  - Azul Marinho: `#1E3A8A`
  - Cinza Azulado: `#F1F5F9`
- Componentes:
  - Botoes com bordas arredondadas
  - Sombras leves com `shadow-sm`

## 5. Arquitetura de Dados (Mapeamento SQLite)

| Tabela | Origem | Descricao |
| --- | --- | --- |
| `schedulings` | API | Agenda do dia |
| `customers` | API | Dados demograficos do paciente |
| `agreements` | API | Planos de saude e regras de Plano Economico |
| `medical_records` | Local | Dados gerados no consultorio, como anamnese, prescricoes e atestados |

## 6. Fluxo de Trabalho Sugerido (Desenvolvimento)

### Passo 1: Middleware de Conectividade

Criar um `ApiHealthMiddleware` que verifica a disponibilidade da API ACHEMED e injeta uma variavel global `$apiOnline` em todas as views Blade.

### Passo 2: Sincronizador (Command)

Criar um comando:

```bash
php artisan achemed:sync
```

O comando podera ser disparado manualmente ou via Schedule para baixar os JSONs da API e atualizar o SQLite local usando `updateOrCreate()`.

### Passo 3: Interface do Consultorio

Utilizar Alpine.js para gerenciar as abas de Anamnese, Prescricao e Atestado sem recarregar a pagina, reduzindo o risco de perda de dados durante o atendimento.

## 7. Design UI (Sugestao Tailwind)

**Botao primario**

```html
<button class="bg-blue-600 text-white rounded-full px-6 py-2 hover:bg-blue-700 transition">
    Salvar
</button>
```

**Card de paciente**

```html
<div class="bg-white border border-gray-200 rounded-xl p-4 shadow-sm">
    ...
</div>
```

**Status dot**

```html
<div class="w-3 h-3 rounded-full" :class="online ? 'bg-green-500' : 'bg-red-500'"></div>
```

## 8. Acompanhamento do Projeto

Este PRD deve ser tratado como documento vivo. Sempre que houver mudanca relevante de escopo, decisao tecnica ou conclusao de uma etapa funcional, este arquivo deve ser atualizado.

### Estado Atual

- Projeto base Laravel 11 criado.
- Ambiente local com Docker Compose configurado em `docker-composes-local.yaml`.
- README atualizado com comandos de manipulacao dos containers.

### Proximas Etapas Sugeridas

1. Configurar variaveis `.env` para identidade do profissional e endpoints da API ACHEMED.
2. Criar tela de Primeiro Acesso e persistencia de `PROFISSIONAL_CPF` e `PROFISSIONAL_NOME`.
3. Criar middleware ou servico de health check da API.
4. Criar migrations SQLite para dados sincronizados e prontuario local.
5. Implementar comando `php artisan achemed:sync`.
6. Criar layout base Blade com header, indicador de conexao e base visual Tailwind.

### Plano de Execucao

**Fase 0: Ambiente local**

- Subir os servicos Docker locais com `docker-composes-local.yaml`.
- Validar que a aplicacao responde em `http://localhost:8000`.
- Validar que migrations rodam no SQLite local.

**Fase 1: Configuracao base**

- Adicionar variaveis de identidade do profissional e endpoints da API no `.env.example`.
- Criar configuracao centralizada para dados da ACHEMED.
- Definir fallback para Primeiro Acesso quando CPF ou nome do profissional estiverem ausentes.

**Fase 2: Persistencia local**

- Criar migrations e models para dados sincronizados: `schedulings`, `requisitions`, `customers`, `procedures`, `users` e `agreements`.
- Criar estrutura local de prontuario em `medical_records`.
- Preparar seeds ou factories minimas para desenvolvimento offline.

**Fase 3: Conectividade e sincronizacao**

- Criar servico de health check da API.
- Criar middleware ou view composer para expor estado online/offline para o Blade.
- Implementar comando `php artisan achemed:sync` com `updateOrCreate()`.
- Agendar sincronizacao via scheduler.

**Fase 4: Interface principal**

- Criar layout base com Tailwind, header e indicador de conexao.
- Implementar dashboard com indicadores do dia.
- Implementar fila de atendimento com busca rapida via Alpine.js.

**Fase 5: Prontuario medico**

- Implementar tela de prontuario do paciente.
- Integrar editor rich text.
- Criar secoes para Anamnese, Exames, Prescricoes e Atestados.
- Persistir dados localmente para uso offline.

**Fase 6: Validacao**

- Rodar testes automatizados relevantes.
- Validar fluxo offline com API indisponivel.
- Revisar README e PRD ao concluir cada etapa funcional.
