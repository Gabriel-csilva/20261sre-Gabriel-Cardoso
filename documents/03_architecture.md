faça um git coomit e git push do projeto# Arquitetura do Sistema (RM-ODP)

Este documento descreve a arquitetura do pipeline de dados Olist utilizando os cinco pontos de vista do framework RM-ODP, garantindo o atendimento aos requisitos funcionais (RF) e não-funcionais (RNF).

---

## 1. Enterprise Viewpoint (Ponto de Vista de Negócio)
**Objetivo:** Processar ~100k pedidos diários para alimentar dashboards analíticos com dados confiáveis e íntegros.

- **Stakeholders:** Operação Olist, Time de Dados, SRE.
- **Processo:** Ingestão de CSV -> Validação -> Carga no Postgres.
- **Valor:** Decisões baseadas em dados atualizados diariamente (SLA).
- **Atendimento:** RF-001 (Ingestão), RF-005 (SLA), RNF-02 (Latência).

## 2. Information Viewpoint (Ponto de Vista de Informação)
**Objetivo:** Definir a semântica e os estados do dado.

- **Modelo de Dados:**
  - **Staging Area:** Dados brutos dos pedidos.
  - **Analytical Layer:** Dados validados com chave única (`order_id`).
- **Estados:** `Pending` (CSV), `Processing` (In-memory), `Persisted` (Postgres).
- **Atendimento:** RF-003 (Carga), RNF-01 (Completitude), RNF-06 (Idempotência).

## 3. Computational Viewpoint (Ponto de Vista Computacional)
**Objetivo:** Decomposição em componentes funcionais.

- **Ingestor Component:** Responsável pela leitura de CSV e parsing. (Atende: RF-001, RNF-03).
- **Validator Component:** Aplica regras de qualidade e loga inconsistências. (Atende: RF-003, RNF-10).
- **Persistence Component:** Realiza a carga idempotente via UPSERT. (Atende: RF-002, RNF-06).
- **Monitor Component:** Exporta métricas para o CloudWatch/Grafana. (Atende: RF-006, RNF-05).

## 4. Engineering Viewpoint (Ponto de Vista de Engenharia)
**Objetivo:** Infraestrutura e distribuição.

- **Nó de Processamento:** Instância EC2 (t3.medium) rodando motor Python.
- **Nó de Armazenamento:** Banco de Dados Postgres (Instalado na EC2 ou RDS, conforme disponibilidade do Learner Lab).
- **Resiliência:** Uso de transações atômicas por lote de processamento.
- **Atendimento:** RF-004 (Recuperação), RNF-07 (MTTR), RNF-11 (Portabilidade).

## 5. Technology Viewpoint (Ponto de Vista de Tecnologia)
**Objetivo:** Escolhas tecnológicas específicas (AWS Academy).

- **Linguagem:** Python 3.x (Pandas/SQLAlchemy).
- **Infraestrutura:** AWS EC2.
- **Banco de Dados:** PostgreSQL 15.
- **Monitoramento:** Grafana + CloudWatch.
- **Restrição:** Sem AWS Glue ou Redshift (Uso de processamento manual em EC2).
- **Atendimento:** RNF-04 (Postgres), RNF-08 (Segurança), RNF-09 (TLS).

---

## ADR (Architecture Decision Records)

### ADR-01: Uso de UPSERT para Idempotência
- **Contexto:** O pipeline pode falhar e precisar ser reexecutado, ou arquivos podem ser enviados repetidamente (RF-002).
- **Decisão:** Utilizar a cláusula `ON CONFLICT (order_id) DO UPDATE` no PostgreSQL durante a carga.
- **Consequência:** Garante 0 duplicatas (RNF-06), porém exige a existência de uma Unique Constraint no banco de dados.

### ADR-02: Processamento em Chunks (Streaming)
- **Contexto:** A instância EC2 t3.medium possui memória limitada e o arquivo CSV pode crescer (RNF-03).
- **Decisão:** O ingestor lerá o CSV em pedaços (chunks) de 10.000 linhas por vez.
- **Consequência:** Mantém o consumo de RAM abaixo de 512MB (RNF-03), mas aumenta levemente o tempo total de processamento (RNF-02).

### ADR-03: Arquitetura Monolítica em EC2 (Restrição AWS Academy)
- **Contexto:** O ambiente AWS Learner Lab restringe o uso de serviços gerenciados caros como Glue ou Redshift.
- **Decisão:** Todo o processamento (ETL) e o Banco de Dados residirão em instâncias EC2.
- **Consequência:** Redução drástica de custo, maior controle manual sobre a resiliência (RF-004), mas exige maior configuração de monitoramento manual (RNF-10).
