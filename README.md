Teste de SQL Intermediário
Consultas realizadas nas tabelas fornecidas

--•Lista de funcionários ordenando pelo salário decrescente.
SELECT * FROM VENDEDORES ORDER BY salario DESC;


--•Lista de pedidos de vendas ordenado por data de emissão.
SELECT * FROM PEDIDO ORDER BY data_emissao;


--•Valor de faturamento por cliente.
--CONSULTA CONSIDERANDO SOMENTE OS PEDIDOS FECHADOS
SELECT CLIENTES.razao_social, SUM(PEDIDO.valor_total) as faturamento FROM PEDIDO 
LEFT JOIN CLIENTES ON (CLIENTES.id_cliente = PEDIDO.id_cliente)
WHERE PEDIDO.situacao = "Fechado"
GROUP BY PEDIDO.id_cliente;


--•Valor de faturamento por empresa.
--CONSULTA CONSIDERANDO SOMENTE OS PEDIDOS FECHADOS
SELECT EMPRESA.razao_social, SUM(PEDIDO.valor_total) as faturamento FROM PEDIDO 
LEFT JOIN EMPRESA ON (EMPRESA.id_empresa = PEDIDO.id_empresa)
WHERE PEDIDO.situacao = "Fechado"
GROUP BY PEDIDO.id_empresa;


--•Valor de faturamento por vendedor.
--CONSULTA CONSIDERANDO SOMENTE OS PEDIDOS FECHADOS
SELECT VENDEDORES.id_vendedor, VENDEDORES.nome, SUM(PEDIDO.valor_total) AS faturamento
FROM PEDIDO
JOIN CLIENTES ON PEDIDO.id_cliente = CLIENTES.id_cliente
JOIN VENDEDORES ON CLIENTES.id_vendedor = VENDEDORES.id_vendedor
WHERE PEDIDO.situacao = "Fechado"
GROUP BY VENDEDORES.id_vendedor;

-- Consultas de Junção
WITH UltimosPedidos AS (
-- Subconsulta para obter os últimos pedidos de cada cliente
    SELECT 
        id_cliente,
        MAX(PEDIDO.id_pedido) AS ultimo_pedido
    FROM 
        PEDIDO
    GROUP BY 
        id_cliente
)

SELECT 
    IP.id_produto,
    PR.descricao AS descricao_produto,
    P.id_cliente,
    C.razao_social AS razao_social_cliente,
    P.id_empresa,
    E.razao_social AS razao_social_empresa,
    V.id_vendedor,
    V.nome AS nome_vendedor,
    CP.preco_minimo,
    CP.preco_maximo,
       CASE
           WHEN CP.preco_minimo > IP.preco_praticado THEN CP.preco_minimo
           WHEN CP.preco_maximo < IP.preco_praticado THEN CP.preco_maximo
           ELSE IP.preco_praticado
       END AS preco_base
FROM 
    UltimosPedidos UP
JOIN 
    PEDIDO P ON UP.id_cliente = P.id_cliente AND UP.ultimo_pedido = P.id_pedido
JOIN 
    ITENS_PEDIDO IP ON P.id_pedido = IP.id_pedido
JOIN 
    PRODUTOS PR ON IP.id_produto = PR.id_produto
JOIN 
    CLIENTES C ON P.id_cliente = C.id_cliente
JOIN 
    EMPRESA E ON P.id_empresa = E.id_empresa
JOIN 
    VENDEDORES V ON C.id_vendedor = V.id_vendedor
LEFT JOIN 
    CONFIG_PRECO_PRODUTO CP ON IP.id_produto = CP.id_produto;
