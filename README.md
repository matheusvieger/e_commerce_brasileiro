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
- **Escalabilidade**: MySQL oferece boas opções de escalabilidade, tanto

## Passo 3: Criamos as tabelas no banco de dados com base na documentação do Kaggle

Para criar as tabelas no banco de dados MySQL com base na documentação do Kaggle para o conjunto de dados da Olist, seguimos alguns passos básicos:

### 1. Criação do Banco de Dados:
```sql
CREATE DATABASE olist_db;
USE olist_db;

CREATE TABLE customers (
    customer_id VARCHAR(255) PRIMARY KEY,
    customer_unique_id VARCHAR(255),
    customer_zip_code_prefix INT,
    customer_city VARCHAR(255),
    customer_state CHAR(2)
);

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
