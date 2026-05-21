# <img src="https://raw.githubusercontent.com/marwin1991/profile-technology-icons/refs/heads/main/icons/python.png" width="40" height="40" valign="middle" alt="Python Logo"> DESCRIÇÃO DA PROGRAMAÇÃO DE LIMPEZA E TRATAMENTO DE DADOS ATRAVÉS DO PYTHON/PANDAS SOBRE A PLANILHA DE TRANSPORTES DE PRODUTOS PERIGOSOS/COMBUSTÍVEIS.

ETL (Extract, Transform, Load - Extração, Transformação e Carga), focado na limpeza e padronização de dados logísticos de transporte de combustíveis.

Aqui está a descrição detalhada, linha por linha, de cada ação executada pelo programa:

## PART 1: IMPORTAÇÃO E CONEXÃO COM O GOOGLE DRIVE

Python
import pandas as pd
Ação: Importa a biblioteca pandas (apelidada de pd), que é a principal ferramenta do Python para manipulação e análise de tabelas de dados (DataFrames).

Python
from google.colab import drive
drive.mount('/content/drive')
Ação: Conecta o seu notebook do Google Colab ao seu Google Drive pessoal, permitindo que o código leia e salve arquivos diretamente nas suas pastas do Drive.

Python
origem = '/content/drive/MyDrive/logteam/Sprint 2/'
arq = origem + 'Transporte_de_Produtos_Quimicos_Perigosos_ou_Combustíveis..csv'
Ação: Define variáveis de texto (strings) com o caminho da pasta (origem) e o caminho completo do arquivo CSV de entrada (arq) que será analisado.

---

## PART 2: CARGA INICIAL E PADRONIZAÇÃO DE TEXTO

Python
Dados_Padronizados = pd.read_csv(arq, sep=';')
Ação: Lê o arquivo CSV usando o caractere ponto e vírgula (;) como separador de colunas e joga esses dados para dentro de uma tabela chamada Dados_Padronizados.

Python
Dados_Padronizados = Dados_Padronizados.applymap(lambda s: s.upper() if type(s) == str else s)
Ação: Percorre cada célula de toda a tabela. Se o conteúdo for um texto (string), ele o transforma em letras MAIÚSCULAS. Se não for texto (números ou nulos), ele mantém como está. Isso evita problemas com buscas de texto case-sensitive (como diferenciar "Gasolina" de "GASOLINA").

Python
Dados_Padronizados.head(3)
Ação: Exibe na tela as 3 primeiras linhas da tabela para que você possa fazer uma inspection visual rápida se os dados foram carregados corretamente.

---

## PART 3: LIMPEZA DA COLUNA DE QUANTIDADE

Python
Dados_Padronizados['Quantidade Transportada'] = (
Dados_Padronizados['Quantidade Transportada']
.astype(str) # Convert to string first
.str.replace('.', '', regex=False)
.str.replace(',', '.', regex=False)
.astype(float)
)
Ação: Corrige a formatação numérica brasileira para o padrão americano/computacional. Ela:
- Transforma o número em texto.
- Remove o ponto separador de milhar (ex: 1.000 vira 1000).
- Substitui a vírgula decimal por ponto (ex: 10,50 vira 10.50).
- Converte tudo de volta para o tipo numérico decimal (float), permitindo que cálculos matemáticos sejam feitos.

---

## PART 4: CONVERSÃO DE UNIDADES DE MEDIDA PARA LITROS

Nesta etapa, o código identifica registros que não estão em litros e aplica fatores de conversão matemáticos baseados na regra de negócio do projeto:

Python
mask_metros = Dados_Padronizados['Unidade de Medida'].str.contains('METRO', na=False)
Dados_Padronizados.loc[mask_metros, 'Quantidade Transportada'] *= 1000
Dados_Padronizados.loc[mask_metros, 'Unidade de Medida'] = 'LITRO'
Ação: Cria uma "máscara" (filtro booleano) para achar linhas que contêm a palavra "METRO" (presumivelmente metros cúbicos). Multiplica o valor transportado por 1000 (1 m³ = 1000L) e altera o nome da unidade para 'LITRO'.

Python
mask_kg = Dados_Padronizados['Unidade de Medida'].str.contains('KILOGRAMAS', na=False)
Dados_Padronizados.loc[mask_kg, 'Quantidade Transportada'] *= 1.25
Dados_Padronizados.loc[mask_kg, 'Unidade de Medida'] = 'LITRO'
Ação: Filtra linhas com "KILOGRAMAS". Multiplica a quantidade por 1.25 (uma estimativa de densidade/conversão para combustível líquido) e muda a unidade para 'LITRO'.

Python
mask_ml = Dados_Padronizados['Unidade de Medida'].str.contains('MILILITRO', na=False)
Dados_Padronizados.loc[mask_ml, 'Quantidade Transportada'] *= 0.001
Dados_Padronizados.loc[mask_ml, 'Unidade de Medida'] = 'LITRO'
Ação: Filtra linhas com "MILILITRO". Multiplica por 0.001 (divide por 1000) para converter para 'LITRO'.

Python
mask_galao_acento = Dados_Padronizados['Unidade de Medida'].str.contains('GALÃO', na=False)
Dados_Padronizados.loc[mask_galao_acento, 'Quantidade Transportada'] *= 3.785
Dados_Padronizados.loc[mask_galao_acento, 'Unidade de Medida'] = 'LITRO'
Ação: Filtra linhas com "GALÃO" (com acento). Multiplica por 3.785 (tamanho padrão de um galão americano em litros) e muda a unidade para 'LITRO'.

Python
mask_galao = Dados_Padronizados['Unidade de Medida'].str.contains('GALAO', na=False)
Dados_Padronizados.loc[mask_galao, 'Quantidade Transportada'] *= 3.785
Dados_Padronizados.loc[mask_galao, 'Unidade de Medida'] = 'LITRO'
Ação: Faz exatamente a mesma coisa que a linha anterior, mas captura os casos onde a palavra "GALAO" foi escrita sem acento.

Python
mask_ton = Dados_Padronizados['Unidade de Medida'].str.contains('TONELADA', na=False)
Dados_Padronizados.loc[mask_ton, 'Quantidade Transportada'] *= 1200.0
Dados_Padronizados.loc[mask_ton, 'Unidade de Medida'] = 'LITRO'
Ação: Filtra linhas com "TONELADA". Multiplica por 1200
---

## PART 5: PRIMEIRO FILTRO TEMPORAL

Python
Dados_Padronizados['Ano'] = pd.to_numeric(Dados_Padronizados['Ano'], errors='coerce')
Ação: Garante que a coluna 'Ano' seja estritamente numérica. O argumento errors='coerce' diz ao Pandas que, se houver algum texto inválido ou campo corrompido no ano, ele deve transformá-lo em um valor nulo (NaN) em vez de travar o código.

Python
Dados_Padronizados = Dados_Padronizados[Dados_Padronizados['Ano'] >= 2013]
Ação: Filtra e remove da tabela qualquer registro cujo ano seja menor que 2013, mantendo apenas dados históricos recentes.

Python
Dados_Padronizados = Dados_Padronizados.sort_values(by='Ano', ascending=True)
Ação: Organiza as linhas da tabela de forma cronológica, do ano mais antigo (2013) para o mais recente.

Python
Dados_Padronizados.head(3)
Ação: Exibe as 3 primeiras linhas pós-filtro temporal.

---

## PART 6: FILTRAGEM AVANÇADA DE PRODUTOS E MAPEAMENTO DE UFS

Python
df_filtrado = Dados_Padronizados[Dados_Padronizados['Produto'].str.contains('GASOLINA|ETANOL|DIESEL|GLP|COMBUSTIVEIS', na=False)].copy()
Ação: Cria uma nova tabela chamada df_filtrado, contendo apenas linhas onde o produto seja um combustível (Gasolina, Etanol, Diesel, GLP ou contenha a palavra "Combustíveis"). O .copy() garante que esta tabela seja independente da tabela original na memória do computador.

Python
termos_para_excluir = 'AUTOMÓVEIS|AUTOMOVEIS|CAMINHÕES|CAMINHOES|MOTORES|ETANOLAMINAS|PEÇAS|PECAS|VEÍCULOS|TURBOALIMENTADORES'
df_filtrado = df_filtrado[~df_filtrado['Produto'].str.contains(termos_para_excluir, na=False)]
Ação: O sinal de til (~) funciona como uma exclusão (NÃO). Essa linha remove registros indesejados que passaram no filtro anterior por conterem nomes similares, mas que referem-se a autopeças, veículos ou compostos químicos diferentes (ex: remove "Motores a Diesel" ou "Etanolaminas").

Python
df_filtrado = df_filtrado[df_filtrado['Unidade de Medida'].str.contains('LITRO', na=False)]
df_filtrado = df_filtrado[df_filtrado['Ano'] >= 2013]
df_filtrado = df_filtrado.sort_values(by='Ano', ascending=True).reset_index(drop=True)
Ação: Reafirma os filtros garantindo que apenas registros em 'LITRO' e acima de 2013 permanecem, reordena por ano e limpa os índices das linhas (reinicia a contagem das linhas do 0, 1, 2...).

Python
mapeamento_ufs = { ... }
Ação: Cria um dicionário de tradução para converter nomes de estados por extenso (ex: 'SÃO PAULO') para suas respectivas siglas de duas letras (ex: 'SP').

Python
colunas_uf = ['Estado', 'UF - origem', 'UF - destino']
for col in colunas_uf:
df_filtrado[col] = df_filtrado[col].astype(str).str.strip().str.upper()
df_filtrado[col] = df_filtrado[col].map(mapeamento_ufs).fillna(df_filtrado[col])
Ação: Passa por cada uma das três colunas de localização territoriais, remove espaços em branco extras, garante que estão em maiúsculas, e substitui os nomes extensos pelas siglas criadas no dicionário. Se o nome não estiver no dicionário, o .fillna() mantém o valor original.

Python
pd.options.display.float_format = '{:.2f}'.format
Ação: Configura o Pandas para mostrar números decimais na tela com apenas duas casas após a vírgula (formato padrão de leitura financeira/humana).

---

## PART 7: CRUZAMENTO DE DADOS COM REGIÕES METROPOLITANAS (MERGE)

Python
caminho_regioes = origem + 'Regioes_adm_MAIUSCULO.csv'
df_regioes = pd.read_csv(caminho_regioes, sep='\t')
Ação: Carrega uma segunda base de dados auxiliar (df_regioes), separada por tabulação (\t), que correlaciona municípios com suas respectivas Regiões Metropolitanas.

Python
df_regioes = df_regioes[['Município', 'Região Metropolitana']].copy()
df_regioes['Município'] = df_regioes['Município'].astype(str).str.upper().str.strip()
df_regioes['Região Metropolitana'] = df_regioes['Região Metropolitana'].astype(str).str.strip()
df_regioes = df_regioes.drop_duplicates(subset=['Município'])
Ação: Limpa essa base auxiliar selecionando apenas as duas colunas necessárias, padronizando o texto do município para maiúsculo e removendo cidades duplicadas para evitar erros de multiplicação de linhas no cruzamento.

Python
df_filtrado.info()
df_filtrado.head(3)
Ação: Exibe metadados (como tipos de dados e memória gasta) e as 3 primeiras linhas da tabela principal antes do cruzamento.

Python
df_filtrado = pd.merge(
df_filtrado,
df_regioes,
left_on='Município - origem',
right_on='Município',
how='left'
)
Ação: Faz um cruzamento de tabelas (PROCV / Left Join). Ele busca o nome da cidade em Município - origem dentro da coluna Município da tabela de regiões e traz a informação da Região Metropolitana daquela cidade de origem.

Python
df_filtrado = df_filtrado.rename(columns={'Região Metropolitana': 'RM_Origem'})
df_filtrado = df_filtrado.drop(columns=['Município', 'Município_y', 'Município_x'], errors='ignore')
Ação: Renomeia a coluna que acabou de chegar para 'RM_Origem' e apaga colunas redundantes geradas pelo processo de cruzamento.

Python
df_filtrado = pd.merge(
df_filtrado,
df_regioes,
left_on='Município - destino',
right_on='Município',
how='left'
)
df_filtrado = df_filtrado.rename(columns={'Região Metropolitana': 'RM_Destino'})
df_filtrado = df_filtrado.drop(columns=['Município', 'Município_y', 'Município_x'], errors='ignore')
Ação: Repete exatamente o mesmo processo de cruzamento anterior, mas agora focado no município de destino da carga, renomeando o resultado para 'RM_Destino'.

---

## PART 8: TRATAMENTO DE NULOS E FORMATAÇÃO DE SIGLAS

Python
df_filtrado['RM_Origem'] = df_filtrado['RM_Origem'].astype(str).str.replace('REGIÃO METROPOLITANA', 'R.M.', regex=False)
df_filtrado['RM_Destino'] = df_filtrado['RM_Destino'].astype(str).str.replace('REGIÃO METROPOLITANA', 'R.M.', regex=False)
Ação: Abrevia o texto "REGIÃO METROPOLITANA" para "R.M." nas duas novas colunas para deixar os dados visualmente mais enxutos.

Python
df_filtrado['RM_Origem'] = df_filtrado['RM_Origem'].fillna('R.M. NÃO INFORMADA')
df_filtrado['RM_Destino'] = df_filtrado['RM_Destino'].fillna('R.M. NÃO INFORMADA')
df_filtrado['RM_Origem'] = df_filtrado['RM_Origem'].astype(str).replace({'nan': 'R.M. NÃO INFORMADA', '': 'R.M. NÃO INFORMADA'})
df_filtrado['RM_Destino'] = df_filtrado['RM_Destino'].astype(str).replace({'nan': 'R.M. NÃO INFORMADA', '': 'R.M. NÃO INFORMADA'})
Ação: Essas quatro linhas tratam dados ausentes. Cidades que não fazem parte de nenhuma Região Metropolitana oficialmente receberiam o valor de nulo (NaN ou nan). O código substitui esses vazios e textos nulos pela frase padrão 'R.M. NÃO INFORMADA'.

Python
df_filtrado.head(3)
Ação: Nova checagem visual das 3 primeiras linhas após a inserção das Regiões Metropolitanas.

---

## PART 9: CÁLCULO FINAL E EXPORTAÇÃO DO ARQUIVO

Python
df_filtrado['Quantidade m³'] = df_filtrado['Quantidade Transportada'] / 1000
Ação: Cria uma nova coluna chamada Quantidade m³ (metros cúbicos), dividindo o valor total em litros por 1000. Isso facilita análises macro de volumetria de carga líquida.

Python
df_filtrado.head(10)
Ação: Inspeção final exibindo as 10 primeiras linhas do resultado consolidado.

Python
df_filtrado.to_csv(origem + 'Planilha_Limpa_Filtrada_V3.2.csv', index=False)
Ação: Salva a tabela final totalmente limpa, padronizada e enriquecida em um novo arquivo CSV chamado Planilha_Limpa_Filtrada_V3.2.csv na pasta do seu Drive. O parâmetro index=False pede que o Pandas adicione uma coluna extra de numeração de linhas no arquivo gerado.

Python
print("Arquivo V3 gerado com sucesso com as siglas R.M.!")
Ação: Exibe uma mensagem de confirmação no console avisando que todo o processo terminou sem erros.
