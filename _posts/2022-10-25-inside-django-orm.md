---
layout: post
title: "Notes from Conferences: Deep Inside Django's ORM: How Django Builds Queries"
description: "Notes from Conferences: Deep Inside Django's ORM: How Django Builds Queries"
date: 2022-10-25
tags: [conferences, django]
---

Talk: https://www.youtube.com/watch?v=OEN5wONsaYU

<br> 

### ORM Structure

* example query: `blogs = Blog.objects.all()`
* behind the scenes:
```python
#               Manager     
#         Model    |    Filter
#          _|_   __|__   |
#         /   \ /     \ / \ 
  blogs = Blog.objects.all()
# \___/
#   |
# Queryset
```

* design goals of ORM:
  * no SQL like words
  * no SQL strings
  * purely function calls 

<br> 

### Call Chain 
* classes which play role in the whole process:
  * Manager
  * QuerySet
  * Query
  * SQLCompiler   

<br> 

### Process 
Manager creates QuerySet
```python
# manager.py

class BaseManager:
  ...
  def get_queryset(self):
    """
    Return a new QuerySet object. Sublasses can
    override this method to customize the behavior
    of the Manager.
    """
    return self._queryset_class(model=self.model,
                                using=self._db,
                                hints=self._hints)
```

QuerySet creates Query
```python
# query.py

class QuerySet:
  """Represent a lazy database lookup
  for a set of objects."""
  ...
  @property
  def query(self):
      if self._deferred_filter:
          negate, args, kwargs = self._deferred_filter
          self._filter_or_exclude_inplace(negate,
                                          args,
                                          kwargs)
          self._deferred_filter = None
      return self._query 
```

Query passes Query to SQLCompiler
```python
# query.py

class Query:
    ...
    def sql_with_params(self):
    """
    Return the query as an SQL string and the parameters
    that will be substituted into the query.
    """
    compiler = self.get_compiler(DEFAULT_DB_ALIAS)
    return compiler.as_sql()
```

What does the compiler do?
```python
# compiler.py

class SQLCompiler:
    ...
    def get_select(self):
    """
    Return three values:
    - a list of 3-tuples of 
      (expression, (sql, params), alias)
    - a klass_info structure,
    - a dictionary of annotations
    
    The (sql, params) is what the expression will 
    produce, and alias is the "AS alias" for the 
    column (possible None).
    
    The klass_info structure contains the following information
    - The base model of the query.
    - Which columns for that model are present in the query (by
      position of the select clause).
    - related_klass_infos: [f, klass_info] to decent into
    
    The annotations is a dictionary of {'attname': column proposition} values.
    """
```
* *the base model of the query* -> table 
* *Which columns for that model are present in the query* -> important also for ordering, 
   as results from the database are enumerated nad not hashed like in a dictionary
* *related_klass_infos* -> important for compiling joins 

#### The Query Object
```python
qs = Blog.objects.all()
print(qs.query)
# SELECT "demo_blog"."id", "demo_blog"."title" FROM "demo_blog"
print(qs.query.alias_map)
# {'demo_blog': <django.db.models.sql.datastructures.BaseTable>}
print(qs.query.alias_refcount)
# {'demo_blog': 0}
print(qs.query.table_map)
# {'demo_blog': ['demo_blog']}
print(qs.query.base_table)
# demo_blog
```
* `alias_map` -> points to the data structure `BaseTable` to reference the actual table
* `alias_refcount` -> gets increased if we add another joint tables
* `base_table` -> derived from the models meta class

<br>

#### What triggers the execution of the query?
* as long as you don't call any of the methods `QuerySet` does nothing with the database
* most of the time the methods that trigger the execution as special Python methods (*dunder*)
  * example of these methods on the `QuerySet`:
    * `__iter__` (e.g. `for blog in qs`)
    * `__len__` (returns count of instances)
    * `__getitem__` (e.g. `qs[:10]) - implements `LIMIT`
    * couple of others like `__bool__`, `__nonzero__`  

<br>

#### How does `QuerySet.filter()` work? 
```python
# query.py

class QuerySet:
    ...
    def _filter_or_exclude_inplace(self, negate, args, kwargs):
        if negate:
            self._query.add_q(Q(*args, **kwargs))
        else:
            self._query.add_q(Q(*args, **kwargs))
    ...
```
* everything that's put in the filter (and exclude) ends up as a regular Q object
* out of the Q objects the Django ORM builds something called the WhereNode 
* example of the WhereNode tree:
  ```python
  blogs = Blog.objects.filter(title__istartswith="A")
  print(blogs.query.where)
  
  # (AND: IStartsWith(Col(demo_blog, demo.Blog.title), 'A'))
  ```
* example of a bit more complex filter (everything that has `id` 1 or starts with `A`):
   ```python
   blogs = Blog.objects.filter(Q(Q(id=1) | Q(title__istartswith="A")))
   print(blogs.query.where)
   
   # (AND:
   #     (OR: Exact(Col(demo_blog, demo.Blog.id), 1),
   #       IStartsWith(Col(demo_blog, demo.Blog.title), 'A')))
   ```
   * two combined filters -> one exact and one IStartsWith
   * `AND` -> a preparation for the next filter to be added
   * how does `AND` work?
   ```python
     def _add_q(
        self,
        ...
        branch_negated=False,
        current_negated=False,
        ...
      ):
      """Add a Q-object to the current filter."""
      connector = q_object.connector
      current_negated = current_negated ^ q_object.negated
      branch_negated = branch_negated or q_object.negated
      ...
      for child in q_object.children:
        ...
      
   ``` 

    * `connector` -> the `AND` connector to the next query
    * `current_negated`, `branch_negated` -> figures out if I have a branch in my current
       tree which is negated and which is not negated 

<br>

#### What about views?
* we have a model
* but we also have a view, which might be accessed faster
* how to query that view but still get our model?

<br>

##### Monkey patch the query object
```python
from django.db.models.sql.datastructures import BaseTable 
bt = BaseTable("demo_secondblog", "demo_secondblog")

blogs.query.alias_map = {'demo_secondblog': bt}
blogs.query.alias_refcount = {'demo_secondblog': 0}
blogs.query.table_map = {'demo_secondblog': ["demp_secondblog"]}
blogs.query.base_table = "demo_secondblog"

print(blogs.query)
# SELECT "demo_secondblog"."id",
# "demo_secondblog"."title" FROM
# "demo_secondblog"
```
* the principle is to overwrite the attributes and change the compiler in that way
  but still end up with a instance of a blog and not of a second blog
* the worst solution

<br>

##### Use Unmanaged Models
```python
class UnmanagedBlog(Blog):
    class Meta:
        managed = False 
        db_table = "demo_secondblog"
```

<br>

### How do values flow back from compiler to the model instance?
Call chain:
* `QuerySet.__iter__`
* `QuerySet.iterator`
* `Query.get_compiler`
* `SQLCompiler.results_iter`
* `SQLCompiler.as_sql`

<br> 

Iter
```python
# query.py

class ModelIterable:
    ...
    def __iter__(self):
        ...
        for row in compiler.results_iter(results):
                obj = model_cls.from_db(
                    db, init_list, row[model_fields_start:model_fields_end]
                )
        ...
    ...
```
* takes the results from the compiler and instantiates the model class
* `init_list, row` -> the database returns the data in an array and it's neccessary
   to have a correspondence between the name of the field and the actual position of the field
   
### The Main Takeaways
* the call chain:
  * Manager
  * QuerySet
  * Query
  * SQLCompiler   
* each of these classed add one leyer of abstactions
* almost every need you might have for custom queries can be met by writting with a custom manager
  * example of soft delete manager, where you don't delete data, but set flag `deleted` and
    don't want this data to end up in the `SELECT` queries:
  ```python
    class BlogManager(models.Manager):
        def get_queryset(self):
            return super().get_queryset().filter(deleted=False)

    class Blog(models.Model):
        objects = BlogManager()

        title = models.CharField(max_length=255)
        deleted = models.BooleanField(default=False)
   ```
