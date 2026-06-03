# GFT - Fundamentos de Cloud com AWS

Anotações e resumos do curso **GFT - Fundamentos de Cloud com AWS**, cobrindo os principais conceitos e serviços da Amazon Web Services estudados ao longo do programa.

---

## ☁️ EC2, EBS e AMI — Fundamentos de Infraestrutura

Visão geral técnica e prática sobre os pilares fundamentais da infraestrutura computacional na AWS: **Amazon Elastic Compute Cloud (EC2)**, **Amazon Elastic Block Store (EBS)** e **Amazon Machine Images (AMI)**.

---

## 🚀 1. Amazon EC2 (Elastic Compute Cloud)

O Amazon EC2 é um serviço que fornece capacidade computacional escalável na nuvem. Ele elimina a necessidade de investir em hardware antecipadamente, permitindo que você lance servidores virtuais conforme a demanda.

### Principais Características

- **Instâncias:** Servidores virtuais que executam suas aplicações.
- **Escalabilidade:** Capacidade de aumentar ou diminuir recursos em minutos.
- **Tipos de Instância:** Otimizadas para diferentes casos de uso (CPU, Memória, GPU, Armazenamento).
- **Segurança:** Controle total via Security Groups (firewalls virtuais).

### Tipos de Instância EC2

```mermaid
graph TD
    EC2[🖥️ Tipos de Instância EC2]

    EC2 --> CPU["⚡ Otimizadas para CPU<br/>(c5, c6i)<br/>Web servers, batch"]
    EC2 --> MEM["🧠 Otimizadas para Memória<br/>(r5, x1)<br/>Bancos de dados, cache"]
    EC2 --> GPU["🎮 Aceleradas por GPU<br/>(p3, g4)<br/>ML, rendering"]
    EC2 --> STO["💾 Otimizadas para Storage<br/>(i3, d3)<br/>NoSQL, data warehouses"]
    EC2 --> GEN["🔄 Uso Geral<br/>(t3, m5)<br/>Diversas cargas de trabalho"]
```

---

## 💾 2. Amazon EBS (Elastic Block Store)

O Amazon EBS fornece volumes de armazenamento em bloco de alto desempenho para uso com instâncias EC2. Pense no EBS como o **"disco rígido"** da sua instância virtual.

### Conceitos Chave

- **Persistência:** Os dados persistem mesmo se a instância for interrompida ou encerrada (se configurado).
- **Snapshots:** Backups incrementais que são salvos no Amazon S3.
- **Tipos de Volume:**
  - **SSD (gp3/io2):** Para cargas de trabalho transacionais e bancos de dados.
  - **HDD (st1/sc1):** Para grandes volumes de dados e processamento sequencial.
- **Elasticidade:** Altere o tamanho ou o tipo de volume sem tempo de inatividade.

### Tipos de Volume EBS

```mermaid
graph LR
    EBS[📦 Amazon EBS]

    EBS --> SSD[SSD]
    EBS --> HDD[HDD]

    SSD --> GP3["gp3 – SSD de uso geral<br/>Até 16.000 IOPS<br/>Cargas de trabalho gerais"]
    SSD --> IO2["io2 – SSD de alta performance<br/>Até 64.000 IOPS<br/>Bancos de dados críticos"]

    HDD --> ST1["st1 – HDD otimizado<br/>para throughput<br/>Big Data, Data Warehouses"]
    HDD --> SC1["sc1 – HDD de baixo custo<br/>Acesso infrequente<br/>Arquivamento"]
```

### Ciclo de Vida de um Snapshot EBS

```mermaid
sequenceDiagram
    participant EC2 as 🖥️ EC2 Instance
    participant EBS as 💾 EBS Volume
    participant S3 as 🪣 Amazon S3

    EC2->>EBS: Grava dados
    Note over EBS: Volume ativo

    EBS->>S3: Cria Snapshot (backup incremental)
    Note over S3: Snapshot armazenado

    S3-->>EBS: Restaura Snapshot
    Note over EBS: Novo volume criado

    EBS-->>EC2: Anexa volume restaurado
```

---

## 💿 3. AMI (Amazon Machine Image)

Uma AMI é um **modelo** que contém a configuração de software (sistema operacional, servidor de aplicativos e aplicações) necessária para lançar uma instância.

### Componentes de uma AMI

- **Template do Volume Raiz:** Contém o SO e aplicações instaladas.
- **Permissões de Lançamento:** Define quais contas AWS podem usar a AMI.
- **Mapeamento de Dispositivos de Bloco:** Especifica os volumes EBS a serem anexados à instância no lançamento.

### Ciclo de Vida de uma AMI

```mermaid
flowchart LR
    A["🖥️ Instância EC2\nexistente"] -->|"1. Criar AMI"| B["💿 AMI\n(sua região)"]
    C["🛒 AWS Marketplace\nou Backup"] -->|"Origem alternativa"| B

    B -->|"2. Replicar\npara outra região"| D["💿 AMI Cópia\n(outra região)"]

    B -->|"3. Lançar"| E["🖥️ Instância A"]
    B -->|"3. Lançar"| F["🖥️ Instância B"]
    B -->|"3. Lançar"| G["🖥️ Instância C"]

    style B fill:#ff9900,color:#fff
    style D fill:#ff9900,color:#fff
```

---

## 🔗 Relacionamento entre os Componentes

A relação entre esses três serviços é o que permite a flexibilidade da nuvem:

1. Você escolhe uma **AMI** (o "molde" do sistema).
2. Você lança uma **instância EC2** baseada nessa AMI.
3. A instância utiliza **volumes EBS** para armazenamento persistente.
4. Você pode criar novos **Snapshots** dos volumes EBS para gerar novas AMIs, fechando o ciclo de backup e replicação de ambiente.

```mermaid
flowchart TD
    AMI["💿 AMI\n(Template)"]
    EC2["🖥️ EC2 Instance"]
    EBS["💾 EBS Volume"]
    SNAP["📸 EBS Snapshot"]
    S3["🪣 Amazon S3"]
    NOVA_AMI["💿 Nova AMI"]

    AMI -->|"1. Lança instância"| EC2
    EC2 <-->|"2. Armazena dados"| EBS
    EBS -->|"3. Cria backup incremental"| SNAP
    SNAP -->|"Armazenado em"| S3
    SNAP -->|"4. Gera nova AMI"| NOVA_AMI
    NOVA_AMI -->|"Reinicia o ciclo ♻️"| EC2

    style AMI fill:#1a73e8,color:#fff
    style EC2 fill:#34a853,color:#fff
    style EBS fill:#ea4335,color:#fff
    style SNAP fill:#fbbc04,color:#000
    style NOVA_AMI fill:#1a73e8,color:#fff
```

### Resumo do Fluxo

| Etapa | Serviço | Ação |
|-------|---------|------|
| 1 | **AMI** | Fornece o template (SO + configurações) |
| 2 | **EC2** | Instância lançada a partir da AMI |
| 3 | **EBS** | Volume anexado para armazenamento persistente |
| 4 | **Snapshot** | Backup do EBS salvo no S3 |
| 5 | **Nova AMI** | Criada a partir do Snapshot para replicar ambientes |

---

> 📚 **Referências:** [Amazon EC2 Docs](https://docs.aws.amazon.com/ec2/) · [Amazon EBS Docs](https://docs.aws.amazon.com/ebs/) · [AMI Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)

---

## ⚙️ AWS Step Functions — Workflows Automatizados

O **AWS Step Functions** é um serviço de orquestração serverless que permite coordenar múltiplos serviços AWS em fluxos de trabalho visuais, chamados de **State Machines** (Máquinas de Estado).

---

## 🧩 4. AWS Step Functions

### O que é?

O Step Functions permite construir aplicações distribuídas e de longa duração combinando serviços como Lambda, ECS, DynamoDB, SNS, SQS e muito mais, sem a necessidade de gerenciar servidores.

### Principais Conceitos

- **State Machine:** Definição do fluxo de trabalho em JSON/YAML usando a linguagem **Amazon States Language (ASL)**.
- **States (Estados):** Cada etapa do fluxo. Podem ser de tarefa, escolha, espera, paralelo, mapa, pass, sucesso ou falha.
- **Execution:** Uma instância em execução de uma State Machine.
- **Transitions:** As transições entre estados, podendo ser condicionais ou sequenciais.
- **Integrations:** Conexões nativas com mais de 220 serviços AWS.

### Tipos de Workflow

```mermaid
graph TD
SF[⚙️ AWS Step Functions]

SF --> STD["🐢 Standard Workflow<br/>Duração: até 1 ano<br/>Execução: Exatamente uma vez<br/>Auditoria completa no histórico"]
SF --> EXP["⚡ Express Workflow<br/>Duração: até 5 minutos<br/>Alta taxa de eventos<br/>Pelo menos uma vez"]

EXP --> SYNC["🔄 Synchronous Express<br/>Aguarda resultado<br/>Ideal para APIs"]
EXP --> ASYNC["📨 Asynchronous Express<br/>Não aguarda resultado<br/>Ideal para event-driven"]

style SF fill:#ff9900,color:#fff
style STD fill:#1a73e8,color:#fff
style EXP fill:#34a853,color:#fff
style SYNC fill:#4285f4,color:#fff
style ASYNC fill:#4285f4,color:#fff
```

### Tipos de Estado (States)

```mermaid
graph LR
ASL["📄 Amazon States Language"]

ASL --> TASK["⚙️ Task<br/>Executa trabalho:<br/>Lambda, ECS, API..."]
ASL --> CHOICE["🔀 Choice<br/>Lógica condicional<br/>(if/else)"]
ASL --> WAIT["⏳ Wait<br/>Pausa por tempo<br/>determinado"]
ASL --> PARALLEL["🔀 Parallel<br/>Executa ramos<br/>em paralelo"]
ASL --> MAP["🗺️ Map<br/>Itera sobre<br/>uma lista"]
ASL --> PASS["➡️ Pass<br/>Passa dados<br/>sem processar"]
ASL --> SUCCEED["✅ Succeed<br/>Encerra com<br/>sucesso"]
ASL --> FAIL["❌ Fail<br/>Encerra com<br/>falha"]

style ASL fill:#ff9900,color:#fff
style TASK fill:#34a853,color:#fff
style CHOICE fill:#1a73e8,color:#fff
style PARALLEL fill:#ea4335,color:#fff
style MAP fill:#9c27b0,color:#fff
```

---

## 🔄 Fluxo de Execução de uma State Machine

```mermaid
flowchart TD
START(["▶️ StartExecution"]) --> S1

S1["⚙️ Task State\nInvocar Lambda\n(validar pedido)"]
S1 -->|Sucesso| S2
S1 -->|Erro| ERR

S2["🔀 Choice State\nPedido válido?"]
S2 -->|Sim| S3
S2 -->|Não| S4

S3["🔀 Parallel State\nProcessamento paralelo"]
S3 --> S3A["📦 Reservar estoque\n(Lambda)"]
S3 --> S3B["💳 Processar pagamento\n(Lambda)"]
S3A --> S5
S3B --> S5

S4["📨 Task State\nNotificar cliente\n(SNS)"]
S4 --> FAIL_END

S5["⏳ Wait State\n30 segundos"]
S5 --> S6

S6["⚙️ Task State\nAtualizar DynamoDB\n(status do pedido)"]
S6 --> SUCCESS_END

ERR["❌ Fail State\nErro na validação"]

SUCCESS_END(["✅ Execution Succeeded"])
FAIL_END(["❌ Execution Failed"])

style START fill:#ff9900,color:#fff
style SUCCESS_END fill:#34a853,color:#fff
style FAIL_END fill:#ea4335,color:#fff
style ERR fill:#ea4335,color:#fff
style S2 fill:#1a73e8,color:#fff
style S3 fill:#9c27b0,color:#fff
```

---

## 🔗 Integrações do Step Functions com outros Serviços AWS

O Step Functions oferece dois tipos de integração:

- **Optimistic Integration:** Chama o serviço e continua sem aguardar.
- **`.sync` Integration:** Aguarda o serviço terminar antes de avançar de estado.
- **`.waitForTaskToken`:** Pausa o fluxo até receber um callback externo (útil para aprovações humanas).

```mermaid
graph TD
SF["⚙️ State Machine"]

SF --> LAMBDA["λ AWS Lambda\nFunções serverless"]
SF --> ECS["🐳 Amazon ECS / Fargate\nContainers"]
SF --> DDB["🗄️ Amazon DynamoDB\nBanco NoSQL"]
SF --> SNS["📢 Amazon SNS\nNotificações"]
SF --> SQS["📬 Amazon SQS\nFilas de mensagens"]
SF --> S3["🪣 Amazon S3\nArmazenamento de objetos"]
SF --> GLUE["🔧 AWS Glue\nETL de dados"]
SF --> BEDROCK["🤖 Amazon Bedrock\nIA Generativa"]

style SF fill:#ff9900,color:#fff
style LAMBDA fill:#1a73e8,color:#fff
style ECS fill:#34a853,color:#fff
style DDB fill:#ea4335,color:#fff
style SNS fill:#fbbc04,color:#000
style SQS fill:#fbbc04,color:#000
style BEDROCK fill:#9c27b0,color:#fff
```

---

## 🧪 Validação: Executando uma State Machine com Lambda

### Etapas Realizadas

| Etapa | Ação | Serviço |
|-------|------|---------|
| 1 | Criar função Lambda para processar a tarefa | AWS Lambda |
| 2 | Criar uma State Machine no Step Functions | Step Functions |
| 3 | Definir os estados em Amazon States Language (JSON) | Step Functions |
| 4 | Configurar a permissão IAM para o Step Functions invocar o Lambda | AWS IAM |
| 5 | Executar a State Machine e monitorar pelo console | Step Functions |
| 6 | Verificar logs de execução no histórico de eventos | Step Functions |

### Exemplo de Definição de Estado (ASL)

```json
{
  "Comment": "Exemplo de State Machine com Lambda",
  "StartAt": "ValidarPedido",
  "States": {
    "ValidarPedido": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789:function:validar-pedido",
      "Next": "VerificarResultado"
    },
    "VerificarResultado": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.valido",
          "BooleanEquals": true,
          "Next": "PedidoAprovado"
        }
      ],
      "Default": "PedidoRejeitado"
    },
    "PedidoAprovado": {
      "Type": "Succeed"
    },
    "PedidoRejeitado": {
      "Type": "Fail",
      "Error": "PedidoInvalido",
      "Cause": "Pedido não passou na validação"
    }
  }
}
```

---

## 🆚 Comparativo: Step Functions vs Outras Abordagens

| Critério | Step Functions | Lambda Encadeado | Fila SQS Pura |
|----------|---------------|------------------|---------------|
| Visibilidade do fluxo | ✅ Visual e auditável | ❌ Sem visibilidade | ❌ Sem visibilidade |
| Tratamento de erros | ✅ Nativo (Retry/Catch) | ⚠️ Manual no código | ⚠️ Manual |
| Longa duração | ✅ Até 1 ano | ❌ Máx. 15 min | ✅ Sim |
| Orquestração paralela | ✅ Estado Parallel/Map | ⚠️ Complexo | ❌ Difícil |
| Custo por execução | 💲 Por transição de estado | 💲 Por invocação | 💲 Por mensagem |
| Casos de uso ideais | Fluxos complexos | Pipelines simples | Desacoplamento |

---

> 📚 **Referências:** [AWS Step Functions Docs](https://docs.aws.amazon.com/step-functions/) · [Amazon States Language](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-amazon-states-language.html) · [Step Functions Workshops](https://catalog.workshops.aws/stepfunctions/en-US)
