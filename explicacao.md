## Justificativa da Arquitetura de Data Lake para E-Shop (Versão Atualizada)

Com base nos requisitos do trabalho e nas melhores práticas de arquitetura de dados, sua arquitetura foi estruturada em camadas bem definidas que atendem aos objetivos de integração, processamento e análise de dados da empresa E-Shop. A atualização reflete uma abordagem mais flexível e realista onde a camada Silver pode ser acessada diretamente por consultas analíticas e modelos de machine learning.[1]

### 🔵 Data Sources (Fontes de Dados)

**Componentes:** Sistemas Transacionais OLTP, APIs e Web Services, Streaming em Tempo Real, Arquivos e Logs, Dados Não Estruturados

**Justificativa:** A diversidade de fontes de dados reflete a realidade da E-Shop, que coleta dados de transações, perfis de clientes, produtos, histórico de navegação e avaliações. Esta arquitetura multifontes permite:[1]

- **Sistemas Transacionais OLTP:** Captura de dados de compras, valores e formas de pagamento[1]
- **APIs e Web Services:** Integração com sistemas externos e coleta de dados de clientes[1]
- **Streaming em Tempo Real:** Monitoramento de comportamento de navegação e eventos em tempo real[2][3]
- **Arquivos e Logs:** Armazenamento de logs de navegação e dados de cliques[1]
- **Dados Não Estruturados:** Comentários de produtos, avaliações e imagens[1]

Esta abordagem heterogênea é fundamental para um data lake, que por definição deve aceitar todos os tipos de dados sem pré-processamento.[4][5]

### 🟢 Camada de Ingestão

**Componentes:** Ingestão em Batch (I1) e Ingestão em Streaming (I2)

**Justificativa da Separação:**

**Ingestão em Batch (I1):** Configurada para processar dados de sistemas transacionais, APIs, arquivos e logs em lotes programados. Esta abordagem é ideal quando:[3][6]
- Os dados não precisam ser consumidos em tempo real[3]
- Há grandes volumes acumulados que podem ser processados em horários específicos (ex: diariamente à noite)[3]
- É necessário minimizar o impacto no ambiente de produção[3]
- Integração com sistemas legados que não suportam streaming[6]

**Ingestão em Streaming (I2):** Dedicada a processar dados em tempo real. Esta camada é essencial para:[7][3]
- Capturar eventos de navegação e cliques instantaneamente[7]
- Alimentar sistemas de recomendação com dados frescos[5][8]
- Gerar notificações e análises em tempo real[7]
- Detectar fraudes e anomalias em transações[7]

Esta **arquitetura dual batch-streaming** é conhecida como Arquitetura Lambda, que combina processamento de alta latência (batch) com baixa latência (streaming), permitindo análises históricas profundas e insights em tempo real simultaneamente.[6][3][7]

### 🟡 Camada de Armazenamento - Arquitetura Medalhão

**Estrutura:** Bronze → Silver → Gold

**Justificativa da Escolha:**

A Arquitetura Medalhão foi selecionada por ser um padrão de design reconhecido pela Databricks que organiza logicamente os dados em um lakehouse, melhorando incremental e progressivamente a qualidade dos dados.[9][10][11][2]

#### **Zona Bronze (Raw - Dados Brutos)**
**Características:** Dados Brutos, Formato Original, Auditoria Completa[2]

**Justificativa:**
- **Preservação da integridade:** Armazena dados exatamente como chegam das fontes, sem transformação[9][2]
- **Auditoria e rastreabilidade:** Permite reconstruir todo o pipeline de dados caso necessário[11][2]
- **Linhagem de dados:** Facilita o cumprimento de regulamentações como LGPD[12][9]
- **Reprodutibilidade:** Se uma regra de negócio mudar, é possível reprocessar tudo a partir da origem[11]
- **Schema-on-read:** Permite armazenar dados sem esquema definido previamente[13][4]
- **Recuperação de erros:** Possibilita corrigir problemas em etapas anteriores do pipeline[2]

Para a E-Shop, esta camada armazena transações brutas, logs de navegação originais, avaliações de produtos e dados de clientes sem tratamento.[1]

**Exemplo prático:** Se um algoritmo de limpeza na camada Silver introduzir um bug e remover dados legítimos, a Bronze preserva os dados originais para reprocessamento.[11]

#### **Zona Silver (Refined - Dados Refinados)**
**Características:** Dados Limpos, Validados, Padronizados[2]

**Justificativa:**
- **Qualidade de dados:** Aplicação de limpeza, validação e remoção de duplicatas[14][2]
- **Padronização:** Normalização de formatos e estabelecimento de relacionamentos[10][2]
- **Modelo corporativo:** Estruturação baseada em domínios de negócio (Cliente, Produto, Transação)[15]
- **Conformidade:** Implementação de regras de qualidade e validação[16][14]
- **Acesso intermediário:** Ponto ótimo para cientistas de dados explorarem dados semi-processados[11][2]

Para a E-Shop, esta camada consolida perfis de clientes únicos, produtos validados e transações consistentes, prontos para análise.[1]

**Importância:** A Silver é o ponto de partida ideal para Data Science, pois os dados já estão limpos mas ainda mantêm detalhes granulares que podem ser úteis para exploração e modelagem.[5][2][11]

#### **Zona Gold (Curated - Dados Curados)**
**Características:** Dados Agregados, Modelados, Prontos para Negócio[2]

**Justificativa:**
- **Otimização para consumo:** Dados desnormalizados e agregados para consultas rápidas[9][2]
- **KPIs pré-calculados:** Métricas de vendas, produtos mais vendidos, taxa de conversão[10][2]
- **Modelagem dimensional:** Star schema ou esquemas otimizados para BI[15][9]
- **Terminologia de negócio:** Dados apresentados na linguagem dos usuários finais[11][2]
- **Performance garantida:** Consultas rápidas para dashboards críticos[9][2]

Para a E-Shop, esta camada contém análises de vendas consolidadas, segmentação de clientes e dados prontos para dashboards.[1]

**Benefícios da Arquitetura Medalhão:**[9][2][11]
- Separação de responsabilidades entre equipes (Engenheiros nas camadas Bronze/Silver, Analistas na Gold)[11]
- Simplicidade e intuitividade para comunicação entre áreas técnicas e de negócio[11]
- Qualidade crescente e rastreabilidade em cada etapa[10][2]
- Flexibilidade de acesso em múltiplos níveis de refinamento[2]

### 🟣 Camada de Processamento

**Componentes:** Motor de Processamento Distribuído (P1), Transformações e Limpeza (P2), Enriquecimento de Dados (P3), Jobs ETL/ELT (P4)

**Justificativa:**

**Motor de Processamento Distribuído (Apache Spark/Hadoop - P1):**[17][18][19]
- **Escalabilidade:** Processa grandes volumes de dados distribuindo a carga entre múltiplos nós[18][17]
- **Performance:** Spark utiliza processamento em memória (RAM), exponencialmente mais rápido que acesso a disco[19][18]
- **Versatilidade:** Suporta múltiplos tipos de processamento: batch, streaming, SQL, machine learning[17][19]
- **Compatibilidade:** Funciona com diversos formatos (Parquet, JSON, CSV, Avro)[17]

Para a E-Shop, o Spark é ideal para processar grandes volumes de transações (milhões/dia) e preparar dados para recomendações de produtos.[5][1]

**Transformações e Limpeza (P2):**[14][16]
- **Validação automatizada:** Aplicação de regras de qualidade durante o processamento[16][14]
- **Padronização:** Normalização de dados para análises consistentes[14]
- **Remoção de duplicatas:** Garantia de unicidade de registros[2]
- **Tratamento de nulos:** Estratégias para lidar com valores faltantes[16][14]
- **Outliers:** Detecção e tratamento de anomalias[16]

Alimenta a zona **Silver**.[2]

**Enriquecimento de Dados (P3):**[20][12]
- **Integração de múltiplas fontes:** Combina dados de transações com perfis de clientes[1]
- **Adição de contexto:** Enriquece dados com informações externas (ex: dados demográficos, tendências de mercado)[12]
- **Agregações:** Calcula métricas derivadas para análises[2]
- **Jointures:** Relaciona dados de diferentes domínios[12]

Alimenta a zona **Gold**.[2]

**Jobs ETL/ELT (P4):**[9]
- **Abordagem ELT preferencial:** Extrai, carrega e depois transforma, aproveitando o poder do data lake[9]
- **Orquestração:** Coordena execução de múltiplos jobs em sequência[10][2]
- **Transformações progressivas:** Move dados de Bronze → Silver → Gold aplicando regras incrementais[10][2]
- **Agendamento:** Executa jobs em horários otimizados[6]
- **Monitoramento:** Rastreia sucesso/falha de cada transformação[12]

**Fluxo de Processamento:**
1. BRONZE recebe dados via event-driven (IB → Event → BRONZE)[2]
2. BRONZE → P4 (inicia processamento de jobs ETL)[2]
3. P4 → P2 (aplica transformações e limpeza)[14][16]
4. P2 → SILVER (armazena dados limpos)[2]
5. SILVER → P1 (carrega em motor distribuído)[18]
6. P1 → P3 (aplica enriquecimento)[20][12]
7. P3 → GOLD (armazena dados finais agregados)[2]

### 🔴 Camada de Governança e Segurança

**Componentes:** Catálogo de Metadados (GOV1), Linhagem de Dados (GOV2), Controle de Acesso (GOV3), Criptografia e Compliance (GOV4), Qualidade de Dados (GOV5)

**Justificativa:**

**Catálogo de Metadados (GOV1):**[12][14]
- **Descoberta de dados:** Facilita localizar e entender os dados disponíveis[12]
- **Documentação centralizada:** Registra origem, transformações e destino dos dados[20]
- **Organização:** Indexa e categoriza dados para busca eficiente[4][12]
- **Glossário corporativo:** Define terminologia padronizada[14][12]

Conecta a **STORAGE** (onde os dados residem) e **CONSUMPTION** (onde são acessados).[14][12]

**Linhagem de Dados (GOV2):**[21][22][12]
- **Rastreabilidade end-to-end:** Documenta o fluxo completo dos dados desde a origem[20][12]
- **Compliance regulatório:** Essencial para atender LGPD e GDPR[21][12]
- **Análise de impacto:** Identifica quais relatórios são afetados por mudanças nos dados[22][20]
- **Auditoria:** Permite identificar quando e por quem os dados foram modificados[12]

Conecta a **STORAGE** para rastrear transformações em cada zona da Medalhão.[22][20][12]

**Controle de Acesso (GOV3):**[23][14]
- **Segurança granular:** Define quem pode acessar quais dados[14]
- **Proteção de dados sensíveis:** Essencial para dados de clientes e transações[14]
- **Conformidade:** Atende requisitos de privacidade da LGPD[24][14]
- **Auditoria de acesso:** Registra quem acessou quais dados e quando[14]

Conecta ao **C1 (Ferramentas de BI)** para garantir que apenas usuários autorizados visualizem dados sensíveis.[23][14]

**Criptografia e Compliance (GOV4):**[25][26][24]
- **Proteção em repouso e trânsito:** Criptografia de dados em todo o ambiente[27][25]
- **LGPD:** Requisito obrigatório para proteção de dados pessoais[24][25]
- **Confidencialidade:** Garante que dados só sejam acessados por pessoas autorizadas[25]
- **Integridade:** Valida que dados não foram violados[25]
- **Não-repúdio:** Rastreia quem realizou cada ação[25]

Conecta a **STORAGE** para proteger dados armazenados em todas as zonas.[26][24][25]

**Qualidade de Dados (GOV5):**[28][16][14]
- **Validação contínua:** Monitoramento de completude, precisão e consistência[16][14]
- **Detecção de anomalias:** Identificação de erros e inconsistências[28][14]
- **Métricas de qualidade:** Rastreamento de porcentagem de valores nulos, distribuição de dados[28]
- **Automação:** Processos automatizados de verificação e correção[16][14]
- **Alertas:** Notifica quando qualidade cai abaixo de limites aceitáveis[16]

Conecta ao **PROCESSING** para implementar validações durante transformações.[28][16][14]

**Importância da Governança:** Em um data lake sem governança robusta, há risco de "data swamp" (pântano de dados) onde ninguém entende a qualidade ou origem dos dados.[23][24][14]

### 🟠 Camada de Consumo e Exposição

**Componentes:** Ferramentas de BI e Visualização (C1), Data Science e ML (C2), APIs de Dados (C3), Aplicações Analíticas (C4), Consultas Ad-hoc (C5)

**Justificativa:**

**Ferramentas de BI e Visualização (C1):**[29][30]
- **Power BI, Tableau, Qlik Sense:** Ferramentas líderes para criação de dashboards e relatórios[29]
- **Análise visual:** Permite monitoramento de vendas e preferências de clientes[29][1]
- **Self-service:** Democratiza acesso aos dados para usuários de negócio[4]
- **KPIs em tempo real:** Monitoramento de métricas críticas[29]

Consome dados da **GOLD** (dados pré-agregados e otimizados).[29][1]

**Data Science e Machine Learning (C2):**[8][31][5]
- **Análise preditiva:** Treinamento de modelos de recomendação baseados em histórico[5][1]
- **Segmentação de clientes:** Identificação de padrões de comportamento[32][8]
- **Otimização de marketing:** Personalização de experiências de compra[8][32]
- **Detecção de anomalias:** Fraudes e comportamentos incomuns[8]

**Acesso direto à SILVER:** A nova conexão `SILVER -.-> C2` permite que cientistas de dados acessem dados granulares sem passar pela agregação da Gold. Isto é crítico para:[5][11][2]
- **Exploração:** Investigar padrões em dados detalhados[5]
- **Feature engineering:** Criar variáveis derivadas para modelos[5]
- **Modelagem iterativa:** Testar diferentes abordagens rapidamente[5]

Para a E-Shop, esta camada é crítica para o sistema de recomendação de produtos solicitado.[8][5][1]

**APIs de Dados (C3):**[33][34][35]
- **Exposição controlada:** Permite que aplicações consumam dados via REST APIs[35][33]
- **Integração com sistemas:** Facilita uso dos dados por outras plataformas[36][37]
- **Acesso programático:** Suporta automações e integrações[34][33]
- **Escalabilidade:** Suporta alto volume de requisições[33]

Consome dados da **GOLD** através de endpoints otimizados.[34][35][33]

**Aplicações Analíticas (C4):**[38][5]
- **Aplicações personalizadas:** Sistemas específicos alimentados pelo data lake[5]
- **Tempo real:** Dashboards e alertas em tempo real[5]
- **Mobile e web:** Aplicações para múltiplas plataformas[5]

Integra-se com **C1, C2, C3** para consumir dados.[38][5]

**Consultas Ad-hoc (C5):**[4][5]
- **Exploração livre:** Permite analistas explorarem dados sem restrições[4][5]
- **Flexibilidade:** SQL e outras linguagens para análise exploratória[4]
- **Descoberta:** Busca por padrões não-planejados[5]

**Acesso direto à SILVER:** A nova conexão `SILVER -.-> C5` permite que analistas executem consultas SQL diretamente em dados refinados sem agregação. Justificativa:[4][5][2]
- **Performance intermediária:** Silver é mais rápido que Bronze (dados limpos) mas mantém granularidade de dados[11][2]
- **Análises exploratórias:** Perfeito para perguntas ad-hoc que não justificam pré-agregação na Gold[11][5]
- **Custo-benefício:** Evita manutenção de Gold para cada possível consulta[11][2]

### Decisões Arquiteturais Adicionais

#### **Arquitetura Event-Driven**

O nó "Event" entre a ingestão batch e a zona Bronze representa uma arquitetura orientada a eventos, permitindo:[3][7]
- **Desacoplamento:** Separação entre produtores e consumidores de dados[3]
- **Escalabilidade:** Processamento assíncrono e distribuído[7]
- **Rastreabilidade:** Cada evento é registrado para auditoria[12]
- **Flexibilidade:** Múltiplos consumidores (processadores) podem processar o mesmo evento[7]

#### **Conexão Tracejada IS -.-> BRONZE**

A ingestão em streaming usa conexão tracejada porque:[6][7]
- **Continuidade:** Ingere dados continuamente (não em lotes discretos)[7]
- **Assíncrona:** Não aguarda lote completo para processar[7]
- **Baixa latência:** Ideal para eventos que precisam de resposta imediata[7]

#### **Formato de Armazenamento**

**Justificativa para Parquet como formato preferencial:**[39][40][41]
- **Armazenamento colunar:** Otimizado para consultas analíticas e agregações[40][39]
- **Compressão eficiente:** Reduz drasticamente o tamanho dos arquivos (até 80% em relação a CSV)[39][40]
- **Performance:** Leitura até 100x mais rápida que CSV para análises[40]
- **Compatibilidade:** Padrão da indústria, suportado por todos os frameworks (Spark, Hadoop)[41][39]

**Outros formatos complementares:**[42][39]
- **CSV:** Para dados brutos iniciais e integração com ferramentas simples[39][40]
- **Avro:** Para dados em streaming com esquema evolutivo[42][39]
- **JSON:** Para dados semi-estruturados preservando esquema[39]

### Alinhamento com Requisitos do Trabalho

Sua arquitetura atende completamente aos requisitos especificados:[1]

✅ **Camadas de Ingestão:** Batch e streaming implementadas[3][1]
✅ **Camada de Armazenamento:** Arquitetura Medalhão com formatos apropriados[1][2]
✅ **Camada de Processamento:** Motor distribuído (Spark/Hadoop) para transformações[17][1]
✅ **Camada de Consumo:** BI, ML, APIs e consultas ad-hoc[29][1][5]
✅ **Governança:** Metadados, linhagem, qualidade e segurança[21][12][14]
✅ **Suporte a ML:** Infraestrutura robusta com acesso direto à Silver para Data Science[8][1][5]
✅ **Tipos de dados:** Estruturados, semi-estruturados e não estruturados[4][1]
✅ **Flexibilidade de acesso:** Múltiplos caminhos de consumo (Gold, Silver, APIs)[4][5][2]

### Fluxos de Dados Principais

#### **Fluxo 1: Análise de Negócio (Executivos/Analistas)**
SOURCES → BATCH/STREAMING → BRONZE → PROCESSING → SILVER → GOLD → **C1 (BI e Dashboards)**

- Dados pré-agregados e otimizados para performance máxima[29][2]

#### **Fluxo 2: Análise Exploratória (Data Scientists)**
SOURCES → BATCH/STREAMING → BRONZE → PROCESSING → **SILVER -.-> C2 (ML/Data Science)**

- Dados refinados mantendo granularidade para modelagem[11][5]

#### **Fluxo 3: Consultas Ad-hoc (Analistas)**
SOURCES → BATCH/STREAMING → BRONZE → PROCESSING → **SILVER -.-> C5 (Consultas SQL)**

- Acesso rápido a dados limpos para exploração[4][5]

#### **Fluxo 4: Integração Externa**
SOURCES → BATCH/STREAMING → BRONZE → PROCESSING → GOLD → **C3 (APIs de Dados)**

- Exposição controlada de dados via APIs REST[33][34]

#### **Fluxo 5: Recomendação de Produtos (Caso de Uso Principal)**
SOURCES (Navegação + Compras) → STREAMING → BRONZE → PROCESSING → SILVER -.-> **C2 (ML/Recomendação)**

- Dados frescos em tempo real alimentam modelo de recomendação[8][1][5]

### Vantagens Competitivas da Arquitetura

Esta arquitetura oferece à E-Shop:[8][1][5]

**Escalabilidade:** Cresce conforme o volume de dados aumenta, suportando crescimento de 10x ou mais[18][4]

**Flexibilidade:** Suporta novos tipos de dados sem reestruturação completa[4][5]

**Governança robusta:** Atende LGPD, garante qualidade dos dados e rastreabilidade completa[24][12][14]

**Insights em tempo real:** Combinação de batch (histórico profundo) e streaming (eventos imediatos)[3][7]

**Personalização:** Base para recomendações e marketing direcionado, essencial para retenção de clientes[32][8]

**Custo-efetivo:** Armazenamento em data lake é mais econômico que data warehouse, com flexibilidade aumentada[18][23]

**Democratização:** Acesso controlado mas amplo aos dados em múltiplos níveis (Gold, Silver, APIs)[4][5]

**Inovação acelerada:** Data Scientists têm acesso rápido a dados granulares para experimentação rápida[11][5]

### Resumo das Mudanças Implementadas

A adição da conexão `SILVER -.-> C5 & C2` (tracejada) reflete uma prática recomendada moderna onde:[11][2]

1. **C2 (Data Science):** Acesso direto à Silver permite trabalhar com dados granulares, ideal para treinamento de modelos e experimentação[5][11]
2. **C5 (Consultas Ad-hoc):** Acesso direto à Silver oferece ponto ótimo entre performance e granularidade para análises exploratórias[4][5]

Esta flexibilidade não compromete a segurança (GOV3 ainda controla acesso) nem a qualidade (GOV5 valida Silver), apenas permite que dados refinados sejam consumidos diretamente sem necessidade de agregação prévia na Gold.[11][2]

Esta arquitetura representa uma solução moderna, escalável, flexível e aderente às melhores práticas da indústria para implementação de data lakes em ambientes de e-commerce.[9][5][2][11]

[1](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/134068334/a029afc8-9e1c-4980-b379-8cfe9a6bba04/Trabalho.pdf)
[2](https://www.robertodiasduarte.com.br/entenda-a-medallion-architecture-em-data-lakehouse/)
[3](https://blog.grancursosonline.com.br/ingestao-de-dados-conceitos/)
[4](https://wevy.cloud/blog-data-lake/)
[5](https://www.databricks.com/discover/data-lakes)
[6](https://kondado.com.br/blog/blog/2022/11/11/o-que-e-ingestao-de-dados/)
[7](https://www.programmers.com.br/blog/stream-data-lake-talvez-a-resposta-seja-arquitetura-lambda/)
[8](https://www.salesforce.com/br/blog/data-lake/)
[9](https://www.databricks.com/br/glossary/medallion-architecture)
[10](https://www.robertodiasduarte.com.br/guia-da-metodologia-medallion-architecture-para-data-lakehouse/)
[11](https://blog.dsacademy.com.br/arquitetura-medalhao-o-guia-definitivo-para-organizar-o-data-lakehouse-fundamentos/)
[12](https://www.astera.com/pt/type/blog/metadata-management/)
[13](https://www.cienciaedados.com/data-lake-a-evolucao-do-armazenamento-e-processamento-de-dados/)
[14](https://www.alura.com.br/artigos/governanca-de-dados-data-lake)
[15](https://www.reddit.com/r/dataengineering/comments/1ddkse6/data_modeling_in_medallion_architecture/)
[16](https://blog.dsacademy.com.br/como-controlar-verificacoes-de-qualidade-de-dados-no-data_lake/)
[17](https://hadoop.com.br/tecnologias/processamento/apache-spark)
[18](https://aws.amazon.com/pt/compare/the-difference-between-hadoop-vs-spark/)
[19](https://aws.amazon.com/pt/what-is/apache-spark/)
[20](https://sol.sbc.org.br/index.php/sbbd/article/download/37270/37053/)
[21](https://www.alteryx.com/pt-br/glossary/data-lineage)
[22](https://blog.dsacademy.com.br/linhagem-de-dados-tecnicas-e_exemplos/)
[23](https://www.objective.com.br/insights/data-lake/)
[24](https://repositorio.ufpe.br/handle/123456789/49411)
[25](https://blog.tripla.com.br/criptografia-de-dados-na-lgpd/)
[26](https://repositorio.ufsc.br/bitstream/handle/123456789/247351/PGCC1233-D.pdf?sequence=1)
[27](https://lexcompliance.com.br/lgpd/)
[28](https://www.programaria.org/qualidade-de-dados-o-que-e-como-medir-e-garantir/)
[29](https://ncs.net.br/insights/quais-sao-as-ferramentas-de-bi-mais-conhecidas-e-como-escolher)
[30](https://translate.google.com/translate?u=https%3A%2F%2Fwww.integrate.io%2Fblog%2Ftop-bi-visualization-tools%2F&hl=pt&sl=en&tl=pt&client=srp)
[31](https://www.redhat.com/en/topics/data-storage/what-is-a-data-lake)
[32](https://www.totvs.com/blog/inteligencia-dados/data-lake/)
[33](https://learn.microsoft.com/pt-br/rest/api/datalakeanalytics/)
[34](https://docs.aws.amazon.com/pt_br/lake-formation/latest/dg/aws-lake-formation-api-aws-lake-formation-api-settings.html)
[35](https://learn.microsoft.com/pt-br/rest/api/storageservices/data-lake-storage-gen2)
[36](https://docs.oracle.com/pt-br/solutions/data-platform-lakehouse/index.html)
[37](https://site.sauter.digital/?p=431)
[38](https://encord.com/blog/data-lake-guide/)
[39](https://www.reddit.com/r/dataengineering/comments/1cbx8bb/preferred_file_format_and_why_csv_json_parquet/)
[40](https://www.datacamp.com/pt/tutorial/apache-parquet)
[41](https://www.ibm.com/docs/pt-br/db2/12.1.x?topic=warehouse-supported-file-formats)
[42](https://translate.google.com/translate?u=https%3A%2F%2Fsqream.com%2Fblog%2Favro-vs-parquet-which-file-format-is-right-for-your-data%2F&hl=pt&sl=en&tl=pt&client=srp)
