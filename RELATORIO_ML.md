# Relatório de Machine Learning – ChuvaSegura

**Global Solution FIAP 2026.2 · CitizenTech · Inteligência Artificial & Machine Learning**  

## Ficha da CitizenTech

| Campo | Conteúdo |
|---|---|
| **Nome da CitizenTech** | ChuvaSegura |
| **Cidade e bairro** | São Paulo – Ipiranga (região do Rio Tamanduateí) |
| **Cidadão beneficiado** | Moradores e comerciantes do Ipiranga em ruas com histórico de alagamento, com atenção a idosos e pessoas com mobilidade reduzida |
| **Problema, em uma frase** | Moradores de áreas que alagam recebem o aviso de chuva forte tarde demais, ou de forma genérica demais, para proteger a família e seus bens. |
| **Como funciona** | Um sensor mede o nível da água num ponto crítico; um modelo de ML estima a chance de chuva forte; a API combina os dois e envia alertas ao app do morador, que também registra ocorrências. |
| **O que cada parte faz** | IoT (ESP32/Wokwi): mede e sinaliza o nível · API C#: recebe leituras e calcula risco · API Java: cadastro e alertas · **ML: previsão de chuva forte com dados do INMET** · App Expo: risco do bairro, alertas e ocorrências · Linux: hospeda e monitora o sistema · Cybersecurity: protege os dados dos moradores · QA: features e casos de teste |
| **Link do repositório** | https://github.com/afeifz/ChuvaSeguraIAML.git |
| **Indicador de impacto** | Minutos de antecedência entre o alerta e o nível crítico; % de chuvas fortes detectadas pelo modelo (recall) |
| **ODS conectados** | 11 (Cidades sustentáveis), 13 (Ação climática), 9 (Infraestrutura e inovação), 10 (Redução das desigualdades) |

**Equipe:** 

**Mohamed Afif RM 554445**

**Lucca Cardinale RM556668**


**Vídeo:** https://youtu.be/pBuUeKequg0


## 1. Problema e cidadão

O **ChuvaSegura** ajuda moradores do **Ipiranga (São Paulo)** em ruas com histórico de alagamento. O sensor de nível só detecta a água quando ela já está subindo; um aviso antecipado dá tempo para guardar objetos, combinar ajuda com vizinhos e evitar sair de casa, principalmente para **idosos e pessoas com mobilidade reduzida**.

**Pergunta de ML (classificação binária):** com as medições da estação meteorológica na hora atual e nas últimas horas, vai cair chuva forte (≥ **10 mm acumulados**) nas próximas 3 horas?

## 2. Dados reais e públicos

- **Fonte:** INMET, Dados Históricos (https://portal.inmet.gov.br/dadoshistoricos), estação **A701 – São Paulo (Mirante de Santana)**.
- **Período dos arquivos:** **01/01/2019 a 31/12/2025**.
- **Registros horários brutos:** **61.368 horas**. Após preparação e remoção das linhas sem alvo/histórico de 24 h: **58.327 linhas utilizáveis**.
- **Arquivos anuais lidos:** 2019, 2020, 2021, 2022, 2023, 2024 e 2025. **Data efetiva do download:** 14/11
- **Variáveis:** chuva, pressão, temperatura, ponto de orvalho, umidade, vento, rajada e radiação (horárias).
- **Limitação:** a estação fica na zona norte, não no Ipiranga; os dados são horários.
- **Limiar de “chuva forte”:** **10,0 mm acumulados nas próximas 3 horas**, conforme a configuração `LIMIAR_MM = 10.0` do notebook. No conjunto utilizável, isso gerou **952 horas positivas (1,63%)**, portanto um evento raro. 
## 3. Escolha do modelo

**Random Forest** (classificação). Justificativa: dados tabulares com relações não lineares e interações entre variáveis; funciona bem com poucos eventos raros (foi usado `class_weight="balanced_subsample"`); não precisa normalizar; e é explicável por importância das variáveis, o que ajuda a explicar o aviso ao cidadão. Redes neurais exigiriam mais dados e seriam mais difíceis de explicar; K-Means não é supervisionado e não responde à pergunta.

## 4. Preparação e divisão treino/teste

- **Variáveis:** 20 features: medições da hora (`chuva_mm`, pressão, temperatura, ponto de orvalho, umidade, vento, rajada e radiação); chuva acumulada em 3, 6, 12 e 24 h; variação de pressão, umidade e temperatura em 3 h; depressão do ponto de orvalho; hora do dia e mês (seno/cosseno).
- **Alvo:** chuva somada nas próximas 3 horas **≥ 10 mm**. As variáveis usam só informação até a hora atual (sem vazamento de dados).
- **Limpeza:** `-9999` tratado como ausente; horas faltantes explicitadas; radiação noturna ausente = 0; demais ausentes preenchidos com a mediana **do treino** (dentro do `Pipeline`).
- **Divisão por tempo:** treino = **2019–2024** (**50.430** linhas, **839** eventos positivos; 1,66%); **teste = 2025** (**7.897** linhas, **113** eventos positivos; 1,43%). A divisão não é aleatória porque horas vizinhas são quase iguais e um sorteio daria uma nota irreal; assim simulamos o uso real (prever um ano que o modelo nunca viu).

## 5. Treino com diferentes hiperparâmetros

Busca aleatória (`RandomizedSearchCV`) com **12 combinações** de 3 hiperparâmetros e **validação cruzada temporal** (`TimeSeriesSplit`, **5 dobras**, folga de 24 h) **somente no treino**. Métrica de seleção: **PR-AUC** (adequada a classes raras).

| Hiperparâmetro | Valores testados | O que controla |
|---|---|---|
| `n_estimators` | 100, 200, 300 | número de árvores |
| `max_depth` | 4, 6, 8, 12, 16 | profundidade (controla overfitting) |
| `min_samples_leaf` | 1, 5, 10, 20, 50 | tamanho mínimo das folhas (controla overfitting) |

**Melhor combinação:** `n_estimators=100`, `max_depth=16`, `min_samples_leaf=20` · **PR-AUC média de validação = 0,154**.

| n_estimators | max_depth | min_samples_leaf | PR-AUC treino das dobras | PR-AUC validação | gap de overfitting |
|---:|---:|---:|---:|---:|---:|
| 100 | 16 | 20 | 0,635 | **0,154** | 0,481 |
| 100 | 8 | 50 | 0,385 | **0,154** | 0,231 |
| 200 | 6 | 50 | 0,335 | 0,150 | 0,185 |
| 100 | 4 | 50 | 0,274 | 0,134 | 0,140 |
| 200 | 4 | 20 | 0,290 | 0,133 | 0,157 |
| 100 | 4 | 1 | 0,290 | 0,126 | 0,164 |
| 300 | 8 | 5 | 0,559 | 0,125 | 0,434 |
| 200 | 4 | 5 | 0,291 | 0,125 | 0,166 |
| 200 | 8 | 5 | 0,556 | 0,125 | 0,431 |
| 200 | 16 | 5 | 0,949 | 0,123 | 0,826 |
| 100 | 16 | 5 | 0,940 | 0,120 | 0,820 |
| 200 | 16 | 1 | 0,920 | 0,115 | 0,805 |

**Limiar de decisão:** escolhido pelo **F2 máximo** nas previsões fora da amostra da validação cruzada do treino: **0,423**, porque não avisar uma chuva forte é pior do que um alarme falso.

## 6. Resultados: treino × teste

| Conjunto | Precisão | Recall | F1 | F2 | PR-AUC | ROC-AUC |
|---|---:|---:|---:|---:|---:|---:|
| Treino (dentro da amostra) | 0,225 | 0,999 | 0,368 | 0,592 | 0,615 | 0,991 |
| **Teste (2025)** | **0,120** | **0,531** | **0,196** | **0,316** | **0,156** | **0,901** |
| Referência: regra “choveu ≥ 1 mm em 3 h” (teste) | 0,101 | 0,425 | 0,163 | 0,258 | 0,084 | 0,731 |
| Referência: sorteio pela proporção (teste) | 0,000 | 0,000 | 0,000 | 0,000 | 0,014 | 0,500 |

**PR-AUC na validação cruzada:** **0,154**.

**Gráficos:** as imagens da execução foram incluídas na pasta `resultados/`: `01_exploracao.png` (exploração dos dados), `02_busca_hiperparametros.png` (comparação das 12 combinações), `03_resultados_teste.png` (curva precisão-recall, matriz de confusão e probabilidades previstas) e `04_importancia_variaveis.png` (importância por permutação). Os arquivos `metricas.csv`, `busca_hiperparametros.csv` e `resumo.json` também foram preservados como evidências numéricas.

**Overfitting:** há diferença expressiva entre treino e dados não vistos. Para a melhor combinação, a PR-AUC média nas dobras de treino foi **0,635**, contra **0,154** na validação (gap de **0,481**). A PR-AUC dentro da amostra em todo o treino foi **0,615**, mas essa métrica é naturalmente otimista e não mede a generalização. A combinação `100 árvores, profundidade 8 e folha mínima 50` teve validação também de **0,154**, mas gap menor (**0,231**); já a combinação selecionada pelo processo apresentou gap maior. A PR-AUC da validação (**0,154**) previu muito bem a do teste de 2025 (**0,156**, diferença de 0,002), indicando que a avaliação temporal estimou adequadamente o desempenho no ano inédito. Entre os hiperparâmetros, `max_depth` e `min_samples_leaf` são os controles usados para reduzir o overfitting; a tabela mostra que folhas maiores/profundidades menores reduziram o gap em algumas combinações.

**Variáveis mais importantes:** conforme o gráfico de importância por permutação (queda da PR-AUC no teste), as cinco primeiras são: `orvalho_c` (maior contribuição), `chuva_mm`, `temp_c`, `delta_temp_3h` e `delta_pressao_3h`. Em seguida aparecem `chuva_acum_3h`, `pressao_hpa`, `hora_sin`, `mes_sin`, `radiacao_kj`, `chuva_acum_12h` e `chuva_acum_6h`. O gráfico foi incluído em `resultados/04_importancia_variaveis.png`. A importância representa quanto a PR-AUC cai quando cada variável é embaralhada no conjunto de teste; não significa causalidade.

## 7. Conclusão e leitura para o cidadão

No ano de teste (**2025**), foram identificados **34 episódios reais de chuva forte** (episódios agrupados com intervalo de até 6 horas):
- De cada **34** episódios, o modelo avisou **23 (68%)**. Os outros **11** passaram sem aviso.
- Foram emitidos **118 episódios de alerta**, dos quais **27** foram confirmados por chuva forte e **91** foram alarmes falsos (cerca de **7,6 por mês**).

Na avaliação horária, o modelo identificou **60** das **113** horas positivas (recall de **53,1%**), deixou passar **53** horas positivas e gerou **438** falsos positivos. As métricas por episódio e por hora usam unidades de análise diferentes e não devem ser confundidas.

Para o morador, isso significa um **aviso de até 3 horas** para se preparar. Não é uma previsão exata do bairro.

**Limitações:** estação fora do Ipiranga; dados horários; poucos eventos extremos no ano de teste; o modelo prevê **chuva forte**, não **alagamento** (que depende também de drenagem, rio e solo). A precisão no teste foi **12,0%**, então a quantidade de alarmes falsos ainda é relevante.

**Próximos passos:** usar a probabilidade do modelo como reforço do risco na API de monitoramento (C#); testar outras estações e mais anos; comparar com XGBoost.

## 8. Ligação com o ChuvaSegura e com os ODS

O modelo antecipa a chuva; o sensor (ESP32) confirma a água subindo; a API em C# classifica o risco; a API em Java avisa os moradores. **ODS 11** (Cidades e comunidades sustentáveis), **ODS 13** (Ação contra a mudança global do clima), **ODS 9** (Inovação e infraestrutura) e **ODS 10** (Redução das desigualdades, ao priorizar quem tem mobilidade reduzida).
