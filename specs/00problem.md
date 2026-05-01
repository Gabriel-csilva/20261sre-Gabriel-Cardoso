# Problema

## Contexto

O negócio Olist precisa processar ~100 mil pedidos do marketplace, carregar em um banco analítico e gerar dashboards diários. O ETL deve ser confiável: idempotente, observável e resiliente a falhas parciais. Não podemos perder dados, nem duplicá-los, nem falhar silenciosamente.

## Canvas de Modelagem

### 1. Stakeholders

- Operação Olist (negócio)
- Time de dados
- Clientes internos do dashboard
- Plataforma / SRE

### 2. Fluxos Críticos

- Ingestão diária CSV → Postgres
- Consulta de dashboards Grafana
- Observação de SLA

### 3. Modos de Falha

- Arquivo corrompido ou parcial
- Reprocesso duplicando linhas
- Queda de EC2 durante o run
- Banco indisponível

### 4. Riscos Sistêmicos

- Perda silenciosa de linhas
- Dashboard mostrando dado stale
- Custo AWS explodindo
- Dívida técnica de IA mal revisada
