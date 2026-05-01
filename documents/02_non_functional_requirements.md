# Requisitos Não-Funcionais (RNF) - Detalhamento Técnico

Este documento define as restrições técnicas, atributos de qualidade e as responsabilidades de cada requisito para o pipeline de dados Olist, estruturados conforme a norma ISO 25010.

## 1. Adequação Funcional (Functional Suitability)

- **RNF-01: Completitude dos Dados Ingeridos**
  - **Descrição:** O sistema deve garantir que o volume de dados carregados no Postgres corresponda exatamente ao volume de dados válidos no CSV original.
  - **Responsabilidade:** Prevenir a perda silenciosa de dados (Silent Data Loss). Este requisito obriga o desenvolvedor a implementar logs de contagem de linhas no início e no fim do processo e uma rotina de conciliação para garantir que nenhum pedido seja "esquecido" durante a transformação.
  - **SLI:** Razão entre (linhas no Postgres) / (linhas válidas no CSV).
  - **SLO:** 100% de correspondência para registros válidos em cada execução.
  - **Prioridade:** Must-Have.

## 2. Eficiência de Performance (Performance Efficiency)

- **RNF-02: Tempo de Processamento do Lote (Latência)**
  - **Descrição:** O pipeline deve processar o lote diário de 100k pedidos dentro de 20 minutos.
  - **Responsabilidade:** Garantir a pontualidade da entrega (Timeliness). A responsabilidade deste RNF é assegurar que o dado esteja disponível para os tomadores de decisão no início do dia útil, evitando que atrasos técnicos gerem impacto no negócio Olist.
  - **SLI:** Tempo total de execução (segundos) por lote de 100k linhas.
  - **SLO:** < 20 minutos em instância t3.medium.
  - **Prioridade:** Must-Have.

- **RNF-03: Eficiência de Memória**
  - **Descrição:** O processo de ETL deve operar com consumo estável de memória (< 512MB).
  - **Responsabilidade:** Estabilidade da infraestrutura. Ao limitar o uso de RAM, este requisito impede que o script cause falhas de "Out of Memory" (OOM) na instância EC2, garantindo que o sistema seja capaz de processar arquivos maiores através de técnicas de leitura em pedaços (chunks).
  - **SLI:** Pico de consumo de RAM (MB) durante o processamento.
  - **SLO:** < 512MB de RAM utilizados pelo processo Python.
  - **Prioridade:** Should-Have.

## 3. Compatibilidade (Compatibility)

- **RNF-04: Compatibilidade de Esquema Postgres**
  - **Descrição:** O sistema deve ser compatível com as versões 13+ do PostgreSQL.
  - **Responsabilidade:** Portabilidade de Banco de Dados. Garante que o código não utilize extensões proprietárias ou versões obsoletas do SQL, facilitando futuras atualizações de versão do banco ou migrações para serviços gerenciados como AWS RDS.
  - **SLI:** Sucesso de execução de scripts DDL e DML.
  - **SLO:** 100% de compatibilidade com sintaxe SQL padrão Postgres.
  - **Prioridade:** Must-Have.

## 4. Usabilidade (Usability)

- **RNF-05: Tempo de Resposta do Dashboard Grafana**
  - **Descrição:** Consultas do Grafana devem carregar em menos de 3 segundos.
  - **Responsabilidade:** Eficácia do Monitoramento. Se o dashboard for lento, a equipe de SRE demorará mais para identificar problemas. Este requisito responsabiliza a modelagem de dados (uso de índices e tabelas otimizadas) pela agilidade na observação do SLA.
  - **SLI:** Latência de carregamento dos painéis de SLA.
  - **SLO:** < 3 segundos para 95% das consultas (p95).
  - **Prioridade:** Should-Have.

## 5. Confiabilidade (Reliability)

- **RNF-06: Taxa de Sucesso da Idempotência**
  - **Descrição:** Reprocessamentos não devem gerar duplicatas.
  - **Responsabilidade:** Integridade Referencial. É o requisito mais crítico para SRE neste projeto. Garante que, caso o script precise rodar novamente (após uma falha parcial), o banco de dados não seja corrompido com dados redundantes, preservando a verdade única do dado analítico.
  - **SLI:** Quantidade de registros duplicados após re-run de arquivo idêntico.
  - **SLO:** 0 duplicatas.
  - **Prioridade:** Must-Have.

- **RNF-07: Tempo de Recuperação (MTTR)**
  - **Descrição:** Retomada do processamento em menos de 5 minutos após falha.
  - **Responsabilidade:** Resiliência Operacional. Define a necessidade de automação na recuperação de desastres. Se a EC2 cair, o sistema deve estar pronto para reiniciar e continuar do último checkpoint sem intervenção manual demorada.
  - **SLI:** Tempo entre a detecção da falha e a reinicialização bem-sucedida do pipeline.
  - **SLO:** < 5 minutos.
  - **Prioridade:** Should-Have.

## 6. Segurança (Security)

- **RNF-08: Proteção de Credenciais**
  - **Descrição:** Nenhuma credencial deve estar exposta no código fonte.
  - **Responsabilidade:** Prevenção de Vazamento de Dados. Este requisito obriga o uso de boas práticas de segurança, como variáveis de ambiente ou Secrets Manager, protegendo o acesso ao banco analítico contra acessos não autorizados via repositório de código.
  - **SLI:** Número de segredos encontrados em texto puro no repositório.
  - **SLO:** 0 segredos.
  - **Prioridade:** Must-Have.

- **RNF-09: Integridade de Dados em Trânsito**
  - **Descrição:** Conexão entre EC2 e Postgres via SSL/TLS.
  - **Responsabilidade:** Confidencialidade da Rede. Garante que os dados dos pedidos (que podem conter informações sensíveis do negócio) não sejam interceptados enquanto viajam entre a aplicação e o banco de dados.
  - **SLI:** Uso de SSL/TLS ativo na conexão.
  - **SLO:** 100% das conexões via túnel criptografado.
  - **Prioridade:** Must-Have.

## 7. Manutenibilidade (Maintainability)

- **RNF-10: Analisabilidade de Falhas (Log Coverage)**
  - **Descrição:** Logs devem possuir contexto suficiente para diagnósticos rápidos.
  - **Responsabilidade:** Diagnóstico (Root Cause Analysis). Reduz o tempo de investigação da equipe de engenharia. Um erro sem log custa horas de debug; um erro bem documentado permite correção imediata.
  - **SLI:** Percentual de erros que possuem traceback e ID de correlação no log.
  - **SLO:** 100% dos erros fatais registrados com contexto.
  - **Prioridade:** Should-Have.

## 8. Portabilidade (Portability)

- **RNF-11: Portabilidade de Ambiente (Dockerization)**
  - **Descrição:** Pipeline executável via Docker.
  - **Responsabilidade:** Reprodutibilidade do Ambiente. Garante que o comportamento do sistema seja idêntico no ambiente de desenvolvimento do engenheiro e na produção (EC2), eliminando o clássico erro "na minha máquina funciona".
  - **SLI:** Tempo para realizar o deploy em um novo ambiente Linux.
  - **SLO:** < 10 minutos para primeira execução bem-sucedida.
  - **Prioridade:** Could-Have.

---

## Matriz de Requisitos Não-Funcionais (Resumo)

| ID | Atributo ISO 25010 | Responsabilidade Principal | SLI (Indicador) | SLO (Objetivo) | Prioridade |
|:---|:---|:---|:---|:---|:---|
| RNF-01 | Adequação Funcional | Integridade do Fluxo | Row Count Match | 100% | Must |
| RNF-02 | Performance | Pontualidade da Entrega | Tempo de Execução | < 20 min | Must |
| RNF-03 | Performance | Estabilidade da Infra | Pico de RAM | < 512 MB | Should |
| RNF-04 | Compatibilidade | Longevidade do Banco | SQL Compatibility | 100% | Must |
| RNF-05 | Usability | Agilidade de Observação | Latência Dashboard | < 3s (p95) | Should |
| RNF-06 | Reliability | Verdade Única do Dado | Registros Duplicados | 0 | Must |
| RNF-07 | Reliability | Recuperação Automática | MTTR | < 5 min | Should |
| RNF-08 | Segurança | Proteção de Segredos | Hardcoded Secrets | 0 | Must |
| RNF-09 | Segurança | Privacidade de Trânsito | TLS Connection | 100% | Must |
| RNF-10 | Manutenibilidade | Facilidade de Debug | Error Traceback | 100% | Should |
| RNF-11 | Portabilidade | Padronização de Dev/Ops | Deploy Time | < 10 min | Could |
