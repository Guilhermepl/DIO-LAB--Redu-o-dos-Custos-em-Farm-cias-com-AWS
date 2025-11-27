# Plano de Adoção de AWS para Distribuidora Farmacêutica

Este repositório documenta um plano de adoção inicial de **serviços AWS** para uma empresa farmacêutica que atua como **hub de distribuição** entre indústrias e farmácias, partindo de um cenário **sem uso prévio de cloud**.

O objetivo principal é **reduzir custos de infraestrutura** e **otimizar processos de distribuição e integração com parceiros**, utilizando três serviços centrais da AWS:

- **Amazon S3** – Armazenamento de documentos e backups  
- **Amazon RDS (PostgreSQL)** – Banco de dados transacional gerenciado  
- **AWS Lambda** – Camada de integração serverless com parceiros

---

## 1. Contexto do Negócio

A empresa `Abstergo Pharma Distribuidora` funciona como um **hub logístico**, conectando:

- Indústrias farmacêuticas  
- Redes de farmácias e drogarias  
- Operadores logísticos e parceiros de transporte  

### 1.1. Situação Atual (On-Premises)

- Servidores físicos ou datacenter terceirizado  
- Arquivos (notas fiscais, laudos, relatórios) em servidor de arquivos local  
- Banco de dados rodando em servidor próprio (licenciamento + manutenção)  
- Backups manuais ou semi-automatizados  
- Integrações com parceiros via planilha, e-mail ou troca de arquivos manual

### 1.2. Objetivos da Migração para AWS

- Reduzir custos de infraestrutura (servidores, storage local, fitas de backup)  
- Ganhar previsibilidade de custos e facilidade de escalar  
- Melhorar segurança, disponibilidade e rastreabilidade dos dados  
- Criar base para integrações com parceiros (APIs, rotinas automatizadas)  
- Preparar a empresa para futuras evoluções (BI, dashboards, automações)

---

## 2. Visão Geral da Arquitetura Proposta

A proposta é uma **migração em três etapas**, focada em ganhos rápidos e tangíveis, sem tentativa de migração completa de uma só vez.

1. **Etapa 1 – Amazon S3**  
   - Armazenamento de documentos fiscais, laudos, relatórios e backups  
   - Substituição gradual do servidor de arquivos local  
   - Políticas de ciclo de vida para reduzir custo (S3 Standard → IA → Glacier)

2. **Etapa 2 – Amazon RDS (PostgreSQL)**  
   - Migração do banco de dados dos sistemas críticos (estoque, pedidos, faturamento)  
   - Banco gerenciado, com backups automáticos e alta disponibilidade  

3. **Etapa 3 – AWS Lambda**  
   - Funções serverless para integrações com parceiros (indústrias e farmácias)  
   - APIs e rotinas agendadas sem necessidade de servidores dedicados

---

## 3. Etapas do Projeto e Serviços AWS

### 3.1. Etapa 1 – Armazenamento e Backup com Amazon S3

**Serviço:**  
- `Amazon S3 (Simple Storage Service)`

**Foco:**  
- Armazenamento centralizado e seguro de documentos e backups  
- Redução de custo com storage local e backup físico

**Casos de Uso:**

- Buckets segmentados por tipo de dado, por exemplo:  
  - `empresa-documentos-fiscais` (XML/PDF de NF-e, CT-e etc.)  
  - `empresa-documentos-regulatorios` (laudos, relatórios ANVISA, etc.)  
  - `empresa-backups-sistemas` (dumps de banco, arquivos de configuração)  

- Configuração de:
  - Versionamento de objetos  
  - Criptografia em repouso (SSE-S3 ou SSE-KMS)  
  - Políticas de ciclo de vida:
    - Arquivos recentes em S3 Standard  
    - Arquivos antigos migrados para S3 Standard-IA  
    - Arquivos muito antigos para S3 Glacier / Glacier Deep Archive

**Resultados esperados:**

- Redução de custo com storage local e mídias externas  
- Processo de backup mais simples, automatizado e auditável  
- Acesso centralizado a documentos em auditorias e fiscalizações

---

### 3.2. Etapa 2 – Banco de Dados Transacional com Amazon RDS

**Serviço:**  
- `Amazon RDS for PostgreSQL`  
  (podendo ser adaptado para outro engine compatível com o ERP atual, se necessário)

**Foco:**  
- Banco de dados transacional gerenciado para sistemas-chave:
  - Estoque  
  - Pedidos  
  - Faturamento  
  - Cadastro de produtos, clientes e parceiros

**Casos de Uso:**

- Criação de uma instância RDS em ambiente de homologação  
- Migração gradual dos módulos mais críticos (por exemplo: estoque e pedidos)  
- Configuração de:
  - Multi-AZ para alta disponibilidade  
  - Backups automáticos diários com retenção definida  
  - Snapshots antes de grandes releases ou migrações

**Resultados esperados:**

- Redução de esforço interno com administração de banco (patch, backup, tuning básico)  
- Menor risco de indisponibilidade por falha física  
- Base sólida para integrações futuras (APIs, relatórios, BI)

---

### 3.3. Etapa 3 – Integrações com Parceiros via AWS Lambda

**Serviços:**  
- `AWS Lambda`  
- (Opcionalmente combinado com `API Gateway`, `SQS` ou `EventBridge`)

**Foco:**  
- Integrações sob demanda com indústrias e farmácias  
- Execução de lógica de negócio sem manter servidores 24/7

**Casos de Uso:**

- Funções Lambda para:
  - Receber pedidos via API (API Gateway → Lambda → RDS)  
  - Consultar status de pedidos e retorná-los aos parceiros  
  - Gerar e enviar arquivos de posição de estoque em horários agendados  

- Gatilhos:
  - HTTP (API Gateway) para integrações em tempo quase real  
  - Events (EventBridge/CloudWatch) para rotinas agendadas  
  - S3 para processar arquivos de parceiros enviados para um bucket específico

**Resultados esperados:**

- Pagamento apenas pelo uso (execuções)  
- Facilidade para criar novas integrações sem compra ou configuração de novos servidores  
- Redução de atividades manuais (envio de planilhas, importações manuais no ERP)

---

## 4. Benefícios para o Negócio

### 4.1. Financeiros

- Redução de custo com:
  - Servidores físicos  
  - Storage local e mídias de backup  
  - Manutenção e licenciamento de banco de dados on-premises  
- Modelo de custo mais previsível e ajustável (pago conforme uso)

### 4.2. Operacionais

- Menos tarefas manuais em backup, integrações e movimentação de arquivos  
- Menos incidentes por falha de hardware ou falta de espaço em disco  
- Melhor controle e rastreabilidade de documentos fiscais e regulatórios

### 4.3. Estratégicos

- Base preparada para:
  - Dashboards de nível executivo (KPIs de logística, SLA, nível de serviço)  
  - Projetos futuros de dados e analytics  
  - Ampliação da rede de parceiros com integrações mais profissionais

---

## 5. Roadmap Resumido

1. **Configuração da conta AWS e segurança básica**
   - Criação da conta corporativa  
   - Configuração de usuários/grupos (IAM) e MFA  
   - Organização inicial de billing e tags de custo

2. **Prova de Conceito com Amazon S3**
   - Migração de um conjunto limitado de documentos fiscais  
   - Validação de acesso, performance e processo de backup

3. **Homologação com Amazon RDS**
   - Subida de ambiente de teste  
   - Migração de um módulo do ERP (ex.: estoque)  
   - Testes de integrações e relatórios

4. **Primeira integração em AWS Lambda**
   - Definição de um parceiro piloto  
   - Criação do fluxo pedido → processamento → resposta  
   - Monitoramento de custo e desempenho

5. **Ajustes e expansão**
   - Ampliação da quantidade de documentos em S3  
   - Migração dos demais módulos do ERP para RDS  
   - Adição de novas integrações em Lambda conforme demanda

---

## 6. Responsáveis e Informações Gerais

- **Empresa:** Abstergo Pharma Distribuidora  
- **Responsável pelo projeto:** Guilherme Pereira Lima  
- **Função:** Engenheiro de Software
- **Data de criação do plano:** 27/11/2025
