# Skill: Elicitação de Requisitos Funcionais (RF)

Esta skill orienta a extração e documentação de Requisitos Funcionais para o pipeline de dados Olist, garantindo que o comportamento esperado do sistema esteja alinhado com as necessidades de negócio e SRE.

## Objetivo
Identificar "o que" o sistema deve fazer para garantir o fluxo de dados do CSV para o banco analítico, mantendo a integridade e visibilidade.

## Diretrizes de Elicitação

Ao elicitar requisitos para este projeto, foque nos seguintes domínios:

### 1. Ingestão e Processamento
- Como os arquivos CSV devem ser lidos?
- Qual a frequência de processamento?
- Como lidar com arquivos já processados (Idempotência)?

### 2. Transformação e Qualidade
- Quais campos são obrigatórios?
- Existem regras de limpeza de dados?
- Como o sistema deve reagir a tipos de dados inválidos?

### 3. Carga e Armazenamento
- Onde os dados serão persistidos?
- Como garantir que não haja duplicidade no banco? (ex: UPSERT)

### 4. Observabilidade e Notificação
- O sistema deve gerar logs de cada etapa?
- Quais métricas devem ser expostas para o Grafana?

## Formato do Requisito
Cada requisito deve seguir o padrão:
- **ID:** RF-XXX
- **Nome:** Título curto e descritivo.
- **Descrição:** Detalhes da funcionalidade.
- **Critérios de Aceitação:** O que deve ser verdade para o requisito ser considerado entregue.

## Exemplo de Requisito Funcional

**RF-001: Processamento de Arquivos CSV**
- **Descrição:** O sistema deve ler arquivos CSV da pasta de origem e carregar os dados na tabela de staging do Postgres.
- **Critérios de Aceitação:** 
  - Suportar arquivos com até 100k linhas.
  - Ignorar cabeçalhos.
  - Registrar no log o início e o fim da leitura.

---
*Esta skill deve ser utilizada sempre que houver necessidade de detalhar novas funcionalidades ou revisar o escopo funcional no arquivo `01_functional_requirements.md`.*
