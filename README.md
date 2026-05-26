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
