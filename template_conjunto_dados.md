# Infoagro | Documentação dos Painéis (BigQuery + Looker Studio)

Painéis interativos baseados em dados do **Google BigQuery** (projeto/dataset únicos) e publicados no **Looker Studio**.

**Painéis**
- ✅ Histórico de Clima (INMET)
- ✅ Produtividade Agrícola (EMATER)
- 🟡 Análise de Solos (em construção)

---

## Sumário
- [1. Arquitetura](#1-arquitetura)
- [2. Repositório de dados (BigQuery)](#2-repositório-de-dados-bigquery)
  - [2.1 Convenções de nomes](#21-convenções-de-nomes)
  - [2.2 Tabelas e views](#22-tabelas-e-views)
- [3. Conjuntos de dados](#3-conjuntos-de-dados)
  - [3.1 Clima (INMET)](#31-conjunto-de-dados-clima-inmet)
  - [3.2 Produtividade Agrícola](#32-conjunto-de-dados-produtividade-agrícola)
  - [3.3 Geografia (RJ Municípios)](#33-conjunto-de-dados-geografia-rj-municípios)
  - [3.4 Solos (RJ) - em construção](#34-conjunto-de-dados-solos-rj---em-construção)
- [4. Painéis](#4-painéis)
  - [4.1 Painel: Histórico de Clima](#41-painel-histórico-de-clima)
  - [4.2 Painel: Produtividade Agrícola](#42-painel-produtividade-agrícola)
  - [4.3 Painel: Análise de Solos (em construção)](#43-painel-análise-de-solos-em-construção)
- [5. Regras de cálculo (oficiais)](#5-regras-de-cálculo-oficiais)
- [6. Limitações e cuidados](#6-limitações-e-cuidados)

---

## 1. Arquitetura

### Visão macro
**BigQuery (camada de dados) → Looker Studio (fontes) → Painéis (visualizações)**

- **BigQuery**
  - Projeto: `infoagro`
  - Dataset: `infoagro`
  - Todas as tabelas ou views ou materialized views consumidas pelos painéis estão nesse caminho.

- **Looker Studio**
  - Fontes conectadas diretamente ao BigQuery (tabelas/views/materialized view).
  - Atualização exibida na conexão: **12 horas**.

### Fluxo padrão
1) BigQuery armazena o dado  
2) Looker Studio cria uma Fonte por domínio  
3) Painéis consomem as Fontes e aplicam filtros, KPIs e gráficos  

---

## 2. Repositório de dados (BigQuery)

### 2.1 Convenções de nomes
- `*_temp` = staging / tabelas temporárias / pré-processamento  
- `vw_*` = views de apoio / padronização (quando usadas)  
- Tabelas “geo” contêm `geometry` para mapas e **chaves de join** (ideal: `codigo_ibge`)  

### 2.2 Tabelas e views (dataset `infoagro`)
> Todas no padrão: `infoagro.infoagro.<tabela_ou_view>`

**Clima**
- `historico_clima_inmet` ✅ (tabela usada no painel)
- `historico_clima_inmet_temp`
- `vw_historico_clima_inmet_temp`

**Produção / Produtividade**
- `produtividade_rural_municipios_rj_2021_2024` ✅ (tabela usada no painel)
- `produtividade_rural_rj`
- `rj_municipios` ✅ (geoespacial, usado nos mapas)

**Solos (painel ainda não publicado)**
- `bdsolos_riodejaneiro_analisefisica_temp`
- `bdsolos_riodejaneiro_analisequimica_temp`
- `bdsolos_riodejaneiro_ataque_sulfurico_temp`
- `bdsolos_riodejaneiro_descricao..._temp`
- `bdsolos_riodejaneiro_observacoes_temp`

---

## 3. Conjuntos de dados

## 3.1 Conjunto de dados: Clima (INMET)

### 1) Descrição
Base histórica climática do INMET, com registros por estação, incluindo tempo (data/hora), localização (lat/long) e variáveis meteorológicas.

**Tabela consumida no painel**
- `infoagro.infoagro.historico_clima_inmet`

### 2) Granularidade
- **Temporal:** horária (há campos `data` e `hora`)
- **Espacial:** estação (com latitude/longitude) + recortes por UF/região  
- **Agregações no painel:** podem ser mensais/anuais via filtros e gráficos

### 3) Campos principais
**Identificação / localização**
- `estacao`
- `codigo_wmo`
- `uf`
- `regiao`
- `latitude`, `longitude`, `latlong`
- `altitude`

**Tempo**
- `data`
- `hora`
- `ano` (campo derivado no Looker)
- `mês` (campo derivado no Looker)

**Variáveis meteorológicas (principais grupos)**
- Precipitação (`precipitacao_total_horaria_mm`)
- Temperatura (campo calculado: `temperatura_media` e variações min/max)
- Pressão (campo calculado: `pressao_atm_media` e variações)
- Umidade (`umidade_rel_media`)
- Vento (`vento_velocidade_horaria_ms`)

### 4) Atualização
- **Automática:** a cada **12 horas** (conforme configuração da conexão no Looker Studio)

### 5) Limitações
- Lacunas por estação/período (dado ausente ou incompleto)
- Comparações entre estações exigem atenção a cobertura e qualidade
- Se o painel estiver agregando para “diário/mensal”, cuidado com a função de agregação (média vs soma)

---

## 3.2 Conjunto de dados: Produtividade Agrícola

### 1) Descrição
Base anual de produtividade agrícola por **região/município/cultura**, com métricas de produção, área, produtores, preço e faturamento.

**Tabela principal**
- `infoagro.infoagro.produtividade_rural_municipios_rj_2021_2024`

### 2) Granularidade
- **Temporal:** anual (`ano`)
- **Espacial:** município (RJ) + `regiao`
- **Dimensão:** `cultura`

### 3) Campos principais (conforme fonte do Looker)
Dimensões:
- `ano`
- `regiao`
- `municipio`
- `estado`
- `cultura`

Métricas:
- `area_colhida_ha`
- `producao_colhida_t`
- `produtores`
- `preco_kg`
- `produtividade_t_ha`
- `faturamento_bruto`

### 4) Atualização
- **Automática:** a cada **12 horas**

### 5) Limitações
- `produtores`: dependendo do grão (por cultura), pode haver dupla contagem se somar sem critério
- `preco_kg` e `produtividade_t_ha`: não devem ser simplesmente somados. Ver regra na seção 5.

---

## 3.3 Conjunto de dados: Geografia (RJ Municípios)

### 1) Descrição
Base geoespacial para mapas (polígonos municipais) com campo `geometry` e chaves para relacionamento.

**Tabela**
- `infoagro.infoagro.rj_municipios`

### 2) Granularidade
- Espacial: município

### 3) Campos principais
- `codigo_ibge` (recomendado como chave)
- `municipio`
- `municipio_uf`
- `geometry` (Geoespacial)

> A fonte no Looker também pode conter métricas duplicadas do fato. O recomendado é usar `rj_municipios` como **dimensão geo** (geometry + chaves) e a tabela de produtividade como **fato**.

### 4) Atualização
- conforme atualização do dataset / conexão (12h)

### 5) Limitações
- Join por nome pode falhar (acentos/abreviações). Prefira `codigo_ibge`.

---

## 3.4 Conjunto de dados: Solos (RJ) - em construção

### 1) Descrição
Conjunto planejado para análise de solo por localidade (RJ), cobrindo análises físicas, químicas, ataque sulfúrico e descrições/observações.

### 2) Granularidade
- Temporal: por coleta/campanha (se existir no dado)
- Espacial: município e/ou ponto (latitude/longitude, se existir)

### 3) Tabelas
- `bdsolos_riodejaneiro_analisefisica_temp`
- `bdsolos_riodejaneiro_analisequimica_temp`
- `bdsolos_riodejaneiro_ataque_sulfurico_temp`
- `bdsolos_riodejaneiro_descricaomorfologica_temp`
- `bdsolos_riodejaneiro_descricaomorfologica_cor_temp`
- `bdsolos_riodejaneiro_observacoes_temp`


---

## 4. Painéis

## 4.1 Painel: Histórico de Clima

### Objetivo
Entrega de leitura rápida do histórico climático por estação/região/UF, com filtros por mês/ano e parâmetros (precipitação, temperatura, pressão, umidade, vento).

### Fonte de dados
- `historico_clima_inmet` (BigQuery)

### Principais KPIs (cards)
- Precipitação (mm)
- Temperatura média (°C)
- Pressão atmosférica (hPa)
- Umidade do ar (%)
- Velocidade do vento (m/s)

### Filtros
- Estações
- Regiões
- Mês
- Ano
- Campo de busca por município (para “estação mais próxima”, quando aplicável no layout)

### Visualizações
- Série temporal (linha/área) do parâmetro selecionado
- Variações mínima/média/máxima (quando aplicável)
- Tabela de estações + municípios mais próximos
- Mapa com pontos das estações

---

## 4.2 Painel: Produtividade Agrícola

### Objetivo
Consolidar produção agrícola e permitir drill-down de visão executiva → regional → cultura → município → histórico.

### Fontes de dados
- Fato: `produtividade_rural_municipios_rj_2021_2024`
- Geo: `rj_municipios` (geometry)

### Páginas / navegação (no layout)
- Resumo
- Detalhamento Regional
- Detalhamento Culturas
- Histórico

### KPIs (cards)
- Área colhida (ha)
- Nº de produtores
- Produção colhida (t)
- Produtividade (t/ha)
- Preço médio (R$/kg)
- Faturamento (R$)

### Filtros
- Ano
- Região
- Município
- Cultura (em páginas específicas)

### Visualizações
- Donut (participação por região/município/cultura)
- Gráfico combinado (barras + linha) por região
- Mapa coroplético por município (faturamento/produtividade)
- Dispersão (bolhas): produtividade vs nº produtores (tamanho por faturamento)
- Tabelas dinâmicas (drill de região/produção/cultura/faturamento)

---

## 4.3 Painel: Análise de Solos (em construção)

### Status
🟡 Ainda não publicado. Aguardando bases no BigQuery e seguirão o mesmo caminho de consumo no Looker Studio.

### Entregáveis planejados - to be defined (tbd)
- Página “Resumo” (KPIs de qualidade do solo por recorte)
- Página “Mapa” (município/ponto com intensidade por parâmetro)
- Página “Parâmetros” (distribuição, boxplot/histograma)
- Página “Histórico” (se houver data de coleta)

### Dependências
- Definir modelo de consumo
- Padronizar chaves geográficas (municipio / codigo_ibge)
- Normalizar unidades e parâmetros

---

## 5. Regras de cálculo

**Produtividade (t/ha)**  
✅ Recomendado: média

⚠️ Evitar:
- Soma de produtividade_t_ha

**Preço médio (R$/kg)**
✅ Recomendado: média

⚠️ Evitar:
- Soma de preco_kg

**Faturamento bruto (R$)**
- `Faturamento bruto = SUM(faturamento_bruto)`

---

## 6. Limitações e cuidados

- **Dados ausentes**: clima pode ter buracos por estação e períodos.
- **Join por município**: preferir `codigo_ibge`.

