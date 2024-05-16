<h1 align="center"> Projeto de Processamento de Dados - UNIFOR </h1>

## Índice

* [1. Descrição do Projeto](#1-descrição-do-projeto)
* [2. Sobre a API](#2-sobre-a-api)
* [3. Tabela Estatística](#3-tabela-estatística)
* [4. Bibliotecas Utilizadas](#4-bibliotecas-utilizadas)
* [5. Acessando os dados e resultados](#5-acessando-os-dados-e-resultados)
* [5.1 Acesso as dados](#51-acesso-aos-dados)
* [5.2 Abertura do arquivo, tratando colunas e pivotando os dados](#52-abertura-do-arquivo-tratando-colunas-e-pivotando-os-dados)
* [5.3 Analisando o dataframe pivotado](#53-analisando-o-dataframe-pivotado)
* [6. Gerando Gráficos - Variação Mensal](#6-gerando-gráficos-para-análises---variação-mensal)
* [7. Gerando Gráficos - Variação Anual Acumulada](#7-gerando-gráficos-para-análises---variação-anual-acumulada)
* [8. Gerando Gráficos - Variação Anual Acumulada por Estado](#8-gerando-gráficos-para-análises---variação-anual-acumulada-do-estado)

## 1. Descrição do Projeto
Extrair e analisar um conjunto de dados fornecido por uma API pública.
## 2. Sobre a API
A API é fornecida pelo Instituto Brasileiro de Geografia e Estatística (IBGE) por meio do Banco de Tabelas Estatísticas através do Sistema IBGE de Recuperação Automática (SIDRA).

Informações sobre o sidra: https://sidra.ibge.gov.br/ajuda
## 3. Tabela Estatística
A escolha da tabela foi a Pesquisa Industrial Mensal - Produção Física (PIMPF), ela produz indicadores de curto prazo relativos ao comportamento do produto real da indústria, tendo como unidade de investigação a empresa formalmente constituída cuja principal fonte de receita seja a atividade industrial.

Link da tabela: https://sidra.ibge.gov.br/tabela/8888

Link da API (JSON) = https://apisidra.ibge.gov.br/values/t/8888/n3/all/v/all/p/last%201/c544/all/d/v11601%201,v11602%201,v11603%201,v11604%201,v12606%205,v12607%202

## 4. Bibliotecas Utilizadas

* python = "^3.12.1"
* requests = "^2.31.0"
* pandas = "^2.2.2"
* openpyxl = "^3.1.2"
* matplotlib = "^3.9.0"
* numpy = "^1.26.4" 

## 5. Acessando os dados e resultados
Amostragem de dados referente ao último mês disponível (Março-2024) atualizado em 09/05/2024.

* ### 5.1 Acesso aos dados:
```Python
# Realizando requisição GET para verificar o status da API e baixar os arquivos em JSON e Excel com colunas selecionadas.
response = requests.get(url, headers={'application': 'json'}, verify=False)

try:
    if response.status_code == 200:
        data = response.json()

        # Chaves selecionadas
        selected_keys = ['D1N', 'V', 'D2N', 'D3N', 'D4N']

        # Filtrar os dados
        filtered_data = [
            {key: entry[key] for key in selected_keys if key in entry}
            for entry in data
        ]

        # Salvar como JSON
        with open(path_json, 'w', encoding='utf-8') as json_file:
            json.dump(filtered_data, json_file, ensure_ascii=False)
        print(f'Arquivo JSON salvo em {path_json}')

        # Converter os dados filtrados em um DataFrame do pandas
        df = pd.DataFrame(filtered_data)

        # Salvar o DataFrame em um arquivo Excel
        df.to_excel(path_excel, index=False)
        print(f'Arquivo Excel salvo em {path_excel}')

except Exception as err:
    print(f'Falha ao acessar o site! Erro: {err}.')
```

* ### 5.2 Abertura do arquivo, tratando colunas e pivotando os dados
 
![alt text](./Imagens/image-3.png)
```Python
# Realizando abertura do arquivo excel, pivotando as variaveis para colunas, alterando o tipo de dado e salvando em XLSX
path_excel = fr'C:\Users\user\Desktop\projeto_processamento\tests\data\dados.xlsx'

tipos_dados = {
    'Unidade da Federação': str,
    'Valor': float,
    'Variável': str,
    'Mês': str,
    'Seções e atividades industriais (CNAE 2.0)': str
}

choose_coluns = list(tipos_dados.keys())

# Adicionando skiprows para ignorar a primeira linha
df = pd.read_excel(path_excel, usecols=choose_coluns,
                   dtype=tipos_dados, na_values='-', skiprows=1)

df['Variável'] = df['Variável'].str.replace('PIMPF - ', '')

# Pivotar os dados
pivot_df = df.pivot_table(index=['Unidade da Federação', 'Mês', 'Seções e atividades industriais (CNAE 2.0)'],
                          columns='Variável',
                          values='Valor',
                          aggfunc='sum')

# Resetar o índice para tornar as colunas pivotadas em colunas normais
pivot_df = pivot_df.reset_index()


pivot_df.to_excel(path_pivot_excel, index=False, header=True)

print(f"Arquivo Excel com valores pivotados salvos em: '{path_pivot_excel}' com sucesso.")
```


* ### 5.3 Analisando o dataframe pivotado
pivot_df.sample(3)
![alt text](./Imagens/image.png)
pivot_df.info(3)
![alt text](./Imagens/image-1.png)
pivot_df.describe()
![alt text](./Imagens/image-2.png)

## 6. Gerando gráficos para análises - Variação Mensal
Plotar um gráfico de barras do Estado e a variável de variação mês/mês da Indústria Geral para identificar qual estado apresentou crescimento com relação ao mês anterior (Fevereiro-2024)

```Python
# Gerando gráficos com matplotlib, variação mês/mês dos estados com relação ao mês anterior
pivot_df_order = pivot_df.sort_values(by='Unidade da Federação', ascending=True)

# Filtrar o dataset pelo CNAE de indústria geral
pivot_df_filtrado = pivot_df_order[(pivot_df_order['Seções e atividades industriais (CNAE 2.0)'].str.contains('1 Indústria geral', case=False))]

# Criar o gráfico de barras
plt.figure(figsize=(10, 6))
bars = plt.bar(pivot_df_filtrado['Unidade da Federação'], pivot_df_filtrado['Variação mês/mês imediatamente anterior, com ajuste sazonal (M/M-1)'], color='skyblue')

# Adicionar rótulos e título
plt.xlabel('Unidade da Federação')
plt.ylabel('Variação mês/mês com ajuste sazonal (M/M-1)')
plt.title('Variação Mês/Mês por Unidade da Federação na Indústria Geral - Março/2024')

# Adicionar valores nas barras
for bar in bars:
    yval = bar.get_height()  # Obtém a altura da barra
    plt.text(bar.get_x() + bar.get_width()/2, yval, yval, ha='center', va='bottom')

# Mostrar o gráfico
plt.xticks(rotation=45)  # Rotaciona os rótulos do eixo x para melhorar a legibilidade
plt.tight_layout()  # Ajusta o layout para evitar sobreposição de elementos
plt.show()
```
![alt text](./Imagens/image-4.png)

Estado do Pará, seguido de Maranhão e Rio de Janeiro foram os estados com maior crescimento industrial com relação ao mês anterior.

## 7. Gerando gráficos para análises - Variação Anual Acumulada

Criar um gráfico de barras horizontais para verificar qual cnae teve maior crescimento com relação a indústria.

```Python
# Gerando gráficos com matplotlib, variação acumulada em 12 meses em relação ao periodo anterior das cnaes
pivot_df_order = pivot_df.sort_values(by='Seções e atividades industriais (CNAE 2.0)', ascending=False)
pivot_df_agrupado = pivot_df_order.groupby('Seções e atividades industriais (CNAE 2.0)')['Variação acumulada em 12 meses (em relação ao período anterior de 12 meses)'].sum().reset_index()


# Criar o gráfico de barras
plt.figure(figsize=(10, 6))
bars = plt.barh(pivot_df_agrupado['Seções e atividades industriais (CNAE 2.0)'], pivot_df_agrupado['Variação acumulada em 12 meses (em relação ao período anterior de 12 meses)'].round(2), color='skyblue')

# Adicionar rótulos e título
plt.xlabel('Variação Mês/Mês')
plt.ylabel('Seções e atividades industriais (CNAE 2.0)')
plt.title('Variação Acumulada em 12 Meses de Seções e atividades industriais (CNAE 2.0) - Março/2024')

# Adicionando os valores nas barras
for bar in bars:
    plt.text(bar.get_width() + 0.1,   # Posição x do texto
             bar.get_y() + bar.get_height() / 2,  # Posição y do texto
             f'{bar.get_width()}',    # Texto a ser exibido (o valor da barra)
             ha='center', va='center')  # Alinhamento do texto

# Mostrar o gráfico
plt.xticks(rotation=45)  # Rotaciona os rótulos do eixo x para melhorar a legibilidade
plt.tight_layout()  # Ajusta o layout para evitar sobreposição de elementos
plt.show()
```
![alt text](./Imagens/image-5.png)

Cnae 3.19 - Fabricação de coque, de produtos derivados do petróleo e de biocombustíveis teve maior expressão de crescimento acumulado com relação aos ultimos 12 meses.

## 8. Gerando gráficos para análises - Variação Anual Acumulada do Estado

Gerar um gráfico de calor para exemplificar qual cnae por estado com maior variação anual.

```Python
# Agrupando os valores por 'Seções e atividades industriais (CNAE 2.0)'
pivot_df_agrupado = pivot_df_order.groupby(['Seções e atividades industriais (CNAE 2.0)','Unidade da Federação'])['Variação acumulada em 12 meses (em relação ao período anterior de 12 meses)'].sum().reset_index()

# Pivotando os dados para o formato necessário para o heatmap
heatmap_data = pivot_df_agrupado.pivot_table(index='Seções e atividades industriais (CNAE 2.0)', columns='Unidade da Federação', values='Variação acumulada em 12 meses (em relação ao período anterior de 12 meses)')

# Criando a figura
plt.figure(figsize=(10, 6))

# Plotando o mapa de calor
heatmap = plt.imshow(heatmap_data, cmap='viridis', interpolation='nearest')

# Adicionando a barra de cores
cbar = plt.colorbar(heatmap)
cbar.set_label('Variação acumulada em 12 meses')

# Adicionando rótulos e título
plt.xlabel('Unidade da Federação')
plt.ylabel('Seções e atividades industriais (CNAE 2.0)')
plt.title('Mapa de Calor - Variação Acumulada em 12 meses por CNAE e UF')

# Ajustando os ticks
plt.xticks(ticks=np.arange(heatmap_data.columns.size), labels=heatmap_data.columns, rotation=90)
plt.yticks(ticks=np.arange(heatmap_data.index.size), labels=heatmap_data.index)

# Adicionando anotações (valores) nas células
# for i in range(heatmap_data.shape[0]):
#     for j in range(heatmap_data.shape[1]):
#         value = heatmap_data.iloc[i, j]
#         if not np.isnan(value):
#             plt.text(j, i, f'{value:.2f}', ha='center', va='center', color='white')

plt.tight_layout()
plt.show()
```
![alt text](./Imagens/image-6.png)

Estado que mais teve maior variação foi Pernambuco com 3.30 Fabricação de outros equipamentos de transporte, exceto veículos automotores.
