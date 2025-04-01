# Projeto Conceitual de Banco de Dados - "E-COMMERCE"

### COMO ME ENCONTRAR?
[![LinkedIn](https://img.shields.io/badge/LinkedIn-000000?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/rafaeloliveirarso/) 
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/rafaeloliveirarso)
[![Gmail](https://img.shields.io/badge/Gmail-000000?style=for-the-badge&logo=gmail&logoColor=red)](mailto:rafael.silvaoliveira1992@gmail.com)

### REFINAMENTO DO MODELO RELACIONAL

Foram realizadas as alterações no esquema conceitual para que o mesmo refletisse o modelo relacional.

Foi criada uma tabela "payment", onde ela recebe um atributo "idPayment" e um atributo "typePayment", que recebem os dados de pagamento do pedido, independente da forma escolhida pelo cliente.

O scrip do BD e das Queries estarão aqui abaixo para uma rápida consulta, mas o arquivo final com todos os scripts tbm constará na pasta do trabalho, junto com todos os scripts de registros.

### SCRIPT BD
    create database ecommerce;
    use ecommerce;

    -- criação tabela cliente
    create table client(
        idClient int auto_increment primary key,
        Fname varchar(10),
        Minit char(3),
        Lname varchar(20),
        CPF char(11) not null,
        Address varchar(40),
        constraint unique_cpf_client unique (CPF)
    );

    -- criação tabela produto
    create table product(
        idProduct int auto_increment primary key,
        Pname varchar(10) not null,
        classification_kids bool default false,
        category enum('Eletronico','Vestimenta','Brinquedos','Alimentos','Móveis') not null,
        rating float default 0,
        size varchar(10)
    );

    -- criação tabela pagamento
    create table payment(
        idPayment int auto_increment primary key,
        idClient int,
        typePayment enum('Boleto','Cartão','Dois Cartões'),
        limitAvailable float,
        constraint fk_payment_client foreign key (idClient) references client(idClient)
    );

    -- criação tabela pedido
    create table orders(
        idOrder int auto_increment primary key,
        idOrderClient int,
        idPayment int,
        orderStatus enum('Cancelado','Confirmado','Em Processamento') default 'Em Processamento',
        orderDescription varchar(255),
        sendValue float default 10,
        paymentCash boolean default false,
        trackingCode varchar(50),
        constraint fk_orders_client foreign key (idOrderClient) references client(idClient),
        constraint fk_orders_payment foreign key (idPayment) references payment(idPayment)
    );

    -- criação tabela estoque
    create table productStorage(
        idProdStorage int auto_increment primary key,
        storageLocation varchar(255),
        quantity int default 0
    );

    -- criação tabela fornecedor
    create table supplier(
        idSupplier int auto_increment primary key,
        SocialName varchar(255) not null,
        CNPJ char(15) not null,
        contact char(11) not null,
        constraint unique_supplier unique (CNPJ)
    );

    -- criação tabela vendedor
    create table seller(
        idSeller int auto_increment primary key,
        SocialName varchar(255) not null,
        AbstName varchar(255),
        CNPJ char(15),
        CPF char(11),
        location varchar(255),
        contact char(11) not null,
        constraint unique_cnpj_seller unique (CNPJ),
        constraint unique_cpf_seller unique (CPF)
    );

    -- criação tabela vendedor terceiros
    create table productSeller(
        idPseller int,
        idProduct int,
        prodQuantity int default 1,
        primary key (idPseller, idProduct),
        constraint fk_product_seller foreign key (idPseller) references seller(idSeller),
        constraint fk_product_product foreign key (idProduct) references product(idProduct)
    );

    -- criação tabela product order
    create table productOrder(
        idPoProduct int,
        idPorder int,
        poQuantity int default 1,
        poStatus enum('Disponível','Sem Estoque') default 'Disponível',
        primary key (idPoProduct, idPorder),
        constraint fk_productorder_product foreign key (idPoProduct) references product(idProduct),
        constraint fk_productorder_order foreign key (idPorder) references orders(idOrder)
    );

    -- criação tabela local estoque
    create table storageLocation(
        idLproduct int,
        idLstorage int,
        location varchar(255) not null,
        primary key (idLproduct, idLstorage),
        constraint fk_storage_location_product foreign key (idLproduct) references product(idProduct),
        constraint fk_storage_location_storage foreign key (idLstorage) references productStorage(idProdStorage)
    );

### SCRIPTS QUERIES
    -- Quantos pedidos foram feitos por cada cliente?
    
    select client.Fname, client.Lname, count(orders.idOrder) as TotalOrders
    from client
    join orders on client.idClient = orders.idOrderClient
    group by client.idClient;
    
    -- Algum vendedor também é fornecedor?
    
    select seller.SocialName as SellerName, supplier.SocialName as SupplierName
    from seller
    join supplier on seller.CNPJ = supplier.CNPJ or seller.CPF = supplier.contact;
    
    -- Relação de produtos fornecedores e estoques
    
    select product.Pname, supplier.SocialName, productStorage.storageLocation, productStorage.quantity
    from product
    join productSeller on product.idProduct = productSeller.idProduct
    join seller on productSeller.idPseller = seller.idSeller
    join supplier on seller.CNPJ = supplier.CNPJ
    join storageLocation on product.idProduct = storageLocation.idLproduct
    join productStorage on storageLocation.idLstorage = productStorage.idProdStorage;
    
    -- Relação de nomes dos fornecedores e nomes dos produtos
    
    select supplier.SocialName as SupplierName, product.Pname as ProductName
    from supplier
    join seller on supplier.CNPJ = seller.CNPJ
    join productSeller on seller.idSeller = productSeller.idPseller
    join product on productSeller.idProduct = product.idProduct;
    
    -- Quais produtos estão sem estoque?
    
    select product.Pname, productOrder.poStatus
    from product
    join productOrder on product.idProduct = productOrder.idPoProduct
    where productOrder.poStatus = 'Sem Estoque';
    
    -- Qual a quantidade total de cada produto em estoque?
    
    select product.Pname, sum(productStorage.quantity) as TotalQuantity
    from product
    join storageLocation on product.idProduct = storageLocation.idLproduct
    join productStorage on storageLocation.idLstorage = productStorage.idProdStorage
    group by product.idProduct;
    
    -- Recuperações simples com SELECT Statement

    -- Recuperar todos os clientes
    select * from client;

    -- Recuperar todos os produtos
    select * from product;
    
    -- Filtros com WHERE Statement
    
    -- Recuperar clientes com sobrenome 'Silva'
    select * from client
    where Lname = 'Silva';

    -- Recuperar produtos da categoria 'Eletronico'
    select * from product
    where category = 'Eletronico';

    -- Crie expressões para gerar atributos derivados:

    -- Calcular o valor total dos pedidos (sendValue + 10% de taxa)
    select idOrder, orderDescription, sendValue, (sendValue * 1.10) as TotalValue
    from orders;

    -- Calcular a média de avaliação dos produtos
    select category, avg(rating) as AverageRating
    from product
    group by category;
    
    -- Defina ordenações dos dados com ORDER BY:

    -- Ordenar clientes pelo sobrenome em ordem alfabética
    select * from client
    order by Lname;

    -- Ordenar produtos pelo preço (rating) em ordem decrescente
    select * from product
    order by rating desc;

    -- Condições de filtros aos grupos – HAVING Statement:

    -- Recuperar categorias de produtos com média de avaliação maior que 4.5
    select category, avg(rating) as AverageRating
    from product
    group by category
    having avg(rating) > 4.5;

    -- Recuperar clientes que fizeram mais de 2 pedidos
    select client.Fname, client.Lname, count(orders.idOrder) as TotalOrders
    from client
    join orders on client.idClient = orders.idOrderClient
    group by client.idClient
    having count(orders.idOrder) > 2;
    
    -- Crie junções entre tabelas para fornecer uma perspectiva mais complexa dos dados:
    
    -- Recuperar informações dos pedidos junto com os dados dos clientes
    select orders.idOrder, orders.orderDescription, client.Fname, client.Lname
    from orders
    join client on orders.idOrderClient = client.idClient;

    -- Recuperar informações dos produtos junto com os dados dos fornecedores
    select product.Pname, supplier.SocialName
    from product
    join productSeller on product.idProduct = productSeller.idProduct
    join seller on productSeller.idPseller = seller.idSeller
    join supplier on seller.CNPJ = supplier.CNPJ;