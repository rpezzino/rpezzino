# Documentação dos Painéis (Funcional + Técnica)

## Padrão (como ler esta documentação)
Cada painel tem 2 blocos:
- **Funcional**: o que resolve, para quem, como usa
- **Técnico**: fontes, joins, campos calculados, regras, atualização, riscos

---

# 1) Painel: Histórico de Clima (INMET)

## 1.1 Documentação Funcional
### Objetivo
Permitir análise do **histórico climático** por estação/UF/região/tempo, com visão rápida (KPIs) e visão analítica (séries temporais + mapa + tabela).

### Público-alvo
Gestores, analistas e usuários que precisam consultar padrões de clima (chuva, temperatura, etc.) para planejamento/monitoramento.

### Perguntas que o painel responde
- Como evoluiu a precipitação/temperatura/umidade ao longo do tempo?
- Quais estações têm maiores/menores valores no período?
- Qual estação atende melhor uma região/município (aproximação)?

### Filtros (inputs)
- Estações
- Regiões
- Mês
- Ano
- Busca por município (quando aplicável no layout)

### KPIs (outputs principais)
- Precipitação (mm)
- Temperatura média (°C)
- Pressão atmosférica (hPa)
- Umidade do ar (%)
- Velocidade do vento (m/s)

### Visualizações e navegação
- Série temporal do parâmetro selecionado (linha/área)
- Comparação mínima/média/máxima (quando aplicável)
- Tabela de estações + municípios mais próximos
- Mapa com pontos das estações

### Regras de uso (para evitar interpretação errada)
- Chuva: analisar por **soma** no período
- Temperatura/pressão/umidade/vento: usar **média** no período

---

## 1.2 Documentação Técnica
### Fontes e tabelas
- Fonte Looker: `historico_clima_inmet`
- BigQuery: `infoagro.infoagro.historico_clima_inmet`

### Grão e agregação
- Dado horária (`data` + `hora`)
- Agregações por período são feitas no Looker:
  - chuva = soma
  - temperatura/pressão/umidade/vento = média/min/max

### Campos calculados (recomendado manter no Data Source)
- `ano` = YEAR(`data`)
- `mês` = MONTH(`data`)

### Atualização
- automática: 12 horas

### Performance (prático)
- Evitar filtros muito “abertos” com série horária longa (muitos anos sem recorte)
- Recomenda-se padrão de filtro inicial (ex.: ano atual) para reduzir carga

### Riscos / limitações técnicas
- Lacunas por estação/período
- Direção do vento não deve ser média simples (usar apenas como informação de apoio)

---

# 2) Painel: Produtividade Agrícola

## 2.1 Documentação Funcional
### Objetivo
Entregar visão executiva e analítica da **produção e produtividade agrícola** no RJ com drill-down por região, município e cultura.

### Público-alvo
Gestores/analistas do projeto Infoagro e usuários que consultam desempenho agrícola.

### Perguntas que o painel responde
- Quais regiões/municípios lideram em faturamento e produtividade?
- Como está a produção por cultura no recorte selecionado?
- Como a série histórica evolui por região/cultura?
- Quais municípios se destacam no mapa e nos rankings?

### Páginas (layout)
- Resumo
- Detalhamento Regional
- Detalhamento Culturas
- Histórico

### Filtros (inputs)
- Ano
- Região
- Município
- Cultura (em páginas específicas)

### KPIs (outputs)
- Área colhida (ha)
- Nº de produtores
- Produção colhida (t)
- Produtividade (t/ha)
- Preço médio (R$/kg)
- Faturamento (R$)

### Visualizações
- Donut de participação (por região/município/cultura)
- Gráfico combinado (barras + linha) por região
- Mapa coroplético por município (faturamento/produtividade)
- Dispersão (bolhas): produtividade vs nº produtores (tamanho por faturamento)
- Tabelas dinâmicas (drill)

### Regras de leitura
- Produtividade/Preço médio devem ser ponderados (não somar)
- Produtores pode duplicar se estiver por cultura (interpretar com cuidado)

---

## 2.2 Documentação Técnica
### Fontes e tabelas
- Fato: `produtividade_rural_municipios_rj_2021_20...`
  - BigQuery: `infoagro.infoagro.produtividade_rural_municipios_rj_2021_20...`
- Geo: `rj_municipios`
  - BigQuery: `infoagro.infoagro.rj_municipios`

### Join / relacionamento (mapas)
- Recomendado: `codigo_ibge` (quando disponível)
- Alternativa: `municipio_uf` padronizado

### Regras oficiais (campos calculados recomendados no Data Source)
**1) Produtividade calculada (t/ha)**
- `produtividade_calc_t_ha = SUM(producao_colhida_t) / SUM(area_colhida_ha)`

**2) Preço médio calculado (R$/kg)**
- `preco_medio_rs_kg = SUM(faturamento_bruto) / (SUM(producao_colhida_t) * 1000)`

> Se você usar `produtividade_t_ha` e `preco_kg` direto, o Looker pode aplicar “Soma” e quebrar KPI.

### Atualização
- automática: 12 horas

### Performance
- Preferir agregações por ano/região antes de ir ao nível de município/cultura com muito detalhe
- Mapas com `geometry` podem ficar pesados se rodar sem filtro

### Riscos / limitações técnicas
- `produtores` pode duplicar dependendo do grão (por cultura)
- `preco_kg` e `produtividade_t_ha` nunca devem ser somados

---

# 3) Painel: Análise de Solos (em construção)

## 3.1 Documentação Funcional (planejada)
### Objetivo
Permitir análise de parâmetros de solo por localidade (município/ponto), com KPIs e comparações por recorte.

### Status
Em construção. Bases existem no BigQuery (staging `_temp`).

### Páginas previstas
- Resumo
- Mapa
- Parâmetros (distribuições, rankings)
- Histórico (se houver data de coleta)

---

## 3.2 Documentação Técnica (planejada)
### Tabelas staging
- `bdsolos_riodejaneiro_analisefisica_temp`
- `bdsolos_riodejaneiro_analisequimica_temp`
- `bdsolos_riodejaneiro_ataque_sulfurico_temp`
- `bdsolos_riodejaneiro_descricao..._temp`
- `bdsolos_riodejaneiro_observacoes_temp`

### Recomendação (pra ficar sustentável)
Criar view de consumo:
- `vw_solos_rj_consolidado`

Motivo: Looker gosta de fonte “limpa” e consistente. Sem isso vira gambiarra por gráfico.
