# Projeto E-Commerce brasileiro
Este repositório abriga um projeto de banco de dados e toda sua documentação construtiva.
Ele servirá como nota final na disciplina de Data Prep & Transformation no curso de pós-graduação de Engenharia de Dados da Universidade Presbiteriana Mackenzie.


1. Criação do Banco de Dados Transacional
•	Passo 1: Baixamos o conjunto de dados do Kaggle
Esse conjunto de dados da Olist no Kaggle contém dados de 100.000 pedidos realizados em diversas lojas online de 2016 a 2018.
Alguns pontos principais:
Pedidos: Informações detalhadas sobre cada pedido, incluindo a data de compra, status do pedido e data de entrega.
Produtos: Dados sobre os produtos vendidos, como categoria, preço e frete.
Clientes: Informações demográficas e geográficas dos clientes, ajudando a entender quem são os compradores e de onde eles vêm.
Avaliações: Feedback dos clientes sobre os produtos, incluindo notas e comentários.

•	Passo 2: Escolhemos um SGBD 
Escolhemos o MySQL como Sistema de Gerenciamento de Banco de Dados (SGBD) para este conjunto de dados da Olist por várias razões:
Popularidade e Suporte: MySQL é um dos SGBDs mais populares no mundo, o que significa que há uma vasta comunidade de usuários e desenvolvedores prontos para ajudar, além de uma documentação abrangente.
Desempenho: MySQL é conhecido por seu desempenho eficiente em consultas e transações, o que pode ser vantajoso ao trabalhar com grandes volumes de dados, como os presentes nesse conjunto.
Escalabilidade: MySQL oferece boas opções de escalabilidade, tanto vertical quanto horizontal, permitindo que o banco de dados cresça conforme a quantidade de dados e o número de usuários aumentam.
Facilidade de Uso: Com uma interface amigável e uma curva de aprendizado relativamente suave, MySQL é acessível para iniciantes, mas também robusto o suficiente para usuários avançados.
Compatibilidade: MySQL é compatível com várias linguagens de programação e plataformas, o que facilita a integração com diferentes sistemas e ferramentas de análise de dados.
Recursos: MySQL oferece recursos avançados como replicação de dados, clustering e backup, que podem ser úteis para garantir a alta disponibilidade e a integridade dos dados.
Esses pontos fazem do MySQL uma escolha sólida para gerenciar o conjunto de dados da Olist, especialmente buscando uma solução confiável, eficiente e bem suportada.

•	Passo 3: Criamos as tabelas no banco de dados com base na documentação do Kaggle.
Para criar as tabelas no banco de dados MySQL com base na documentação do Kaggle para o conjunto de dados da Olist, seguimos alguns passos básicos:
1.	Criação  do banco de dados:
sql
CREATE DATABASE olist_db;
USE olist_db;

2.	Criação das tabelas:
Tabela de Clientes:
sql
CREATE TABLE customers (
    customer_id VARCHAR(255) PRIMARY KEY,
    customer_unique_id VARCHAR(255),
    customer_zip_code_prefix INT,
    customer_city VARCHAR(255),
    customer_state CHAR(2)
);

Tabela de Pedidos:
sql
CREATE TABLE orders (
    order_id VARCHAR(255) PRIMARY KEY,
    customer_id VARCHAR(255),
    order_status VARCHAR(255),
    order_purchase_timestamp DATETIME,
    order_approved_at DATETIME,
    order_delivered_carrier_date DATETIME,
    order_delivered_customer_date DATETIME,
    order_estimated_delivery_date DATETIME,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

Tabela de Produtos:
sql
CREATE TABLE products (
    product_id VARCHAR(255) PRIMARY KEY,
    product_category_name VARCHAR(255),
    product_name_length INT,
    product_description_length INT,
    product_photos_qty INT,
    product_weight_g FLOAT,
    product_length_cm FLOAT,
    product_height_cm FLOAT,
    product_width_cm FLOAT
);

Tabela de Itens de Pedido:
sql
CREATE TABLE order_items (
    order_id VARCHAR(255),
    order_item_id INT,
    product_id VARCHAR(255),
    seller_id VARCHAR(255),
    shipping_limit_date DATETIME,
    price FLOAT,
    freight_value FLOAT,
    PRIMARY KEY (order_id, order_item_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

Tabela de Avaliações:
sql
CREATE TABLE order_reviews (
    review_id VARCHAR(255) PRIMARY KEY,
    order_id VARCHAR(255),
    review_score INT,
    review_comment_title TEXT,
    review_comment_message TEXT,
    review_creation_date DATETIME,
    review_answer_timestamp DATETIME,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

•	Passo 4: Implementamos um script para carregar os dados nas tabelas criadas.
Para implementar um script que carregue os dados nas tabelas criadas no MySQL, nós  usamos Python junto com a biblioteca pandas para manipulação de dados e a mysql-connector-python para a conexão com o banco de dados:

1.	Instalamos as bibliotecas necessárias:
sh
pip install pandas mysql-connector-python

2.	Escrevemos o script para carregar os dados:
python
import pandas as pd
import mysql.connector

# Configurações da conexão com o banco de dados
db_config = {
    'user': 'seu_usuario',
    'password': 'sua_senha',
    'host': 'localhost',
    'database': 'olist_db'
}

# Conectar ao banco de dados
conn = mysql.connector.connect(**db_config)
cursor = conn.cursor()

# Função para carregar dados em uma tabela
def load_data(table_name, csv_file):
    data = pd.read_csv(csv_file)
    for row in data.itertuples(index=False):
        placeholders = ', '.join(['%s'] * len(row))
        columns = ', '.join(data.columns)
        sql = f"INSERT INTO {table_name} ({columns}) VALUES ({placeholders})"
        cursor.execute(sql, row)
    conn.commit()

# Carregar dados em cada tabela
load_data('customers', 'path/to/customers.csv')
load_data('orders', 'path/to/orders.csv')
load_data('products', 'path/to/products.csv')
load_data('order_items', 'path/to/order_items.csv')
load_data('order_reviews', 'path/to/order_reviews.csv')
# Fechar a conexão
cursor.close()
conn.close()

Passo a passo:
1.	Conexão com o banco de dados: Configura as credenciais de acesso ao MySQL.
2.	Leitura dos dados: Usa o pandas para ler os arquivos CSV com os dados das tabelas.
3.	Inserção dos dados: Usa um loop para inserir linha por linha os dados em cada tabela do banco de dados MySQL.
4.	Fechamento da conexão: Assegura que a conexão com o banco de dados é fechada após a inserção.


2. Modelagem de Banco de Dados Analítico
•	Star Schema:
Passo 1: Criamos a Tabela Fato
Criamos uma tabela fato que agregue as vendas por ano e mês. Esta tabela conterá as principais métricas que desejamos analisar, como o total de vendas.
sql
CREATE TABLE fact_sales (
    date_id INT,
    total_sales DECIMAL(10, 2),
    PRIMARY KEY (date_id),
    FOREIGN KEY (date_id) REFERENCES dim_date(date_id)
);

Passo 2: Criamos Tabelas Dimensão
As tabelas dimensão conterão dados descritivos relacionados aos produtos, clientes, vendedores, etc:
sql
CREATE TABLE dim_date (
    date_id INT PRIMARY KEY,
    year INT,
    month INT,
    day INT
);

CREATE TABLE dim_product (
    product_id VARCHAR(255) PRIMARY KEY,
    product_category_name VARCHAR(255)
);

CREATE TABLE dim_customer (
    customer_id VARCHAR(255) PRIMARY KEY,
    customer_city VARCHAR(255),
    customer_state CHAR(2)
);

CREATE TABLE dim_seller (
    seller_id VARCHAR(255) PRIMARY KEY,
    seller_city VARCHAR(255),
    seller_state CHAR(2)
);

Agora temos uma tabela fato (fact_sales) e tabelas dimensão (dim_date, dim_product, dim_customer, dim_seller) para análise.

•	Wide Table:
Para criarmos uma Wide Table, combinamos todas as informações relevantes em uma única tabela, agregada por ano e mês.
sql
CREATE TABLE wide_sales (
    year INT,
    month INT,
    product_category_name VARCHAR(255),
    customer_city VARCHAR(255),
    customer_state CHAR(2),
    seller_city VARCHAR(255),
    seller_state CHAR(2),
    total_sales DECIMAL(10, 2),
    PRIMARY KEY (year, month, product_category_name, customer_city, customer_state, seller_city, seller_state)
);

Passo a Passo para Carregar os Dados
#Instalamos as Bibliotecas Necessárias
sh
pip install pandas mysql-connector-python

#Criamos o Script em Python
python
import pandas as pd
import mysql.connector

# Configurações da conexão com o banco de dados
db_config = {
    'user': 'seu_usuario',
    'password': 'sua_senha',
    'host': 'localhost',
    'database': 'olist_db'
}

# Conectar ao banco de dados
conn = mysql.connector.connect(**db_config)
cursor = conn.cursor()

#Função para Carregar Dados em uma Tabela
python
def load_data(table_name, csv_file):
    data = pd.read_csv(csv_file)
    for row in data.itertuples(index=False):
        placeholders = ', '.join(['%s'] * len(row))
        columns = ', '.join(data.columns)
        sql = f"INSERT INTO {table_name} ({columns}) VALUES ({placeholders})"
        cursor.execute(sql, row)
    conn.commit()

# Carregar dados em cada tabela
load_data('dim_date', 'caminho/para/dim_date.csv')
load_data('dim_product', 'caminho/para/dim_product.csv')
load_data('dim_customer', 'caminho/para/dim_customer.csv')
load_data('dim_seller', 'caminho/para/dim_seller.csv')
load_data('fact_sales', 'caminho/para/fact_sales.csv')
load_data('wide_sales', 'caminho/para/wide_sales.csv'

# Fechar a conexão
cursor.close()
conn.close()


3. Fluxo de Transformação de Dados
Início --> Conexão ao BD Transacional 
Conexão ao BD Transacional --> Extração dos Dados
Extração dos Dados --> Transformação dos Dados 
Transformação dos Dados --> Transformação Star Schema 
Transformação dos Dados --> Transformação Wide Table 

Transformação Star Schema --> Criar Tabelas de Dimensões 
Criar Tabelas de Dimensões --> Criar Tabela Fato 
Criar Tabela Fato --> Carga no BD Analítico Star Schema 

Transformação Wide Table --> Unificar Dados
 Unificar Dados --> Agregar Dados por Ano e Mês
 Agregar Dados por Ano e Mês --> Carga no BD Analítico Wide Table 

Carga no BD Analítico Star Schema --> Fim
 Carga no BD Analítico Wide Table --> Fim


Descrição do Fluxo:
Início: Início do processo de ETL.
Conexão ao BD Transacional: Conectar ao banco de dados transacional.
Extração dos Dados: Extrair os dados das tabelas olist_customers_dataset, olist_orders_dataset, olist_products_dataset, olist_order_items_dataset, olist_order_reviews_dataset.
Transformação dos Dados: Transformar os dados para os formatos necessários.
Transformação - Star Schema:
	Criar tabelas de dimensões (dim_date, dim_product, dim_customer).
	Criar tabela fato (fact_sales).
Transformação - Wide Table:
	Unificar dados de várias tabelas.
	Agregar dados por ano e mês (wide_sales).
Carga no BD Analítico: Carregar os dados transformados no banco de dados analítico.
Fim: Conclusão do processo de ETL.

4. Seleção da Abordagem de Tempestividade
Para selecionar a abordagem de tempestividade mais adequada ao processar o Brazilian E-Commerce Public Dataset by Olist usando Python e MySQL, precisamos considerar as características do conjunto de dados e as necessidades de análise.

Abordagens de Tempestividade
1. Batch
•	Descrição: Processamento em grandes lotes, executado em intervalos definidos (diariamente, semanalmente, etc.).
•	Vantagens: Ideal para grandes volumes de dados que não precisam de atualização em tempo real. Simplifica o gerenciamento e a execução do processo ETL.
•	Desvantagens: Menor tempestividade, pois os dados não são atualizados com frequência.



2. Micro-Batch
•	Descrição: Processamento em pequenos lotes, com atualizações mais frequentes (por exemplo, a cada hora).
•	Vantagens: Oferece um bom equilíbrio entre tempestividade e carga de processamento. Dados mais atualizados em comparação ao batch.
•	Desvantagens: Pode exigir mais recursos computacionais do que o processamento em batch.

3. Fluxo Contínuo
•	Descrição: Processamento em tempo real, onde os dados são processados conforme chegam.
•	Vantagens: Ideal para aplicações que requerem dados constantemente atualizados.
•	Desvantagens: Requer uma infraestrutura mais robusta e pode ser mais complexo de implementar e manter.

Escolha da Abordagem: Micro-Batch

Justificativa: Para o Brazilian E-Commerce Public Dataset by Olist, a abordagem de Micro-Batch é a mais adequada pelos seguintes motivos:
1.	Volume de Dados: O dataset é grande, mas os dados de comércio eletrônico normalmente não exigem atualização em tempo real. Atualizações horárias ou até mesmo diárias são suficientes para a maioria das análises.
2.	Recursos Computacionais: O micro-batch proporciona um equilíbrio entre a frequência de atualização e a carga de processamento, otimizando o uso de recursos.
3.	Flexibilidade: Permite ajustar a frequência de processamento conforme necessário, oferecendo flexibilidade para aumentar ou diminuir a frequência com base nas necessidades.
4.	Complexidade: É menos complexo de implementar e manter em comparação ao fluxo contínuo, especialmente ao usar Python e MySQL.

Implementação do Micro-Batch
Agendar a Execução do Script: Usar um agendador de tarefas como cron (Linux) ou Task Scheduler (Windows) para executar o script Python em intervalos definidos.

Script Python de Processamento:
python
import mysql.connector
import pandas as pd

# Configurar a conexão com o banco de dados
db_config = {
    'user': 'seu_usuario',
    'password': 'sua_senha',
    'host': 'localhost',
    'database': 'olist_db'
}

# Conectar ao banco de dados transacional
trans_conn = mysql.connector.connect(**db_config)
# Conectar ao banco de dados analítico
analytical_conn = mysql.connector.connect(**db_config)

# Função para extração, transformação e carga
def etl_process():
    # Extração dos dados
    customers_df = pd.read_sql('SELECT * FROM olist_customers_dataset', trans_conn)
    orders_df = pd.read_sql('SELECT * FROM olist_orders_dataset', trans_conn)
    products_df = pd.read_sql('SELECT * FROM olist_products_dataset', trans_conn)
    order_items_df = pd.read_sql('SELECT * FROM olist_order_items_dataset', trans_conn)
    order_reviews_df = pd.read_sql('SELECT * FROM olist_order_reviews_dataset', trans_conn)

    # Transformação dos dados (exemplos de transformação para Star Schema e Wide Table)
    # ... (transformação conforme descrito anteriormente)

    # Função para carregar dados no banco analítico
    def load_data_to_db(table_name, dataframe, conn):
        cursor = conn.cursor()
        for row in dataframe.itertuples(index=False):
            placeholders = ', '.join(['%s'] * len(row))
            columns = ', '.join(dataframe.columns)
            sql = f"INSERT INTO {table_name} ({columns}) VALUES ({placeholders})"
            cursor.execute(sql, row)
        conn.commit()

    # Carregar dados nas tabelas
    load_data_to_db('dim_date', dim_date, analytical_conn)
    load_data_to_db('dim_product', dim_product, analytical_conn)
    load_data_to_db('dim_customer', dim_customer, analytical_conn)
    load_data_to_db('fact_sales', fact_sales, analytical_conn)
    load_data_to_db('wide_sales', wide_sales, analytical_conn)

# Executar o processo ETL
etl_process()

# Fechar as conexões
trans_conn.close()
analytical_conn.close()



