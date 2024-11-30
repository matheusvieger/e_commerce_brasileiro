# Projeto E-Commerce brasileiro
Este repositório abriga um projeto de banco de dados e toda sua documentação construtiva.
Ele servirá como nota final na disciplina de Data Prep & Transformation no curso de pós-graduação de Engenharia de Dados da Universidade Presbiteriana Mackenzie.


# Criação do Banco de Dados Transacional

## Passo 1: Baixamos o Conjunto de Dados do Kaggle
Esse conjunto de dados da Olist no Kaggle contém dados de 100.000 pedidos realizados em diversas lojas online de 2016 a 2018.

### Alguns Pontos Principais:
- **Pedidos**: Informações detalhadas sobre cada pedido, incluindo a data de compra, status do pedido e data de entrega.
- **Produtos**: Dados sobre os produtos vendidos, como categoria, preço e frete.
- **Clientes**: Informações demográficas e geográficas dos clientes, ajudando a entender quem são os compradores e de onde eles vêm.
- **Avaliações**: Feedback dos clientes sobre os produtos, incluindo notas e comentários.

## Passo 2: Escolhemos um SGBD

Escolhemos o MySQL como Sistema de Gerenciamento de Banco de Dados (SGBD) para este conjunto de dados da Olist por várias razões:

- **Popularidade e Suporte**: MySQL é um dos SGBDs mais populares no mundo, o que significa que há uma vasta comunidade de usuários e desenvolvedores prontos para ajudar, além de uma documentação abrangente.
- **Desempenho**: MySQL é conhecido por seu desempenho eficiente em consultas e transações, o que pode ser vantajoso ao trabalhar com grandes volumes de dados, como os presentes nesse conjunto.
- **Escalabilidade**: MySQL oferece boas opções de escalabilidade.

## Passo 3: Criamos as tabelas no banco de dados com base na documentação do Kaggle

Para criar as tabelas no banco de dados MySQL com base na documentação do Kaggle para o conjunto de dados da Olist, seguimos alguns passos básicos:

### 1. Criação do Banco de Dados:
```sql
CREATE DATABASE olist_db;
USE olist_db;

CREATE TABLE customers (
    customer_id VARCHAR(255) NOT NULL PRIMARY KEY,
    customer_unique_id VARCHAR(255) NOT NULL,
    customer_zip_code_prefix INT NOT NULL,
    customer_city VARCHAR(255) NOT NULL,
    customer_state CHAR(2) NOT NULL
);

CREATE TABLE orders (
    order_id VARCHAR(255) NOT NULL PRIMARY KEY,
    customer_id VARCHAR(255) NOT NULL,
    order_status VARCHAR(255) NOT NULL,
    order_purchase_timestamp DATETIME NOT NULL,
    order_approved_at DATETIME NOT NULL,
    order_delivered_carrier_date DATETIME NOT NULL,
    order_delivered_customer_date DATETIME NOT NULL,
    order_estimated_delivery_date DATETIME NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE products (
    product_id VARCHAR(255) NOT NULL PRIMARY KEY,
    product_category_name VARCHAR(255) NOT NULL,
    product_name_length INT NOT NULL,
    product_description_length INT NOT NULL,
    product_photos_qty INT NOT NULL,
    product_weight_g FLOAT NOT NULL,
    product_length_cm FLOAT NOT NULL,
    product_height_cm FLOAT NOT NULL,
    product_width_cm FLOAT NOT NULL
);

CREATE TABLE order_items (
    order_id VARCHAR(255) NOT NULL,
    order_item_id INT NOT NULL,
    product_id VARCHAR(255) NOT NULL,
    seller_id VARCHAR(255) NOT NULL,
    shipping_limit_date DATETIME NOT NULL,
    price FLOAT NOT NULL,
    freight_value FLOAT NOT NULL,
    PRIMARY KEY (order_id, order_item_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE order_reviews (
    review_id VARCHAR(255) NOT NULL PRIMARY KEY,
    order_id VARCHAR(255) NOT NULL,
    review_score INT NOT NULL,
    review_comment_title TEXT NOT NULL,
    review_comment_message TEXT NOT NULL,
    review_creation_date DATETIME NOT NULL,
    review_answer_timestamp DATETIME NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

## Passo 4: Implementamos um script para carregar os dados nas tabelas criadas

Para implementar um script que carregue os dados nas tabelas criadas no MySQL, nós usamos Python junto com a biblioteca pandas para manipulação de dados e a mysql-connector-python para a conexão com o banco de dados:

### 1. Instalamos as Bibliotecas Necessárias:
```sh
pip install pandas mysql-connector-python

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
```

## 2. Modelagem de Banco de Dados Analítico

### Star Schema:

#### Passo 1: Criamos a Tabela Fato
Criamos uma tabela fato que agregue as vendas por ano e mês. Esta tabela conterá as principais métricas que desejamos analisar, como o total de vendas.
```sql
CREATE TABLE fact_sales (
    date_id INT NOT NULL,
    total_sales DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (date_id),
    FOREIGN KEY (date_id) REFERENCES dim_date(date_id)
);

CREATE TABLE dim_date (
    date_id INT NOT NULL PRIMARY KEY,
    year INT NOT NULL,
    month INT NOT NULL,
    day INT NOT NULL
);

CREATE TABLE dim_product (
    product_id VARCHAR(255) NOT NULL PRIMARY KEY,
    product_category_name VARCHAR(255) NOT NULL
);

CREATE TABLE dim_customer (
    customer_id VARCHAR(255) NOT NULL PRIMARY KEY,
    customer_city VARCHAR(255) NOT NULL,
    customer_state CHAR(2) NOT NULL
);

CREATE TABLE dim_seller (
    seller_id VARCHAR(255) NOT NULL PRIMARY KEY,
    seller_city VARCHAR(255) NOT NULL,
    seller_state CHAR(2) NOT NULL
);


CREATE TABLE wide_sales (
    year INT NOT NULL,
    month INT NOT NULL,
    product_category_name VARCHAR(255) NOT NULL,
    customer_city VARCHAR(255) NOT NULL,
    customer_state CHAR(2) NOT NULL,
    seller_city VARCHAR(255) NOT NULL,
    seller_state CHAR(2) NOT NULL,
    total_sales DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (year, month, product_category_name, customer_city, customer_state, seller_city, seller_state)
);


pip install pandas mysql-connector-python


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

# Função para Carregar Dados em uma Tabela
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
load_data('wide_sales', 'caminho/para/wide_sales.csv')

# Fechar a conexão
cursor.close()
conn.close()
```

## 3. Fluxo de Transformação de Dados

| Passo | Ação                           | Descrição                                                                                                            | Ferramentas                    |
|-------|--------------------------------|----------------------------------------------------------------------------------------------------------------------|--------------------------------|
| 1     | Extração dos Dados             | Conectar ao banco de dados transacional e extrair dados das tabelas olist_customers_dataset, olist_orders_dataset, olist_products_dataset, olist_order_items_dataset, olist_order_reviews_dataset. | Python (Pandas, MySQL Connector) |
| 2     | Transformação dos Dados (Star Schema) | - Criar tabelas de dimensões (dim_date, dim_product, dim_customer). <br> - Agregar dados e criar a tabela fato (fact_sales). | Python (Pandas)               |
| 3     | Transformação dos Dados (Wide Table) | - Unificar dados de várias tabelas. <br> - Agregar dados por ano e mês para criar a tabela ampla (wide_sales).        | Python (Pandas)               |
| 4     | Carga dos Dados Transformados  | Conectar ao banco de dados analítico e carregar dados nas tabelas (dim_date, dim_product, dim_customer, fact_sales, wide_sales). | Python (Pandas, MySQL Connector) |


## Descrição do Fluxograma

- **Início**: Início do processo de ETL.
- **Conexão ao BD Transacional**: Conectar ao banco de dados transacional.
- **Extração dos Dados**: Extrair os dados das tabelas `olist_customers_dataset`, `olist_orders_dataset`, `olist_products_dataset`, `olist_order_items_dataset`, `olist_order_reviews_dataset`.
- **Transformação dos Dados**: Transformar os dados para os formatos necessários.
  - **Transformação - Star Schema**:
    - Criar tabelas de dimensões (`dim_date`, `dim_product`, `dim_customer`).
    - Criar tabela fato (`fact_sales`).
  - **Transformação - Wide Table**:
    - Unificar dados de várias tabelas.
    - Agregar dados por ano e mês (`wide_sales`).
- **Carga no BD Analítico**: Carregar os dados transformados no banco de dados analítico.
- **Fim**: Conclusão do processo de ETL.


## 4. Seleção da Abordagem de Tempestividade

Para selecionar a abordagem de tempestividade mais adequada ao processar o Brazilian E-Commerce Public Dataset by Olist usando Python e MySQL, precisamos considerar as características do conjunto de dados e as necessidades de análise.

### Abordagens de Tempestividade

#### 1. Batch
- **Descrição**: Processamento em grandes lotes, executado em intervalos definidos (diariamente, semanalmente, etc.).
- **Vantagens**: Ideal para grandes volumes de dados que não precisam de atualização em tempo real. Simplifica o gerenciamento e a execução do processo ETL.
- **Desvantagens**: Menor tempestividade, pois os dados não são atualizados com frequência.

#### 2. Micro-Batch
- **Descrição**: Processamento em pequenos lotes, com atualizações mais frequentes (por exemplo, a cada hora).
- **Vantagens**: Oferece um bom equilíbrio entre tempestividade e carga de processamento. Dados mais atualizados em comparação ao batch.
- **Desvantagens**: Pode exigir mais recursos computacionais do que o processamento em batch.

#### 3. Fluxo Contínuo
- **Descrição**: Processamento em tempo real, onde os dados são processados conforme chegam.
- **Vantagens**: Ideal para aplicações que requerem dados constantemente atualizados.
- **Desvantagens**: Requer uma infraestrutura mais robusta e pode ser mais complexo de implementar e manter.

### Escolha da Abordagem: Micro-Batch

**Justificativa**: Para o Brazilian E-Commerce Public Dataset by Olist, a abordagem de Micro-Batch é a mais adequada pelos seguintes motivos:
1. **Volume de Dados**: O dataset é grande, mas os dados de comércio eletrônico normalmente não exigem atualização em tempo real. Atualizações horárias ou até mesmo diárias são suficientes para a maioria das análises.
2. **Recursos Computacionais**: O micro-batch proporciona um equilíbrio entre a frequência de atualização e a carga de processamento, otimizando o uso de recursos.
3. **Flexibilidade**: Permite ajustar a frequência de processamento conforme necessário, oferecendo flexibilidade para aumentar ou diminuir a frequência com base nas necessidades.
4. **Complexidade**: É menos complexo de implementar e manter em comparação ao fluxo contínuo, especialmente ao usar Python e MySQL.

### Implementação do Micro-Batch

**Agendar a Execução do Script**: Usar um agendador de tarefas como cron (Linux) ou Task Scheduler (Windows) para executar o script Python em intervalos definidos.

**Script Python de Processamento**:
```python
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
```
