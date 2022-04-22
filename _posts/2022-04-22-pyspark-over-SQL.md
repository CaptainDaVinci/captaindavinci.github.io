---
layout: post
title: "PySpark over SQL for ETL pipelines"
description: "Moving from SQL First to PySpark First Approach for Developing Pipelines"
meta: "project"
---

Most of the data pipelines today are built with a SQL first approach where we have a bunch of SQL queries disguised under python scripts. 
The python code in these scripts is basically orchestrating the data flow and you’ll often find `spark.sql` statements doing all the magical transformations.

Overtime I’ve found this approach to be tedious and maybe difficult to maintain in the long run and I hope to share with you some of the drawbacks which I’ve observed with using a SQL first approach.

## Re-usability 

SQL offers limited options when it comes to writing modular queries and it’s no surprise. Sure there are CTEs, but they aren't as flexible as the good old function/methods
you'll find in any programming language. 

For instance, let’s say you have a table with a large list of columns that use the same set of transformations – a bunch of case-when-thens for pivoting data in a column,

A SQL might look like

```sql
select 
  case
    when tag like '%class_a%' then 1
    else 0
  end class_a,
  case
    when tag like '%class_b%' then 1
    else 0
  end class_b,
  ... so on
from table
```


In PySpark the same might look like,

```python
for tag in tags:
    df = df.withColumn(tag, F.when(F.col('tag').like(f'%{tag}%'), 1).otherwise(0))
```


 I can hear you say, no one really writes those SQL transformations, we generate them from a template!

### Issue with using templates to improve re-usability

To improve re-usability we may use templates and you’ll often find code snippets which are generating the SQL statements during runtime. 
In case of the above example it may look something like,

```sql
base_case_stmt = 'case when tag like "%{class}%" then 1 else 0 end {class}'
case_stmts = '\n,'.join([cast_stmt.format(class=c) for c in tags])
sql = 'select {} from table'.format(case_stmts)
```

There are a couple of issues with this approach:

* It makes it difficult to catch errors early on – if let’s say you make a typo in the template, the error will be thrown only when the query is executed, 
and if it’s a long chain of transformations, that may be much later down the line. But in-case of PySpark based approach an IDE will be able to hint these types of error as soon as you err.

* It makes the transformation harder to understand and modify – the above example is a simple one, but in a complex chain of transformations understanding 
the existing query and modifying it are needlessly complicated in a template based approach. 

I’d argue that using a runtime generated transformation is a dangerous approach when compared to using a static SQL. You don’t get the benefit of declarative nature of SQL and neither do you get the complete benefits of using PySpark’s APIs.


## Complex Queries can be hard to understand

There are no variables in SQL or functions that can help break the query down into smaller chunks. 
You’ll find a long list of columns often being copied across CTEs and temporary views. 
CTEs and creating temporary view help to a certain extent but even they can be difficult to follow without guiding comments or docs. 

PySpark on the other hand is able to solve this issue by breaking the transformations into smaller chunks of methods, which if reasonably well developed, I’d say can offer much more clarity than best written SQL queries.

## Unit testing 

Unit tests are extremely helpful and I can’t stress this enough, I won’t go into details of why here, but I’ll leave this <https://u-tor.com/topic/unit-testing-importance> article for you to refer. 

Any modern programming language has extensive set libraries and tooling already available for creating unit tests. If we the pipelines are developed using pure functions, 
testing it just becomes a matter of providing a sample input data and expected output data. 

Consider the previous example of pivoting a table :

```python
def get_pivoted_tags_df(df, tags):
  for tag in tags:
    df = df.withColumn(tag, F.when(F.col('tag').like(f'%{tag}%'), 1).otherwise(0))
  return df

def test_get_pivoted_tags(df):
  sample_df = spark.createDataframe([('a, b'), ('b')], columns=['tag'])
  expected_df = spark.createDataframe([(1, 1), (0, 1)], columns=['a', 'b'])
  
  assert expected_df == get_pivoted_tags_df(sample_df, ['a', 'b'])
```

## Code Quality Standards

At present ETL pipelines are a mix of python scripts with SQL statements in them and adding any sort of code linting or style guide like PEP, SQL Fluff will offer little value at the moment. 

## Conclusion

I think to make full use of software development best practices we need to also move to a PySpark based approach to developing pipelines.

### So is SQL dead?

Absolutely not, I think every tool has it’s place in the tech ecosystem. You might find yourself using SQL:

* In environments that don’t support PySpark, e.g, viz tools – Tableau, Redash.
* If the pipeline is going to be worked upon by teams that don’t have experience working with Python or similar programming languages. 
* Since SQL is declarative, it makes it possible to define the query and pass it to services or config based tools.
