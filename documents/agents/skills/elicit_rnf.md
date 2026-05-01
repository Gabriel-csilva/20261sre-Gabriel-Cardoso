# Skill: Elicitação de Requisitos Não-Funcionais (RNF)

Esta skill orienta a extração e documentação de Requisitos Não-Funcionais para o projeto Olist, focando nos pilares de SRE: confiabilidade, performance, observabilidade e segurança.

## Objetivo
Identificar as restrições e qualidades do sistema (o "como" ele deve operar) para garantir que o pipeline de dados seja robusto e atenda às expectativas de nível de serviço (SLAs).

## Diretrizes de Elicitação

Ao elicitar requisitos não-funcionais, utilize os seguintes pilares:

### 1. Confiabilidade e Disponibilidade (SRE)
- Qual o tempo máximo aceitável para o pipeline estar fora do ar?
- Qual a taxa de erro tolerável antes de disparar um alerta?
- Como o sistema deve se comportar em falhas de infraestrutura (resiliência)?

### 2. Performance e Escalabilidade
- Qual o tempo máximo para processar o lote diário de 100k pedidos?
- O sistema suportaria um aumento de 10x no volume de dados sem refatoração?
- Quais os limites de consumo de CPU/Memória na instância EC2?

### 3. Observabilidade
- Qual a retenção mínima dos logs de erro?
- Qual a frequência de atualização das métricas no Grafana (ex: a cada 1 minuto)?
- As métricas devem ser granulares o suficiente para identificar a etapa exata de falha?

### 4. Segurança e Conformidade
- Como as credenciais do banco e da AWS serão protegidas (ex: Environment Variables, Secrets Manager)?
- Os dados em repouso no Postgres precisam de criptografia?
- Quem terá acesso aos logs e dashboards?

### 5. Idempotência e Integridade
- Em caso de falha, o tempo de recuperação (MTTR) deve ser de quanto tempo?
- Como garantir que a integridade referencial seja mantida mesmo em cargas parciais?

## Formato do Requisito
Cada requisito deve ser mensurável sempre que possível:
- **ID:** RNF-XXX
- **Nome:** Título curto (ex: Performance de Carga).
- **Descrição:** Explicação detalhada da restrição ou qualidade.
- **Métrica/Alvo:** O valor quantitativo que define o sucesso (ex: < 30 minutos).

## Exemplo de Requisito Não-Funcional

**RNF-001: Performance de Processamento**
- **Descrição:** O tempo total do processo de ETL (Extração, Transformação e Carga) para um arquivo de 100k linhas não deve impactar a janela operacional.
- **Métrica/Alvo:** Processamento concluído em menos de 15 minutos em uma instância t3.medium.

---
*Esta skill deve ser utilizada para preencher e revisar o arquivo `02_non_functional_requirements.md`.*
