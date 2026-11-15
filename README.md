# ChuvaSegura – Machine Learning

**Pergunta de ML:** com o que a estação meteorológica mede agora e nas últimas horas, vai cair **chuva forte nas próximas 3 horas** em São Paulo?

Modelo: **Random Forest** (classificação binária), dados reais do **INMET** (estação A701), validação cruzada temporal e teste em um ano inédito.

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



## Como rodar

```bash
cd ml
python -m venv .venv
# Windows: .venv\Scripts\activate   |   Linux/Mac: source .venv/bin/activate
pip install -r requirements.txt

# 1) baixe os ZIPs anuais do INMET e coloque em dados/inmet/  (veja dados/README.md)
# 2) abra e execute o notebook
jupyter lab
```

Abra `notebook_chuva_forte.ipynb`, rode **Kernel → Restart Kernel and Run All Cells**

## Conteúdo

| Arquivo / pasta | O que é |
|---|---|
| `notebook_chuva_forte.ipynb` | Notebook completo: dados, preparação, treino, resultados e conclusão |
| `dados/README.md` | Fonte, link, período e variáveis dos dados |
| `requirements.txt` | Dependências |
| `RELATORIO_ML.md` | Relatório (2 a 4 páginas), do problema à conclusão |
| `resultados/` | Gráficos, tabelas e `resumo.json` gerados pelo notebook |
| `modelo/` | Modelo treinado (`.joblib`) |


