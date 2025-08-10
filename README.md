# Ponderada Sprint 1 - Engenharia de Requisitos
Nome: Kauã Rodrigues dos Santos
Curso: Engenharia de Software
Professor: Reginaldo Arakaki

Este repositório documenta a expansão de um sistema de automação residencial (iluminação) de **1 casa** para **10.000 casas**, coordenadas por **uma plataforma digital como serviço (SaaS)**.  
A modelagem segue as **5 visões do RM-ODP** e usa **BPMN textual simplificado** e **UML (PlantUML)** para subsidiar implementação.

---

## Visão 1 — Business Drivers (Processo de Negócio)

**Objetivo:** reduzir custo de energia, aumentar conforto e segurança, operar em escala (10k casas) com SLA corporativo.  
**Atores:** Morador, Administrador (Plataforma), Dispositivos (Sensores e Atuadores), Fornecedor de Energia, Correios/Instalação, Suporte NOC/SOC, APIs de Terceiros (pagamento, notificação).  
**Fatores críticos:** escalabilidade, baixa latência (<1s por comando), alta disponibilidade (≥99,9%), segurança ponta-a-ponta, observabilidade e compliance (LGPD).

### 1.1 BPMN
```
[Início: Cliente contrata o serviço]
    ↓
[Registrar casa e dispositivos na plataforma]
    ↓
[Provisionar credenciais de acesso e tópicos IoT]
    ↓
[Instalar ou parear dispositivos na casa]
    ↓
[Gateway: Dispositivos respondem ao health-check?]
    ├── Sim --> [Ativar casa e plano no sistema]
    │              ↓
    │       [Disponibilizar acesso ao aplicativo/web para o cliente]
    │              ↓
    │       [Ir para operação diária]
    │
    └── Não --> [Acionar suporte técnico ou instalador]
                   ↓
           [Repetir teste de health-check]
                   ↓
           [Retornar para verificação de resposta dos dispositivos]
                   
---------------- OPERAÇÃO DIÁRIA ----------------
[Cliente cria cenários de automação (regras e rotinas)]
    ↓
[Plataforma envia comandos aos dispositivos e recebe eventos em tempo real]
    ↓
[Gateway: Ocorreu evento de segurança ou energia?]
    ├── Sim --> [Gerar alerta e notificar o cliente]
    │              ↓
    │       [Executar ação automática (acender, apagar ou piscar luz)]
    │              ↓
    │       [Registrar logs e métricas do evento]
    │              ↓
    │       [Retornar para operação diária]
    │
    └── Não --> [Manter rotina normal]
                    ↓
            [Retornar para operação diária]
                    
---------------- MONITORAMENTO E FATURAMENTO ----------------
[Coletar métricas de consumo, eventos e uso da automação]
    ↓
[Gerar relatórios e disponibilizar para o cliente e administradores]
    ↓
[Efetuar cobrança conforme plano contratado]
    ↓
[Fim]

```
---

## Visões 2 e 3 — Requisitos (Funcionais e Não Funcionais) e UML

### 2.1 Elicitação resumida
- Entrevistas com moradores e síndicos, análise de incidentes (segurança, quedas), benchmark de plataformas IoT, NFR workshops (Escala, Segurança, Disponibilidade), cenários de uso (ligar luz, invasão, rotinas).

### 2.2 Requisitos Funcionais (RF)
- **RF-01** Cadastrar casa, cômodos e dispositivos (sensor presença, atuador de luz).
- **RF-02** Controlar remotamente **acender/apagar/piscar** luzes por cômodo/casa/grupo.
- **RF-03** Receber eventos (presença, estado do atuador) em tempo real.
- **RF-04** Criar rotinas e regras (ex.: “acender às 18h”, “apagar quando última pessoa sair”).
- **RF-05** Alertar invasão/anomalia e registrar **log** de auditoria.
- **RF-06** Gerenciar grupos (condomínio/bairro) e políticas em massa.
- **RF-07** Painel com **status**, métricas e histórico (download de relatórios).
- **RF-08** Autenticar usuários e emitir **tokens** para apps e integrações.
- **RF-09** API pública para integrações (webhooks, billing, notificações).
- **RF-10** Provisionar/atualizar dispositivos “over-the-air” (OTA).

### 2.3 Requisitos Não Funcionais (RNF)
- **RNF-01 Escalabilidade:** suportar **10.000 casas** / **100k+ dispositivos**.
- **RNF-02 Desempenho:** latência de comando-→atuador **< 1s** p95.
- **RNF-03 Disponibilidade:** **≥ 99,9%** mensal (plataforma e broker).
- **RNF-04 Segurança:** TLS, **mTLS**/chaves por dispositivo, RBAC, criptografia em repouso, LGPD.
- **RNF-05 Observabilidade:** logs estruturados, métricas, traços distribuídos, auditoria.
- **RNF-06 Resiliência:** retry exponencial, **circuit breaker**, filas, **backpressure**.
- **RNF-07 Portabilidade:** infraestrutura “cloud-agnostic” onde possível (containers).
- **RNF-08 Manutenibilidade:** versionamento de APIs, testes automatizados, IaC.
- **RNF-09 Privacidade:** minimização de dados pessoais, retenção e descarte.
- **RNF-10 Custo:** otimização por uso (serverless/auto-scale) mantendo SLA.

### 2.4 Rastreabilidade (amostra)

| ID | Fonte | Atende | Observações |
|----|-------|--------|-------------|
| RF-02 | Entrevista/benchmark | Seq. “Acender remotamente” | Regras de grupo incluídas |
| RF-05 | Incidentes de segurança | Seq. “Tratar invasão” | Logs e notificações |
| RNF-02 | Workshop NFR | Arq. broker+workers | p95 < 1s usando MQTT e edge-cache |
| RNF-03 | SLA | Multi-AZ + fila | Health-checks e failover |

---

## 2.5 UML – Modelagem Estática
O diagrama de classes representa a plataforma responsável por gerenciar múltiplas casas com automação de iluminação. A **Plataforma** administra **Casas**, que contêm **Cômodos** equipados com **Sensores** (acesso e presença) e **Atuadores de luz**. Eventos e comandos são registrados para controle e auditoria, enquanto **Grupos** e **Políticas** permitem gerenciar várias casas de forma centralizada. Enums padronizam estados, tipos de comandos e eventos.

<p align="center">
  <img width="931" height="1002" alt="image" src="https://github.com/user-attachments/assets/b65dcc0a-4209-421b-8ae1-9274bd5f86ef" />
</p>

---

## 2.6 UML – Modelagem Dinâmica

**(a) Sala vazia: entra uma pessoa → acender luz**  
<p align="left">
  <img width="458" height="429" alt="image" src="https://github.com/user-attachments/assets/d640fe72-eb0c-4340-a7b5-b680b3af9287" />
</p>

**(b) Sai a última pessoa → apagar luz**  
<p align="left">
  <img width="442" height="398" alt="image" src="https://github.com/user-attachments/assets/350be4d9-cde7-4323-832f-e0dd5ed404ef" />
</p>

**(c) Tratar invasão (plataforma varre cômodos e pisca luz)**  
<p align="left">
  <img width="613" height="387" alt="image" src="https://github.com/user-attachments/assets/a94abb58-9f54-4109-91e8-0069818fb86c" />
</p>

**(d) Acender remotamente por grupo (Plataforma → múltiplas casas)**  
<p align="left">
  <img width="788" height="298" alt="image" src="https://github.com/user-attachments/assets/6cb6a1d0-1d31-473d-8bdf-24e7af79e9b5" />
</p>

## Visão 4 — Decisões de Engenharia (Componentes e Mecanismos)

### Arquitetura
- **Estilo:** Microserviços orientados a eventos.  
- **Componentes principais:**  
  - **Broker MQTT**: telemetria e comandos.  
  - **API HTTP**: integração com aplicativos e painel administrativo.  
  - **Workers assíncronos**: processamento de regras, rotinas e alertas.  
  - **Banco de Dados**: armazenar estado e histórico.  

---

### Topologia
- **Fluxo principal:**  
  `IoT Core/Broker (multi-AZ)` ⇄ `Gateway/Edge (opcional)` ⇄ `Dispositivos`  
- **Camadas:**  
  - **Control Plane:** API, autenticação (Auth), motor de regras (Rules).  
  - **Data Plane:** ingestão, armazenamento de logs e métricas.  

---

### Escala e Resiliência
- Auto-scaling dinâmico.  
- Particionamento e **sharding** de dados.  
- **DLQ** (Dead Letter Queue) para mensagens não processadas.  
- Idempotência em operações críticas.  
- **Retry** com backoff exponencial.  
- **Circuit breaker** para isolar falhas.  

---

### Segurança
- **mTLS** por dispositivo.  
- **RBAC/OIDC** para autenticação e controle de acesso de usuários.  
- **KMS** para gerenciamento seguro de chaves.  
- Segregação de **tenants** via `houseId`.  
- **Rate limiting** para evitar abusos.  
- **WAF** (Web Application Firewall) para proteção contra ataques.  

---

### Observabilidade
- **Tracing distribuído** para rastreamento de requisições.  
- **Logs estruturados** para análise e auditoria.  
- **Métricas** por casa e por grupo.  
- **Auditoria imutável** para conformidade regulatória.  

---

### Atualização OTA (Over-The-Air)
- Canal assinado para distribuição segura de firmware/software.  
- Versionamento e controle de versões.  
- **Rollout gradual** para minimizar riscos.  
- **Rollback** em caso de falhas.  

---

### Custos
- Uso de **serverless** e containers gerenciados para otimizar recursos.  
- **Armazenamento em camadas**: dados quentes para acesso rápido, dados frios para histórico.  


## 4.1 UML – Diagrama de Componentes
<img width="1384" height="656" alt="image" src="https://github.com/user-attachments/assets/faca3267-6dbe-43cf-9528-56362cf3129b" />

## Visão 5 — Plataformas, Ferramentas e Linguagens

### Cloud / IoT
- **AWS IoT Core** (ou alternativas como **Azure IoT Hub** / **GCP IoT**).
- **API Gateway**, **AWS Lambda** e **ECS** para execução e orquestração.

---

### Mensageria
- **MQTT**: mosquitto ou EMQX para comunicação IoT.  
- **Filas internas**: SQS ou Kafka para processamento assíncrono.

---

### Banco de Dados
- **Redis**: gerenciamento de estado em memória.  
- **PostgreSQL** ou **MongoDB**: armazenamento de metadados.  
- **TimeSeries** / **Parquet**: histórico e análise de séries temporais.

---

### Linguagens
- **Backend / Workers**: Node.js com TypeScript ou Python.  
- **Aplicativos móveis**: Swift, Kotlin ou Flutter.

---

### Infra & DevOps
- **Docker / Docker Compose** para empacotamento e execução local.  
- **Terraform** para infraestrutura como código.  
- **CI/CD** com GitHub Actions.  
- **OpenTelemetry** para observabilidade.

---

### Diagramas
- **PlantUML** para modelagem UML.  
- **draw.io** para fluxos e arquiteturas visuais.  
- **C4-PlantUML** (opcional) para diagramas de arquitetura no modelo C4.
