# Requisitos Funcionais (RF)

Este documento detalha as funcionalidades do pipeline de dados Olist, refinadas com base nos modos de falha e riscos identificados no problema de negócio.

## 1. Ingestão e Processamento

### RF-001: Ingestão Diária de Pedidos
- **Descrição:** O sistema deve realizar a ingestão de arquivos CSV contendo aproximadamente 100 mil pedidos originados do marketplace da Olist.
- **Critérios de Aceitação:**
  - O processamento deve ser acionado diariamente.
  - O sistema deve processar arquivos de grande volume sem exceder os limites de memória da instância EC2.
  - Deve ser possível identificar a origem e a data do arquivo processado.

### RF-002: Garantia de Idempotência no Processamento
- **Descrição:** O pipeline deve permitir que o mesmo conjunto de dados seja processado múltiplas vezes sem resultar em registros duplicados ou dados inconsistentes no banco analítico.
- **Critérios de Aceitação:**
  - Implementação de lógica de `UPSERT` baseada na chave única do pedido (`order_id`).
  - O sistema deve verificar se o arquivo já foi processado com sucesso antes de iniciar uma nova carga (a menos que um reprocessamento seja forçado).

## 2. Qualidade e Resiliência

### RF-003: Validação de Integridade dos Dados de Entrada
- **Descrição:** O sistema deve validar a estrutura e o conteúdo dos arquivos CSV para evitar falhas silenciosas ou corrupção de dados no banco analítico.
- **Critérios de Aceitação:**
  - Identificar e logar linhas com campos obrigatórios ausentes ou tipos de dados inválidos.
  - O sistema não deve interromper o processamento total por causa de uma única linha malformada, mas sim isolá-la para auditoria.

### RF-004: Recuperação de Falhas (Checkpoints)
- **Descrição:** O sistema deve ser capaz de retomar o processamento a partir do último ponto de sucesso em caso de interrupções (ex: queda de EC2 ou banco indisponível).
- **Critérios de Aceitação:**
  - Implementação de transações ou marcações de progresso que permitam saber quais lotes foram persistidos.
  - Garantir que a integridade dos dados seja mantida mesmo após uma retomada forçada.

## 3. Observabilidade e Monitoramento

### RF-005: Monitoramento de SLA e Saúde do Pipeline
- **Descrição:** O sistema deve coletar e expor métricas para permitir a visualização da saúde do ETL e o cumprimento do SLA em dashboards do Grafana.
- **Critérios de Aceitação:**
  - Coleta de métricas: tempo total de execução, volume de registros processados (sucesso/erro) e latência da carga.
  - Alerta visual no dashboard caso os dados estejam "stale" (atrasados em relação ao cronograma diário).

### RF-006: Registro de Auditoria e Logs Técnicos
- **Descrição:** Todas as etapas do ciclo de vida do dado (Ingestão -> Transformação -> Carga) devem ser registradas detalhadamente.
- **Critérios de Aceitação:**
  - Logs devem conter timestamp, nível de severidade (INFO, WARN, ERROR) e contexto da operação.
  - Em caso de falha de conexão com a AWS ou Postgres, o log deve capturar a mensagem de erro específica para depuração rápida.
