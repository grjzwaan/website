---
title: "Inject Context"
description: "Advanced context injection in Python. Not for the faint of heart."
date: 2020-03-17
draft: false
tags: [Python]
layout: post
---

This is a small post on how to inject services/contexts into functions.

The motivating example is working with databases and sessions in a webservice. Each call you create a database session, do your stuff, and close the session. When everything goes right, commit. On errors it need to rollback. 

Luckily Python has a nice feature called _context managers_ that you can use with a `with` statement:

```python
# POST /cats/{cat_id:int}
def route(request):
    # Get the parameters and information from the form
    description = request.form().get('description', None)
    cat_id = request.path_params['cat_id']

    # Open a session and query the database
    with db.session() as session:
        cat = session.query(model.Cats).filter(model.Cats.id == cat_id).first()
        if cat is None or description is None:
            raise Exception("Need more information")
        cat.description = description
        session.commit()  
        # An ORM like SQLAlchemy automatically flushes the changes of cat to the database
```

This gets tedious when a lot or your routes have this piece of code. Some web frameworks allow 'middleware' and you could inject the session there...but now **every** route opens a session with your database: that seems overkill. (But hopefully you or your library uses a connection pool).

Premature optimization is the root of fun and evil, so it'll be interesting to see how the code looks like. 

Our goal will be to get the following form:

```python
# POST /cats/{cat_id:int}
def route(request, db_session):
    description = request.form().get('description', None)
    cat_id = request.path_params['cat_id']
    cat = db_session.query(model.Cats).filter(model.Cats.id == cat_id).first()
    if description is None or cat is None:
        throw Exception("Need more info!")
    cat.description = description
```

We introduce a `Depends` class that wraps around a context manager and is declared as default parameter. In Python default parameter values are evaluated when the function is defined, so the `Depends` wrapper will later give us the session context manager. To change the function we'll use a **decorator** `automagic` that will inject this new functionality.

```python
# POST /cats/{cat_id:int}
@automagic
def route(request: Request,
          db_session: DBSession = Depends(db.session()),
):
    description = request.form().get('description', None)
    cat_id = request.path_params['cat_id']
    cat = db_session.query(model.Cats).filter(model.Cats.id == cat_id).first()
    if description is None or cat is None:
        throw Exception("Need more info!")
    cat.description = description
```

## Alternative
The alternative is to make a simpler decorator that injects the session in a pre-determined keyword like this:
```python
@inject_db_session
def route(request: Request,
          db_session: DBSession),
):
    description = request.form().get('description', None)
    cat_id = request.path_params['cat_id']
    cat = db_session.query(model.Cats).filter(model.Cats.id == cat_id).first()
    if description is None or cat is None:
        throw Exception("Need more info!")
    cat.description = description
```

The decorator is much easier to write:
```python
from functools import wraps

def inject_db_session(func):

    @wraps(func)
    def wrapped(*args, **kwargs):
        return func(*args, **{**kwargs, db_session})
    return wrapped
```

It has several features of interest though:

* It uses `functools.wraps`. This ensures that the name and the documentation of the wrapped function stay inplace. See [this excellent Stackoverflow answer](https://stackoverflow.com/questions/308999/what-does-functools-wraps-do).
* It combines the named arguments `**kwargs` with the new keyword in such a way that the new `db_session` supersedes the case when the function would be called with the same  keyword. 

## The code
Below you can see the code that I wrote for this and it works with an arbitrary number of `Depends`.

The followup question is of course: 

Can we also get rid of the boilerplate for getting a path parameter and formdata? For a request endpoint most of the information should come from path parameters and the form (i.e. the body of the request). 


Here we have the following assumptions:

* We will use Pydantic to parse the formdata.
* Depends parameters are replaced with their context managers;
* Simple (int, str, ...) arguments are replace by path parameters according to name;
* Pydantic model arguments are used to parse the form data. Errors are handled by the automagic. You can have at most
one of these models.

It'll look like this:
```python
# POST /cats/{cat_id:int}
@automagic
def route(request: Request,
          cat_id: int,
          catUpdateRequest: CatUpdateRequest,
          db_session: DBSession = Depends(db.session()),
):
    cat = db_session.query(model.Cats).filter(model.Cats.id == cat_id).first()
    if cat is None:
        throw HTTPException(404, "Give a valid cat id")
    cat.description = catUpdateRequest.description
```

This is also  the route that [FastAPI](https://fastapi.tiangolo.com/) took to create their routes. However, I don't know about the implementation. Funnily I found this framework because I wanted the above functionality, made a prototype and Googled it later.

Let's first define some mock objects:

```python
from typing import Callable, List
from contextlib import contextmanager, ExitStack
import inspect
import pydantic
from functools import wraps


class DBSession:
    """
    A mock DBSession class that has the most important functions of a session with a database.
    """
    def query(self, needle, haystack):
        return f"Finding {needle} in {haystack}: {needle in haystack}"

    def commit(self):
        pass

    def rollback(self):
        pass

    def close(self):
        pass


class Database:
    """
    A mock Database class that can create a session scope (this is similar to how it would work with SQLAlchemy).
    """

    def __init__(self, url):
        # Do some initialization
        self.url = url

    def connect(self):
        pass

    def disconnect(self):
        pass

    def create_session(self):
        return DBSession()

    @contextmanager
    def session_scope(self):
        session = self.create_session()
        try:
            print(f"Create session for database {self.url}")
            yield session
            session.commit()
        except:
            session.rollback()
            raise
        finally:
            print(f"Close session for database {self.url}")
            session.close()
```

Now we write our `Depends` wrapper:
```python
class Depends:
    """
    Small wrapper to be able to determine which resources
    """
    def __init__(self, c: Callable):
        self.c = c

    def __call__(self):
        return self.c
```

Now the **good stuff**, the `automagic` function:

```python
def automagic(f):
    # Get the signature and extract dependencies and the formdata
    signature = inspect.signature(f)
    dependencies = {n: p.default 
                    for n, p in signature.parameters.items() 
                    if p.default.__class__ == Depends}
    form_name, form_class = next(((n, p.annotation) 
                                  for n, p in signature.parameters.items() 
                                  if issubclass(p.annotation, pydantic.BaseModel)), None)

    # Create the wrapped function, construct the context managers and inject the information
    @wraps(f)
    def wrapped(*args, **kwargs):
        form = {}
        if form_name:
            form = {form_name: form_class(**request['form_data'])}

        with ExitStack() as stack:
            resources = {}
            for name, resource in dependencies.items():
                res = stack.enter_context(resource())
                resources[name] = res
               
            f(*args, **{**kwargs, **request['path_params'], **resources, **form})

    return wrapped
```

Finally using all these goodies:

```python
db1 = Database("localhost:666/beast")
db2 = Database("localhost:42/answers")

class Form(pydantic.BaseModel):
    answer: int
    answers: List[int]

@automagic
def sample_route(
        request,
        neighbour: int,
        form: Form,
        s1=Depends(db1.session_scope()),
        s2=Depends(db2.session_scope())):
    print(f"Query: {s1.query(neighbour, [665, 667])}")
    print(f"Query: {s2.query(form.answer, [42])}")
    print("I know all the answers now!")

request = {
    'path_params': {
        'neighbour': 667
    },
    'form_data': {
        'answer': 43,
        'answers': ['42']
    }
}

sample_route(request)
```

