## Preql Quickstart

This guide introduces you to writing and querying with Preql. A preql model consists of two core definitions - concepts
and datasources. Whil Preql syntax is closely related to SQL, it also shares some core concepts with programming languages
such as python and should generally be more concise.

## Statements

Preql syntax is define as one of more statements. 

## Queries

The simplest statement in Preql consistes of the keyword `select` followed by one or more `concepts`. A concept
is often namespaced by importing the definition from another file, 
and those namespaces can be referenced with . notation.

A basic preql query:
```sql
select
    customer_name,
    total_sales,
;
```
Note: trailing commas are accepted.


In a larger model, this might be expressed as:

```sql
import concepts.customer as customer;
import concepts.finance as finance;

select
    customer.name,
    finance.total_sales,
;
```


To be referenced, a concept first must be defined. A concept can have one of 3 purposes and have a 
well-defined type

- a key
- a property
- a metric

You can read more about each purpose later on in this guide, but broadly:

- A key uniquely identifies something. 
- A property is an attribute associated with a given key that may not be unique. 
- A metric is an aggregate computation off of a key or property.

A concept is typically defined via a statement of the form `[type]` `[name]` `[purpose]` `[optional metadata]`,
but can be created through other syntax as well. These will be covered in depth later on.

Notably, metrics are always defined as the result of a *function*, in the form

metric `[name]` <- `function`( `arguments` )

For the basic query above, the concepts may look something like the below

```python
key order_key int;
property order_key.customer_name string;
property order_key.sale_amount float;
metric total_sale_amount <-sum(sale_amount);
```

A full preql file will require declarations of all metrics used, and will be validated at parse time to ensure that
the types and operations are valid. 

For example, in the below model it's required that the property passed into the sum() function is 

```python
key order_key int;
property order_key.customer_name string;
property order_key.sale_amount float;
metric total_sale_amount <-sum(sale_amount);

select
customer_name,
total_sales,
;

```

## Datasources

Though the above model is parseable, you wouldn't actually be able to generate SQL from it. Generating SQL 
requires a `datasource` to be defined. A datasource maps a logical concept to a physical instantiation of it.

A datasource declaration is straightforward; the keyword `datasource`, a unique name, and then a parenthetical
list of assignments of columns on the table to concepts. 

Lastly, the 'address' property defines the name that needs to be rendered into SQL. 

```datasource fact_orders (
    orderID:order_key,
    customerName: customer_name,
    amount:sale_amount,
    )
    address sales.tblOrderData
;
```

