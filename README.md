# Data Analyst Portfolio Project 2 – Data Base Sakila

## Introdução

O banco de dados Sakila é um esquema perfeitamente normalizado que modela uma loja de aluguel de DVD 's de filmes, no qual seus schemas apresentam informações como: atores, relação filme-ator e uma mesa de inventário central que conecta filmes, lojas, clientes e aluguéis.

### Download Banco de dados Sakila
- O mesmo pode ser baixado no link: [Clique aqui](https://dev.mysql.com/doc/index-other.html)

- Vá no quadro **Example Databases**.
	Escolha o documento de nome `sakila database`


A estrutura do banco de dados Sakila contém informações pertinentes, como:
- Relacionamento muitos para muitos.
- Múltiplos caminhos entre entidades (por exemplo, filme-estoque-aluguel-pagamento vs filme-estoque-loja-cliente-pagamento) para praticar junções.
- Nomenclatura consistente de colunas:
  -  As chaves primárias são chamadas de `[nome da tabela] _ [id]`
  - As chaves estrangeiras são chamadas como sua chave primária referenciada, se possível. Isso permite o uso da sintaxe JOIN .. USING onde for compatível.
  - As tabelas de relacionamento não têm chaves substitutas, mas usam chaves primárias compostas
  - Cada tabela tem uma coluna de última atualização `[last_update]`.

## Objetivo da Análise

Analisando todas as tabelas do banco de dados Sakila, identifiquei que era possível fazer uma suposta análise preditiva, fazendo com que esses dados sejam utilizados para melhorar as estratégias de marketing, direcionando conteúdo com alto valor agregado para grupos segmentados, como por exemplo:
Fazer uma promoção para aqueles clientes que mais alugaram DVD’s.
Para um país que tenha grande quantidade de DVD’s alugados.
Ou até mesmo fazer investimento diferente nas lojas que cada Staff é responsável. 

Reparem que não coloquei estratégias para conseguir novos clientes, mas sim fidelizar aqueles que 	conhecem a “empresa” , como já dizia **Philip Kotler – “Pai do marketing”**:

**“Já não basta simplesmente satisfazer clientes: é preciso encantá-los.
Se você criar um caso de amor com seus clientes, eles próprios farão sua publicidade.”**

Mesmo que seja apenas uma análise hipotética (até porque ninguém aluga DVD mais) esse projeto terá vários insight que podem ser utilizados em qualquer setor, seja para fidelizar clientes ou conseguir novos clientes.

Abaixo segue requerimento para conclusão da análise:

|Staff | Objetivo | Análise | Critérios de aceitação|
| :--: |     :--: |    :--: |                  :--: |
|Mike Hillyer|Visão geral detalhada da quantidade que cada loja faturou para estratégias de marketing.|Acompanhamento dos clientes que mais alugaram DVd’s, nome, endereço, país, etc.|Um painel do Power BI que me permite filtrar os dados de cada Staff, classe de filme, faturamento, etc.
|Jon Stephens|Visão geral detalhada da quantidade que cada loja faturou para estratégias de marketing.|Acompanhamento dos clientes que mais alugaram DVd’s, nome, endereço, país, etc.|Um painel do Power BI que me permite filtrar os dados de cada Staff, classe de filme, faturamento, etc.|

## Limpeza e Transformação de Dados (SQL)
Nessa etapa foi feito a extração e limpeza dos dados utilizando linguagem SQL, no qual o propósito era seguir o requerimento visto na tabela acima.
Identificando dados para os futuros anúncios da equipe de marketing.

### Abaixo estão as instruções SQL para limpar e transformar os dados necessários.

#### Número de vezes que cada filme foi alugado.
``` 
select f.title Título, count(i.inventory_id) 'Vezes alugado' 
from inventory i
	join film f 
		on i.film_id = f.film_id 
group by f.film_id
having count(i.inventory_id) = 8
order by count(i.inventory_id) desc;
```
Sabendo que a maior quantidade de vezes que um filme foi alugado é de 8 vezes, fiz a consulta de todos 
aqueles que tem quantidade igual a 8. Não limitando por top 10 ou top 20.

Para saber a quantidade de cada filme sem limitação, basta retirar a linha que está descrito na consulta:
having count(i.inventory_id) = 8 

#### Output

#### Total de vezes que cada categoria de filme foi alugado.
```
select c.name Categoria, count(i.inventory_id) 'Total de vezes alugado' 
from inventory i
	join film f 
		on i.film_id = f.film_id
	join film_category fc 
		on f.film_id = fc.film_id 
	join category c 
		on fc.category_id = c.category_id 
group by c.name 
order by count(i.inventory_id) desc;

```
Consulta feita pelas categorias dos filmes, obtendo o resultado de vezes que cada tipo de filme foi alugado.

#### Output

#### Top 10 Países que mais tiveram arrecadação com o aluguel de filmes.
```
select c3.country País, sum(p.amount) 'Valor Arrecadado' 
from payment p
	join customer c 
		on p.customer_id = c.customer_id 
	join address a 
		on c.address_id = a.address_id 
	join city c2 
		on a.city_id = c2.city_id 
	join country c3 
		on c2.country_id = c3.country_id 
group by c3.country 
order by sum(p.amount)desc
limit 10;

```
#### Output

#### Valor total arrecadado. 
```
select sum(amount) 'Arrecadação Total' from payment p ;
```
Para comprovação da consulta **"Top 10 Países que mais tiveram arrecadação com o aluguel de filmes."**, decide colocar a query do valor total de arrecadação.
#### Output

#### Top 20 Cliente que mais alugaram DVD's.
```
select p.customer_id 'Id cliente',concat(c.first_name," ",c.last_name) 'Nome Completo', 
c.email 'E-mail',  count(*) 'Quant. de vezes que alugou' 
from payment p
	join customer c 
		on p.customer_id = c.customer_id 
	join address a 
		on c.address_id = a.address_id 
	join city c2 
		on a.city_id = c2.city_id 
	join country c3 
		on c2.country_id = c3.country_id 
group by p.customer_id
order by count(*) desc
limit 20;
```
Informações dos top 20 clientes que mais alugaram filmes.
#### Output


#### Loja que mais alugou filmes e o faturamento de cada uma.
```
select s2.first_name 'Responsavel da Loja', p.rental_id 'Quant. de Alugueis', sum(p.amount) 'Valor Total'
from store s
	join staff s2 
		on s.store_id = s2.store_id 
	join payment p 
		on s2.staff_id = p.staff_id
group by s2.first_name ;
```
#### Output

## Modelo de Dados
#### Abaixo está uma captura de tela do modelo de dados e as conexões das tabelas.

![tabela_conectada](https://github.com/djalmarodriguess/Data_Analystic_Project_2/blob/main/Sakila_structure.png)

## Painel de gerenciamento de filmes alugados.

