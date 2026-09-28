# AWS Step Functions

## O que é

O AWS Step Functions é um serviço da AWS usado para orquestrar workflows entre diferentes serviços e aplicações.

Ele permite definir, visualizar e executar um fluxo de trabalho composto por várias etapas, controlando:

- Ordem de execução
- Condições
- Paralelismo
- Repetições
- Tratamento de erros
- Tempo de espera
- Entrada e saída de dados

A definição dos workflows é feita usando principalmente a Amazon States Language (ASL), uma linguagem baseada em JSON.

> **Ideia principal:** o Step Functions define o que deve acontecer, em qual ordem e em quais condições, enquanto outros serviços executam o trabalho propriamente dito.

## Principais características

- **Orquestração de serviços:** integra serviços como Lambda, SNS, SQS, ECS, Batch, DynamoDB e outros.
- **Workflows visuais:** permite visualizar o fluxo de execução no console da AWS.
- **Low-code:** grande parte da lógica pode ser definida declarativamente usando ASL.
- **Controle de erros:** permite configurar Retry e Catch.
- **Execução paralela:** possibilita executar diferentes etapas simultaneamente.
- **Decisões condicionais:** permite criar fluxos semelhantes a if/else.
- **Loops:** permite executar uma etapa para cada item de uma coleção.
- **Integração entre workflows:** uma Step Function pode iniciar outra execução.
- **Observabilidade:** permite acompanhar o histórico e o estado de cada execução.

## State Machine

O principal conceito do Step Functions é a State Machine.

Uma State Machine representa o workflow completo e é formada por vários States (estados).

### Exemplo

<pre>
Início
  │
  ▼
Validar pedido
  │
  ├── inválido ──► Falha
  │
  ▼
Processar pagamento
  │
  ▼
Enviar notificação
  │
  ▼
Fim

</pre>

Cada etapa possui uma responsabilidade e pode receber dados de entrada, processá-los e produzir dados de saída para o próximo estado.

# Tipos de orquestração

## 1. Sequenciamento

Permite executar tarefas em uma ordem definida.

<pre>
Task A
  ↓
Task B
  ↓
Task C

</pre>

### Exemplo

<pre>
Validar pedido
      ↓
Processar pagamento
      ↓
Atualizar banco
      ↓
Enviar notificação

</pre>

É útil quando uma etapa depende do resultado da anterior.

## 2. Retry — repetição em caso de erro

É possível configurar uma tarefa para ser executada novamente caso ocorra uma falha.

<pre>
Task
 │
 ├── sucesso ──► próximo estado
 │
 └── erro
       │
       ▼
     Retry
       │
       └──► Task novamente

</pre>

É possível configurar:

- Quantidade máxima de tentativas
- Tempo entre tentativas
- Multiplicação progressiva do intervalo
- Tipos específicos de erro que devem provocar retry

### Exemplo

<pre>
"Retry": [
  {
    "ErrorEquals": ["Timeout"],
    "IntervalSeconds": 2,
    "MaxAttempts": 3,
    "BackoffRate": 2
  }
]

</pre>

## 3. Catch — tratamento de erros

O Catch permite definir o que deve acontecer quando uma tarefa falha depois das tentativas configuradas.

É semelhante ao conceito de:

<pre>
try {
    executar()
}
catch {
    tratarErro()
}

</pre>

### Exemplo

<pre>
Processar pagamento
       │
       ├── sucesso ──► Finalizar pedido
       │
       └── erro ─────► Registrar erro
                            │
                            ▼
                       Notificar usuário

</pre>

## 4. Execução paralela

O estado Parallel permite executar diferentes branches simultaneamente.

<pre>
             ┌──► Processar pagamento ──┐
             │                          │
Início ──────┼──► Atualizar estoque ────┼──► Próxima etapa
             │                          │
             └──► Enviar notificação ───┘

</pre>

É útil quando as tarefas não dependem umas das outras.

## 5. Fluxo condicional

O estado Choice permite tomar decisões com base nos dados de entrada ou nos resultados de etapas anteriores.

É equivalente conceitualmente a:

<pre>
if / else if / else

</pre>

### Exemplo

<pre>
Valor do pedido
      │
      ▼
    Choice
   /      \
 > 100   <= 100
   │        │
   ▼        ▼
Frete grátis  Frete normal

</pre>

# Principais tipos de State

## Task

Executa uma tarefa ou chama outro serviço.

É um dos estados mais utilizados.

### Exemplos de integrações

- AWS Lambda
- Amazon SNS
- Amazon SQS
- Amazon DynamoDB
- AWS Batch
- Amazon ECS
- AWS Step Functions

### Exemplos de operações

- Lambda → Invoke
- SNS → Publish
- Step Functions → StartExecution

## Choice

Cria decisões condicionais.

<pre>
Choice
 ├── condição A → Task A
 ├── condição B → Task B
 └── Default     → Task C

</pre>

Representa uma lógica semelhante a:

<pre>
if / else

</pre>

## Parallel

Executa múltiplos branches simultaneamente.

<pre>
Parallel
 ├── Branch 1
 ├── Branch 2
 └── Branch 3

</pre>

O fluxo pode continuar depois que os branches terminarem.

## Map

Usado para executar uma mesma lógica para cada item de uma coleção.

É semelhante a um:

<pre>
for each

</pre>

### Exemplo

<pre>
{
  "pedidos": [
    "pedido1",
    "pedido2",
    "pedido3"
  ]
}

</pre>

O Map pode executar:

<pre>
Processar pedido1
Processar pedido2
Processar pedido3

</pre>

É útil para processamento de listas e coleções.

## Pass

Não executa uma operação externa.

Pode ser utilizado para:

- Passar dados para o próximo estado
- Modificar dados
- Adicionar valores
- Criar placeholders durante o desenvolvimento

### Exemplo

<pre>
Input
  ↓
Pass
  ↓
Task

</pre>

## Wait

Pausa a execução por determinado período ou até uma condição relacionada ao workflow.

### Exemplo

<pre>
Task
  ↓
Wait 30 segundos
  ↓
Task

</pre>

Pode ser útil quando o workflow precisa aguardar algum evento, período ou momento específico.

## Succeed

Finaliza a execução com sucesso.

<pre>
Task
 ↓
Succeed

</pre>

## Fail

Finaliza a execução indicando uma falha.

<pre>
Task
 ↓
Fail

</pre>

# Entrada e saída de dados

Os estados podem receber e produzir dados.

O fluxo pode ser entendido como:

<pre>
Input
  ↓
State A
  ↓
Output A
  ↓
State B
  ↓
Output B
  ↓
State C

</pre>

### Exemplo de entrada

<pre>
{
  "pedido": 123,
  "valor": 250
}

</pre>

Uma Task pode processar esses dados e entregar informações para o próximo estado.

O Step Functions possui recursos para controlar quais dados entram e saem dos estados, permitindo construir pipelines de processamento.

# Integração com outros serviços AWS

O Step Functions atua principalmente como orquestrador.

Ele pode coordenar diversos serviços:

<pre>
                    AWS Step Functions
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Lambda             SNS              SQS
          │                │                │
       Executa          Publica          Mensagem
       lógica         notificação       assíncrona

</pre>

### Alguns exemplos

| Serviço            | Uso no Step Functions          |
| ------------------ | ------------------------------ |
| AWS Lambda         | Executar código                |
| Amazon SNS         | Publicar notificações          |
| Amazon SQS         | Enviar mensagens               |
| Amazon DynamoDB    | Consultar ou modificar dados   |
| Amazon ECS         | Executar containers            |
| AWS Batch          | Executar jobs                  |
| AWS Step Functions | Iniciar outra State Machine    |
| Amazon EventBridge | Integração orientada a eventos |

# Step Functions como orquestrador

É importante diferenciar orquestração da execução da lógica.

Imagine um processo de criação de pedido:

<pre>
             Step Functions
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Lambda     DynamoDB     SNS
        │          │          │
    Validação   Persistência  Aviso

</pre>

O Step Functions não precisa conter toda a lógica do sistema.

Ele determina:

1. Execute a validação
2. Se estiver OK, persista o pedido
3. Depois envie a notificação
4. Se falhar, tente novamente
5. Se continuar falhando, trate o erro

Os serviços especializados executam as operações.

# ASL — Amazon States Language

A Amazon States Language (ASL) é a linguagem utilizada para definir uma State Machine.

### Exemplo simplificado

<pre>
{
  "StartAt": "ProcessarPedido",
  "States": {
    "ProcessarPedido": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...",
      "Next": "EnviarNotificacao"
    },
    "EnviarNotificacao": {
      "Type": "Task",
      "Resource": "arn:aws:sns:...",
      "End": true
    }
  }
}

</pre>

### Elementos importantes

<pre>
StartAt
   ↓
Define o primeiro estado

States
   ↓
Define os estados do workflow

Next
   ↓
Define para qual estado seguir

End
   ↓
Indica o término do fluxo

</pre>

# Exemplo de workflow completo

<pre>
                 Início
                   │
                   ▼
           Validar pedido
                   │
                   ▼
                Choice
              /        \
        válido           inválido
          │                 │
          ▼                 ▼
    Processar pedido       Fail
          │
          ▼
       Parallel
       /       \
      /         \
     ▼           ▼
Pagamento     Estoque
     │           │
     └─────┬─────┘
           ▼
     Enviar SNS
           │
           ▼
        Sucesso

</pre>

Nesse workflow temos:

- Task
- Choice
- Parallel
- Tratamento de fluxo
- Integração com serviços AWS
- Succeed / Fail
- Retry + Catch

# Retry + Catch

Um padrão comum é combinar Retry e Catch.

<pre>
             Task
              │
        ┌─────┴─────┐
        │           │
      sucesso      erro
        │           │
        ▼           ▼
     próximo      Retry
                    │
             ┌──────┴──────┐
             │             │
          sucesso        falhou
             │             │
             ▼             ▼
          próximo        Catch
                           │
                           ▼
                     Tratamento

</pre>

Isso permite criar workflows mais resistentes a falhas temporárias.

# Standard vs Express Workflows

O Step Functions possui dois tipos principais de workflows.

## Standard Workflows

Indicados para processos mais longos e que precisam de maior controle sobre a execução.

### Características

- Execução durável
- Execuções de longa duração
- Histórico detalhado da execução
- Duração máxima de até 1 ano
- Modelo de execução com garantias de execução adequadas ao workflow

## Express Workflows

Indicados para workflows de alto volume e curta duração.

### Características

- Alto throughput
- Custo otimizado para grandes quantidades de execuções
- Duração máxima de 5 minutos
- Adequados para arquiteturas orientadas a eventos e processamento de alto volume

A escolha entre Standard e Express depende principalmente de:

- Tempo de execução
- Volume de execuções
- Requisitos de execução
- Necessidade de histórico
- Modelo de processamento

# Step Functions x Mensageria

Um ponto importante:

**Step Functions não é primariamente um serviço de mensageria.**

Serviços como Amazon SQS e Amazon SNS são especializados em comunicação e mensageria.

O Step Functions é principalmente um serviço de orquestração de workflows.

Eles podem ser utilizados juntos:

<pre>
             Step Functions
                   │
                   ▼
             Processar pedido
                   │
                   ▼
                  SQS
                   │
                   ▼
             Outro serviço

</pre>

Nesse cenário:

- **Step Functions:** controla o fluxo
- **SQS:** transporta a mensagem
- **Lambda/ECS/etc.:** executa o processamento

# Principais conceitos

| Conceito      | Função                                          |
| ------------- | ----------------------------------------------- |
| State Machine | Define o workflow completo                      |
| State         | Uma etapa do workflow                           |
| Task          | Executa uma operação                            |
| Choice        | Cria decisões condicionais                      |
| Parallel      | Executa branches em paralelo                    |
| Map           | Itera sobre uma coleção                         |
| Pass          | Manipula ou passa dados sem executar um serviço |
| Wait          | Aguarda determinado período ou condição         |
| Succeed       | Finaliza com sucesso                            |
| Fail          | Finaliza com falha                              |
| Retry         | Tenta novamente uma tarefa                      |
| Catch         | Trata uma falha                                 |
| ASL           | Linguagem de definição do workflow              |
| Execution     | Uma execução concreta da State Machine          |

# Resumo mental

Uma forma simples de entender o AWS Step Functions:

<pre>
              AWS STEP FUNCTIONS
                     │
                     ▼
              ORQUESTRA WORKFLOWS
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
   Sequência      Decisões       Paralelo
       │             │             │
       ▼             ▼             ▼
     Task         Choice         Parallel
       │
       ├── Retry
       ├── Catch
       ├── Wait
       └── Map
                     │
                     ▼
             Serviços AWS
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      Lambda        SNS          SQS
        │            │            │
        ▼            ▼            ▼
      Código     Notificação    Mensagem

</pre>

## Em uma frase

AWS Step Functions é um serviço de orquestração de workflows que permite coordenar serviços e aplicações através de uma State Machine, controlando sequência, decisões, paralelismo, repetição, tratamento de erros, espera e processamento de dados.
