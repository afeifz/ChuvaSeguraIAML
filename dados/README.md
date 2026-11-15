# Dados – ChuvaSegura (ML)

## Fonte

| Item | Descrição |
|---|---|
| **Instituição** | INMET – Instituto Nacional de Meteorologia |
| **Conjunto** | Dados Históricos: observações **horárias** das estações meteorológicas automáticas |
| **Link** | https://portal.inmet.gov.br/dadoshistoricos |
| **Estação usada** | **A701 – São Paulo (Mirante de Santana)** |
| **Período** | 2019 a 2025|
| **Data do download** | 14/11/2026|
| **Natureza** | **Dados reais e públicos.** Nenhum dado simulado foi usado neste projeto de ML. |

> **Limitação:** a estação A701 fica na zona norte de São Paulo, não no Ipiranga. Ela representa a chuva da **cidade**, e não exatamente a do bairro.

## Como obter os dados (passo a passo)

1. Acesse https://portal.inmet.gov.br/dadoshistoricos.
2. Baixe o arquivo **ZIP de cada ano** do período escolhido (sugestão: de 2019 a 2025, todos anos completos).
3. **Não precisa extrair.** Coloque os ZIPs na pasta `ml/dados/inmet/`.
4. Abra o notebook e execute tudo. Ele procura dentro dos ZIPs os arquivos cujo nome contém `A701` e ignora as demais estações.

Se o ZIP do ano corrente (2026) estiver no meio do ano, não tem problema: o notebook só usa como teste um **ano completo** (com os 12 meses).

## Variáveis usadas

O INMET informa dezenas de colunas; usamos estas (nomes padronizados no notebook):

| Nome no notebook | Coluna original (INMET) | Unidade |
|---|---|---|
| `chuva_mm` | Precipitação total, horário | mm |
| `pressao_hpa` | Pressão atmosférica ao nível da estação, horária | mB (= hPa) |
| `temp_c` | Temperatura do ar – bulbo seco, horária | °C |
| `orvalho_c` | Temperatura do ponto de orvalho | °C |
| `umidade_pct` | Umidade relativa do ar, horária | % |
| `vento_ms` | Vento, velocidade horária | m/s |
| `rajada_ms` | Vento, rajada máxima | m/s |
| `radiacao_kj` | Radiação global | kJ/m² |

## Tratamento aplicado

- O arquivo tem 8 linhas de metadados antes da tabela; o notebook localiza o cabeçalho sozinho.
- Separador `;`, decimal com vírgula e `-9999` (ausente) são convertidos para números e `NaN`.
- Data e hora aparecem em formatos diferentes entre os anos; o notebook aceita os dois.
- Os horários vêm em **UTC**. Para o ciclo do dia usamos o horário de Brasília (UTC−3). O notebook confere isso automaticamente: a radiação solar deve ter pico por volta de 15h UTC.
- Horas sem registro viram linhas vazias (`NaN`), para que as janelas de histórico fiquem corretas.
- Valores ausentes restantes são preenchidos com a **mediana do treino** (aprendida só no treino, dentro do `Pipeline`).

## Termos de uso

Os dados são públicos, mas consultem os termos de uso do INMET antes de redistribuir os arquivos brutos. Por isso os ZIPs **não são versionados** no repositório (veja `.gitignore`): quem quiser reproduzir segue o passo a passo acima.
