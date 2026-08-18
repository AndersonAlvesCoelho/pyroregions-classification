# Pirorregiões em Unidades de Conservação Federais

Classificação de regime de fogo em Unidades de Conservação federais brasileiras, inspirada no conceito de pirorregiões (*pyromas*) de Archibald et al., que caracteriza territórios pelo comportamento do fogo ao longo do tempo, não pela vegetação ou pelo clima.

## Sobre o projeto

Duas áreas podem ter vegetação parecida e regime de fogo bem diferente, ou o contrário. Em vez de assumir que fogo e bioma andam sempre juntos, este projeto mede o comportamento do fogo diretamente, usando cinco variáveis centrais, e deixa os padrões emergirem dos dados através de clusterização.

**As cinco variáveis do regime de fogo:**

- **Frequência** — de quanto em quanto tempo uma área volta a queimar
- **Intensidade** — quanta energia o fogo libera quando ocorre
- **Sazonalidade** — em que época do ano o fogo se concentra
- **Tamanho** — extensão do maior evento já registrado
- **Extensão média** — quanto território queima, em média, todo ano

Essas variáveis são calculadas célula a célula, numa grade espacial de 5×5 km cobrindo o território das UCs, e agrupadas por semelhança usando algoritmos de clusterização.

## Fontes de dados

| Fonte | Uso | Cobertura |
|---|---|---|
| **Limites de UCs** (WFS, INDE/ICMBio) | Base espacial da grade | 347 UCs federais |
| **AAF** — Área Afetada por Fogo (ICMBio) | Frequência, sazonalidade, tamanho, extensão média | 2010–2026 |
| **Foco de calor** (INPE) | Intensidade (FRP) | 2003–2025 |

## Pipeline

O projeto segue um fluxo adaptado do CRISP-DM, documentado em etapas dentro do notebook:

1. **Contexto e objetivo** — fundamentação teórica e escopo
2. **Importação dos dados** — download e harmonização das três fontes
3. **Preparação dos dados** — construção da grade, correção de inconsistências, persistência em Parquet
4. **Análise exploratória** — ranking entre UCs, séries temporais, comparação AAF × INPE
5. **Pré-processamento** — cálculo das cinco variáveis por célula, thresholds de robustez, normalização
6. **Modelagem** — três algoritmos de clustering testados (K-means, hierárquico, GMM)
7. **Avaliação do modelo** — comparação de métricas e cruzamento com bioma
8. **Ajuste de hiperparâmetros** — teste de sensibilidade dos thresholds de robustez
9. **Validação final** — resultado consolidado, nomeado e persistido

## Resultado

Das 22.687 células da grade, **5,7%** tiveram dado suficiente nas cinco variáveis para entrar na classificação (Grupo A). O restante recebeu rótulo de dado insuficiente (Grupo B), sem forçar classificação onde a evidência não sustentava.

K-means (K=3) foi o algoritmo escolhido, com convergência confirmada pelo clustering hierárquico. Três regimes de fogo emergiram, aproximados aos pyromas do artigo de referência:

| Regime | Perfil | Aproximação |
|---|---|---|
| Frequente e extenso | FRI ~1,4 anos, maior tamanho e extensão | FIL |
| Frequente e pequeno | FRI ~2,1 anos, menor intensidade e extensão | FCS |
| Raro e extenso | FRI ~4,0 anos, alta extensão quando ocorre | RIL |

Um diagnóstico de cobertura identificou o **AAF como principal fator limitante** da robustez do pipeline, o INPE sozinho cobre 26,7% da grade com confiabilidade, enquanto o AAF cobre apenas 5,7%.

## Limitações conhecidas

- Cobertura de dado confiável restrita a uma fração pequena do território
- Vínculo entre eventos AAF e focos de calor é aproximado, por proximidade espacial e temporal
- Classificação de tipo de evento (manejo vs. incêndio) é recente na série histórica, incompleta em anos anteriores
- Nomenclatura aproximada aos pyromas do artigo original é conceitual, não uma equivalência direta — escala geográfica e temporal são bem menores que o estudo de referência

## Próximos passos

Incorporação de fontes adicionais de área queimada por satélite (MapBiomas Fogo, MODIS MCD64A1, GABAM, FireCCI, LASA-Alarmes) como reforço à cobertura do AAF, atacando diretamente o gargalo identificado no diagnóstico final.

## Ambiente

```bash
conda env create -f environment.yml
conda activate geo
```

Principais bibliotecas: `geopandas`, `pandas`, `scikit-learn`, `scipy`, `matplotlib`, `seaborn`, `geemap`, `earthengine-api`.

## Referência

Archibald, S., Lehmann, C. E. R., Gómez-Dans, J. L., & Bradstock, R. A. (2013). Defining pyromes and global syndromes of fire regimes. *Proceedings of the National Academy of Sciences*, 110(16), 6442–6447.
