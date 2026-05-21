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
