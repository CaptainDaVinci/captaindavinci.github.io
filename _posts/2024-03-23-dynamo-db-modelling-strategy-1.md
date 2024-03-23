---
layout: post
title: "Dynamo DB - Modelling strategies: one-to-many"
description: "Dynamo DB - Modelling One to many relationships"
meta: "dynamodb,modelling,strategy,one-to-many relationship"
---

## Strategies for modelling One-to-many relationships

One-to-many relationship is best understood with an example, consider two entities Employee and Department, one department can have several employees and each
employee is part of only a single department thus establishing a one-to-many relatonship between the department and employees.

On the contrary, many-to-many relationship can be understood using Student and Class entities, one student can be part of several different classes and each class can
have several students.

Generally speaking, modelling one-to-many relationship is easier in dynamo DB compared to many-to-many. 

The strategies discussed here are taken from [Alex De Brie's - The DynamoDB handbook](https://www.amazon.com/DynamoDB-Book-Alex-DeBrie/dp/B0BN7S1GQK)

### Strategy 1: Denormalize by using a complex attribute

Consider a relation between a customer and their address for an e-commerce application. A customer can have multiple addresses, upto a certain limit. We can model this
one-to-many relationship by storing the address as a complex attribute `addresses` along with other customer details. 

This approach works under the assumptions that,

1. No query would need to filter on a particular field within the complex attribute.
2. There is a limit on the size upto which the complex attribute can grow to so that a single item does not go past the 400kb item limit.


### Strategy 2: Denormalize by duplicating data

Consider a relation between a department and employees where we want to fetch the employee details along with department information such as department name, cost center etc.
This can be modelled by duplicating this data in each employee item so that when fetching employee details we have the department details already populated.


| PK     | SK        | name     | dept_name |
|--------|-----------|----------|-----------|
| Dept#1 | EMP#ID_1  | ABC      | DeptName1 |
|        | EMP#ID_2  | XYZ      | DeptName2 |

This works under the assumption that the duplicated data is immutable. Otherwise, any change in that would lead to updating several items. Or, we need to be mindful about how often the data changes and how many items would be affected by the change.

### Strategy 3: Primary key + Query API action

Contintuing on the above example. The more common approach to model this type of relationship is using primary key + query API. 

| PK     | SK         | name     |
|--------|------------|----------|
| Dept#1 | #MetaData  | DeptName |
|        | EMP#ID_123 | EmpName  |


Here using the query API one can easily fetch the metadata and employee information using the `PK` and `SK` and it supports the following access patterns,
1. Get an employee
2. Get a department
3. Get all employees in a department

However, now if department metadata needed then an additional query is needed, this is a trade-off that one needs to consider.

**Note**: This can be modelled using secondary index as well, but I do not consider that as a different strategy, rather a different way of implementing the same strategy when the primary key of the table can't be re-used.


### Strategy 4: Composite sort key for storing heirarchical data 

Loction based data can be modelled by using a composite sort key. Consider a table which stores addresses of a store across the world

| PK    | SK                     | Address    |
|-------|------------------------|------------|
| INDIA | KARNATAKA#BLR#560001#1 | address... |
|       | KARNATAKA#BLR#560053#1 | address... |
| US    | NY#NEWYORKCITY#10019#1 | address... |

This type of modelling enables us to answer queries like,

1. Get all stores in a country.
2. Get all stores in a country & state.
3. Get all stores in a coutry, state & city.
4. Get all stores in a coutry, state, city & zip code.
