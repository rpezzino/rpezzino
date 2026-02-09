# Documentação dos Conjuntos de Dados (BigQuery)

## Arquitetura (dados → Looker Studio)
**BigQuery (infoagro.infoagro.*) → Looker Studio (Fontes de Dados) → Painéis**

- Projeto: `infoagro`
- Dataset: `infoagro`
- Regra do jogo: cada painel consome tabelas do mesmo “repositório” (mesmo projeto/dataset).

---

# 1) Conjunto de Dados: Histórico_Clima_INMET

## 1.1 Identificação
- **Tabela (BigQuery)**: `infoagro.infoagro.historico_clima_inmet`
- **Fonte (Looker Studio)**: `historico_clima_inmet` (conectado na tabela)

## 1.2 Descrição
Base histórica de clima do INMET por **estação**, com registros em granularidade **horária**, incluindo localização (lat/long) e variáveis meteorológicas (precipitação, temperatura, pressão, umidade e vento).

## 1.3 Granularidade (grão do registro)
- **Temporal**: `data` + `hora` (horária)
- **Espacial**: estação (com latitude/longitude); recortes por UF e região
- **Unidade de registro** (conceitual): 1 linha ≈ 1 leitura em 1 estação em 1 hora

## 1.4 Campos (dicionário de dados)
> Tipos conforme Looker Studio / BigQuery (visão prática).  
> “Agregação recomendada” é a forma correta de agregar em gráficos/KPIs.

### Identificação e geografia
| Campo | Tipo | Agregação recomendada | Descrição |
|---|---|---:|---|
| `estacao` | Texto | Nenhum | Nome/identificador da estação INMET |
| `codigo_wmo` | Texto | Nenhum | Código WMO da estação |
| `uf` | Texto | Nenhum | Unidade da federação da estação |
| `regiao` | Texto | Nenhum | Região (ex.: Sudeste) |
| `latitude` | Número | Nenhum | Latitude da estação |
| `longitude` | Número | Nenhum | Longitude da estação |
| `latlong` | Latitude/longitude | Nenhum | Campo geográfico (ponto) para mapas |
| `altitude` | Número | Nenhum | Altitude da estação |

### Tempo
| Campo | Tipo | Agregação recomendada | Descrição |
|---|---|---:|---|
| `data` | Data | Nenhum | Data da leitura |
| `hora` | Texto | Nenhum | Hora da leitura |
| `ano` | Ano (AAAA) | Nenhum | Campo derivado para filtro/agrupamento |
| `mês` | Mês (MM) | Nenhum | Campo derivado para filtro/agrupamento |

### Variáveis meteorológicas (principais)
| Campo | Tipo | Agregação recomendada | Descrição |
|---|---|---:|---|
| `precipitacao_total_horari...` | Número | **Soma** | Precipitação acumulada no período (agregar por soma) |
| `temperatura_media` | Número | **Média** | Temperatura média |
| `temperatura_max_hora...` | Número | **Máx** | Temperatura máxima no recorte |
| `temperatura_min_hora...` | Número | **Mín** | Temperatura mínima no recorte |
| `pressao_atm_media` | Número | **Média** | Pressão atmosférica média |
| `pressao_atm_max_hora...` | Número | **Máx** | Pressão máxima |
| `pressao_atm_min_hota...` | Número | **Mín** | Pressão mínima |
| `umidade_rel_media` | Número | **Média** | Umidade relativa média |
| `umidade_rel_max_hora...` | Número | **Máx** | Umidade máxima |
| `umidade_rel_min_hora...` | Número | **Mín** | Umidade mínima |
| `vento_velocidade_horari...` | Número | **Média** | Velocidade do vento média |
| `vento_rajada_maxima_ms` | Número | **Máx** | Rajada máxima do vento |
| `vento_dir_horaria_gr` | Número | Nenhum | Direção do vento (graus). (Evitar média simples; usar apenas para leitura/tooltip) |
| `radiacao_global_kj_m2` | Número | **Média** | Radiação global (agregar conforme uso) |

> Observação: existem campos adicionais de “orvalho”, “bulbo”, etc. Se estiverem nos gráficos, documente também neste mesmo padrão.

## 1.5 Atualização
- Atualização automática: **12 horas** (conforme conexão exibida no Looker Studio)

## 1.6 Limitações (qualidade e cobertura)
- Pode existir **ausência de dados por estação/período**
- Comparações entre estações exigem atenção à **cobertura temporal**
- Agregações: precipitação soma; temperatura/pressão/umidade/vento médias/min/max conforme regra acima

---

# 2) Conjunto de Dados: Produtividade_Rural_Municípios_RJ

## 2.1 Identificação
- **Tabela (BigQuery)**: `infoagro.infoagro.produtividade_rural_municipios_rj_2021_20...`
- **Fonte (Looker Studio)**: `produtividade_rural_municipios_rj_20...`

## 2.2 Descrição
Base anual de indicadores de produção e produtividade agrícola no RJ, por **região, município e cultura**, com métricas de área colhida, produção, produtores, preço e faturamento.

## 2.3 Granularidade (grão do registro)
- **Temporal**: anual (`ano`)
- **Espacial**: município + região
- **Dimensão**: cultura
- **Unidade de registro**: 1 linha ≈ 1 cultura em 1 município em 1 ano

## 2.4 Campos (dicionário de dados)
| Campo | Tipo | Agregação recomendada | Descrição |
|---|---|---:|---|
| `ano` | Número | Nenhum | Ano de referência |
| `regiao` | Texto | Nenhum | Região (ex.: Serrana) |
| `municipio` | Texto | Nenhum | Município |
| `estado` | Texto | Nenhum | UF (RJ) |
| `cultura` | Texto | Nenhum | Cultura (ex.: café, tomate etc.) |
| `area_colhida_ha` | Número | **Soma** | Área colhida em hectares |
| `producao_colhida_t` | Número | **Soma** | Produção colhida em toneladas |
| `produtores` | Número | **Soma*** | Nº produtores (atenção à duplicidade por cultura) |
| `faturamento_bruto` | Número | **Soma** | Faturamento bruto |
| `preco_kg` | Número | **NÃO somar** | Preço por kg (usar cálculo ponderado) |
| `produtividade_t_ha` | Número | **NÃO somar** | Produtividade (usar cálculo ponderado) |

\* `produtores`: só faz sentido como soma se o dado for “produtor único por município/ano”. Se estiver “por cultura”, somar pode inflar. Tratar como métrica “sensível ao grão”.

## 2.5 Atualização
- Atualização automática: **12 horas**

## 2.6 Regras oficiais (para não sair KPI mentiroso)
- **Produtividade calculada (t/ha)** = `SUM(producao_colhida_t) / SUM(area_colhida_ha)`
- **Preço médio (R$/kg)** = `SUM(faturamento_bruto) / (SUM(producao_colhida_t) * 1000)`

## 2.7 Limitações
- `preco_kg` e `produtividade_t_ha` não devem ser agregados por soma
- risco de duplicidade em `produtores` dependendo do grão

---

# 3) Conjunto de Dados: RJ_Municípios (Geoespacial)

## 3.1 Identificação
- **Tabela (BigQuery)**: `infoagro.infoagro.rj_municipios`
- **Fonte (Looker Studio)**: `rj_municipios`

## 3.2 Descrição
Tabela geoespacial com polígonos dos municípios do RJ (`geometry`) e chaves para relacionamento com fatos (produção/clima/solos).

## 3.3 Granularidade
- **Espacial**: município (1 linha por município)

## 3.4 Campos (dicionário de dados)
| Campo | Tipo | Agregação recomendada | Descrição |
|---|---|---:|---|
| `codigo_ibge` | Número | Nenhum | Chave recomendada para join |
| `municipio` | Texto | Nenhum | Nome do município |
| `municipio_uf` | Texto | Nenhum | Nome + UF padronizado |
| `geometry` | Geoespacial | Nenhum | Polígono do município (mapa) |
| `ano` | Número | Nenhum | Usado quando a fonte incorpora métricas por ano |
| `area_colhida_ha` | Número | Soma | (se presente na fonte) métrica agregável |
| `producao_colhida_t` | Número | Soma | (se presente na fonte) |
| `faturamento_bruto` | Número | Soma | (se presente na fonte) |
| `preco_kg` | Número | NÃO somar | (se presente na fonte) |
| `produtividade_t_ha` | Número | NÃO somar | (se presente na fonte) |

## 3.5 Atualização
- conforme conexão (12 horas)

## 3.6 Limitações
- Join por nome de município pode falhar (acentos/abreviações). Preferir `codigo_ibge`.

---

# 4) Conjunto de Dados: Solos_RJ (staging, em construção)

## 4.1 Identificação
Todas as tabelas abaixo estão em:
- `infoagro.infoagro.<tabela>`

Tabelas staging:
- `bdsolos_riodejaneiro_analisefisica_temp`
- `bdsolos_riodejaneiro_analisequimica_temp`
- `bdsolos_riodejaneiro_ataque_sulfurico_temp`
- `bdsolos_riodejaneiro_descricao..._temp`
- `bdsolos_riodejaneiro_observacoes_temp`

## 4.2 Descrição
Bases preliminares para construção do painel de Análise de Solos. Por enquanto estão em formato staging (`_temp`), exigindo consolidação/padronização antes de virar fonte final do painel.

## 4.3 Granularidade (a confirmar)
- Esperado: registro por amostra/ponto/coleta (depende do schema)
- Esperado: vínculo geográfico por município/código IBGE e/ou coordenada

## 4.4 Recomendação técnica (para o painel nascer direito)
Criar view “camada de consumo”:
- `vw_solos_rj_consolidado` (sugestão)

Objetivo da view:
- padronizar chaves geográficas (municipio/codigo_ibge)
- padronizar unidades
- expor parâmetros em formato pronto para KPI/gráfico (colunas ou estrutura parametro/valor)

## 4.5 Atualização
- seguirá o padrão do projeto (12 horas), após criação da fonte Looker

## 4.6 Limitações
- Sem painel ainda, campos e regras finais dependem do modelo consolidado
