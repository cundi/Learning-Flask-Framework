# Chapter 2. Relational Databases with SQLAlchemy

Relational databases are the bedrock upon which almost every modern Web application is built. Learning to think about your application in terms of tables and relationships is one of the keys to a clean, well-designed project. As you will see in this chapter, the data model you choose early on will affect almost every facet of the code that follows. We will be using SQLAlchemy, a powerful object relational mapper that allows us to abstract away the complexities of multiple database engines, to work with the database directly from within Python.  

In this chapter, we shall:  

本章，我们将：  

- Present a brief overview of the benefits of using a relational database
- Introduce SQLAlchemy, the Python SQL Toolkit and Object Relational Mapper
- Configure our Flask application to use SQLAlchemy
- Write a model class to represent blog entries
- Learn how to save and retrieve blog entries from the database
- Perform queries – sorting, filtering, and aggregation
- Build a tagging system for blog entries
- Create schema migrations using Alembic

- 目前使用关系型数据库的优点简短浏览
- 介绍SQLAlchemy，Python SQL 工具套件以及对象关系映射
- 配置Flask应用以使用SQLALchemy
- 编写一个模型类以表示博客文章
- 学习如何从数据库保存并重新取回文章
- 执行查询——排序，过滤，聚合
- 为博客文章构建一个标签系统
- 使用Alembic创建模型迁移

## Why use a relational database? 为什么使用关系数据库？

Our application's database is much more than a simple record of things that we need to save for future retrieval. If all we needed to do was save and retrieve data, we could easily use flat text files. The fact is, though, that we want to be able to perform interesting queries on our data. What's more, we want to do this efficiently and without reinventing the wheel. While non-relational databases (sometimes known as NoSQL databases) are very popular and have their place in the world of the web, relational databases long ago solved the common problems of filtering, sorting, aggregating, and joining tabular data. Relational databases allow us to define sets of data in a structured way that maintains the consistency of our data. Using relational databases also gives us, the developers, the freedom to focus on the parts of our app that matter.  

In addition to efficiently performing ad hoc queries, a relational database server will also do the following:  

- Ensure that our data conforms to the rules set forth in the schema
- Allow multiple people to access the database concurrently, while at the same time guaranteeing the consistency of the underlying data
- Ensure that data, once saved, is not lost even in the event of an application crash

Relational databases and SQL, the programming language used with relational databases, are topics worthy of an entire book. Because this book is devoted to teaching you how to build apps with Flask, I will show you how to use a tool that has been widely adopted by the Python community for working with databases, namely, SQLAlchemy.  

>#### Note
>SQLAlchemy abstracts away many of the complications of writing SQL queries, but there is no substitute for a deep understanding of SQL and the relational model. For that reason, if you are new to SQL, I would recommend that you check out the colorful book Learn SQL the Hard Way, Zed Shaw available online for free at http://sql.learncodethehardway.org/.

## Introducing SQLAlchemy 介绍SQLAlchemy

SQLAlchemy is an extremely powerful library for working with relational databases in Python. Instead of writing SQL queries by hand, we can use normal Python objects to represent database tables and execute queries. There are a number of benefits to this approach, as follows:  

SQLAlchemy是一个极其强大的库

- Your application can be developed entirely in Python.
- Subtle differences between database engines are abstracted away. This allows you to do things just like a lightweight database, for instance, use SQLite for local development and testing, then switch to the databases designed for high loads (such as PostgreSQL) in production.
- Database errors are less common because there are now two layers between your application and the database server: the Python interpreter itself (this will catch the obvious syntax errors), and SQLAlchemy, which has well-defined APIs and its own layer of error-checking.
- Your database code may become more efficient, thanks to SQLAlchemy's unit-of-work model that helps reduce unnecessary round-trips to the database. SQLAlchemy also has facilities for efficiently pre-fetching related objects known as eager loading.
- **Object Relational Mapping (ORM)** makes your code more maintainable, an aspiration known as **don't repeat yourself**, (**DRY**). Suppose you add a column to a model. With SQLAlchemy it will be available whenever you use that model. If, on the other hand, you had hand-written SQL queries strewn throughout your app, you would need to update each query, one at a time, to ensure that you were including the new column.
SQLAlchemy can help you avoid SQL injection vulnerabilities.
- Excellent library support: As you will see in later chapters, there are a multitude of useful libraries that can work directly with your SQLAlchemy models to provide things such as maintenance interfaces and RESTful APIs.

I hope you're excited after reading this list. If all the items in this list don't make sense to you right now, don't worry. As you work through this chapter and the subsequent ones, these benefits will become more apparent and meaningful.  

Now that we have discussed some of the benefits of using SQLAlchemy, let's install it and start coding.  

>#### Note
>If you'd like to learn more about SQLAlchemy, there is a chapter devoted entirely to its design in The Architecture of Open-Source Applications, available online for free at http://aosabook.org/en/sqlalchemy.html.

### Installing SQLAlchemy 安装SQLAlchemy

We will use pip to install SQLAlchemy into the blog app's virtualenv. As you will recall from the previous chapter, to activate your virtualenv, change directories to source the activate script as follows:  

我们使用pip来安装SQLAlchemy到blog应用到虚拟环境中。

```shell
$ cd ~/projects/blog
$ source blog/bin/activate
(blog) $ pip install sqlalchemy
Downloading/unpacking sqlalchemy
…
Successfully installed sqlalchemy
Cleaning up...
```

You can check if your installation succeeded by opening a Python interpreter and checking the SQLAlchemy version; note that your exact version number is likely to differ.  

```shell
$ python
>>> import sqlalchemy
>>> sqlalchemy.__version__
'0.9.0b2'
```

### Using SQLAlchemy in our Flask app 在我们Flask应用中使用SQLAlchemy

SQLAlchemy works very well with Flask on its own, but the author of Flask has released a special Flask extension named Flask-SQLAlchemy that provides helpers with many common tasks, and can save us from having to re-invent the wheel later on. Let's use pip to install this extension:  

```shell
(blog) $ pip install flask-sqlalchemy
…
Successfully installed flask-sqlalchemy
```

Flask provides a standard interface for the developers who are interested in building extensions. As the framework has grown in popularity, the number of high-quality extensions has increased. If you'd like to take a look at some of the more popular extensions, there is a curated list available on the Flask project website at http://flask.pocoo.org/extensions/.  

### Choosing a database engine 选择数据引擎

SQLAlchemy supports a multitude of popular database dialects, including SQLite, MySQL, and PostgreSQL. Depending on the database you would like to use, you may need to install an additional Python package containing a database driver. Listed next are several popular databases supported by SQLAlchemy and the corresponding pip-installable driver. Some databases have multiple driver options, so I have listed the most popular one first.  

table:omit  

SQLite comes as standard with Python and does not require a separate server process, so it is perfect for getting up-and-running quickly. For simplicity in the examples that follow, I will demonstrate how to configure the blog app for use with SQLite. If you have a different database in mind that you would like to use for the blog project, feel free to use `pip` to install the necessary driver package at this time.  

### Connecting to the database 连接数据

Using your favorite text editor, open the config.py module for our blog project (~/projects/blog/app/config.py). We are going to add a SQLAlchemy-specific setting to instruct Flask-SQLAlchemy how to connect to our database. The new lines are highlighted in the following:  

使用你喜欢的文本编辑器，然后打开博客项目（~/projects/blog/app/config.py）中的config.py模块。我们要添加一个SQLAlchemy专用的设置来吩咐Flask-SQLAlchemy如何对数据库进行连接。如下是要插入的代码：  

```python
import os


class Configuration(object):
    APPLICATION_DIR = os.path.dirname(os.path.realpath(__file__))
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///%s/blog.db' % APPLICATION_DIR
```

The SQLALCHEMY_DATABASE_URI comprises the following parts:  

SQLALCHEMY_DATABASE_URI由下列部分组成：  

```
dialect+driver://username:password@host:port/database
```

Because SQLite databases are stored in local files, the only information we need to provide is the path to the database file. On the other hand, if you wanted to connect to PostgreSQL running locally, your URI might look something like this:  

因为SQLite数据被存储在本地，所以我们唯一需要提供的信心就是数据库文件的路径。换句话说，如果你想要连接本地正在运行的PostgreSQL，

```
postgresql://postgres:secretpassword@localhost:5432/blog_db
```


>#### Note
>If you're having trouble connecting to your database, try consulting the SQLAlchemy documentation on database URIs: http://docs.sqlalchemy.org/en/rel_0_9/core/engines.html.

>#### 注释
>如果有连接数据的问题，那么请尝试查阅SQLAlchemy文档中的数据库部分：http://docs.sqlalchemy.org/en/rel_0_9/core/engines.html.

Now that we've specified how to connect to the database, let's create the object responsible for actually managing our database connections. This object is provided by the Flask-SQLAlchemy extension and is conveniently named SQLAlchemy. Open app.py and make the following additions:  

现在，我们已经说明了如何连接到数据库，让我们创建

```python
from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy

from config import Configuration

app = Flask(__name__)
app.config.from_object(Configuration)
db = SQLAlchemy(app)
```

These changes instruct our Flask app, and in turn SQLAlchemy, how to communicate with our application's database. The next step will be to create a table for storing blog entries and, to do so, we will create our first model.  

这些变更构建了Flask应用，以及如何与数据库通信的SQLAlchemy。接下来要创建的一个排列博文目录的表，所以，我们创建我们的第一个模型。  

## Creating the Entry model 创建模型Entry

A model is the data representation of a table of data that we want to store in the database. These models have attributes called columns that represent the data items in the data. So, if we were creating a Person model, we might have columns for storing the first and last name, date of birth, home address, hair color, and so on. Since we are interested in creating a model to represent blog entries, we will have columns for things like the title and body content.  

模型是一张我们希望存储在数据库中数据的表的数据表现。这些模型拥有称作列的属性，它们代表了数据中的项。所以，如果我们创建一个Person模型，我们可以

>#### Note
>Note that we don't say a People model or Entries model – models are singular even though they commonly represent many different objects. 注释


>#### 注释
>

With SQLAlchemy, creating a model is as easy as defining a class and specifying a number of attributes assigned to that class. Let's start with a very basic model for our blog entries. Create a new file named models.py in the blog project's app/ directory and enter the following code:  

```python
import datetime, re
from app import db

def slugify(s):
    return re.sub('[^\w]+', '-', s).lower()

class Entry(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
    slug = db.Column(db.String(100), unique=True)
    body = db.Column(db.Text)
    created_timestamp = db.Column(db.DateTime, default=datetime.datetime.now)
    modified_timestamp = db.Column(
        db.DateTime,
        default=datetime.datetime.now, 
        onupdate=datetime.datetime.now)

    def __init__(self, *args, **kwargs):
        super(Entry, self).__init__(*args, **kwargs)  # Call parent constructor.
        self.generate_slug()

    def generate_slug(self):
        self.slug = ''
        if self.title:
            self.slug = slugify(self.title)

    def __repr__(self):
        return '<Entry: %s>' % self.title
```

There is a lot going on, so let's start with the imports and work our way down. We begin by importing the standard library datetime and re modules. We will be using datetime to get the current date and time, and re to do some string manipulation. The next import statement brings in the db object that we created in app.py. As you recall, the db object is an instance of the SQLAlchemy class, which is a part of the Flask-SQLAlchemy extension. The db object provides access to the classes that we need to construct our Entry model, which is just a few lines ahead.  

Before the Entry model, we define a helper function slugify, which we will use to give our blog entries some nice URLs (used in Chapter 3, Templates and Views). The slugify function takes a string such as A post about Flask and uses a regular expression to turn a string that is human-readable in to a URL, and so returns a-post-about-flask.  

Next is the Entry model. Our Entry model is a normal class that extends db.Model. By extending db.Model, our Entry class will inherit a variety of helpers that we'll use to query the database.  

The attributes of the Entry model, are a simple mapping of the names and data that we wish to store in the database and are listed as follows:  

- id: This is the primary key for our database table. This value is set for us automatically by the database when we create a new blog entry, usually an auto-incrementing number for each new entry. While we will not explicitly set this value, a primary key comes in handy when you want to refer one model to another, as you'll see later in the chapter.
- title: The title for a blog entry, stored as a String column with a maximum length of 100.
- slug: The URL-friendly representation of the title, stored as a String column with a maximum length of 100. This column also specifies unique=True, so that no two entries can share the same slug.
- body: The actual content of the post, stored in a Text column. This differs from the String type of the Title and Slug as you can store as much text as you like in this field.
- created_timestamp: The time a blog entry was created, stored in a DateTime column. We instruct SQLAlchemy to automatically populate this column with the current time by default when an entry is first saved.
- modified_timestamp: The time a blog entry was last updated. SQLAlchemy will automatically update this column with the current time whenever we save an entry.

>#### Note
>For short strings such as titles or names of things, the String column is appropriate, but when the text may be especially long it is better to use a Text column, as we did for the entry body.

We've overridden the constructor for the class (`__init__`) so that, when a new model is created, it automatically sets the slug for us based on the title.  

The last piece is the `__repr__` method that is used to generate a helpful representation of instances of our `Entry` class. The specific meaning of `__repr__` is not important but allows you to reference the object that the program is working with, when debugging.  

A final bit of code needs to be added to main.py, the entry-point to our application, to ensure that the models are imported. Add the highlighted changes to main.py as follows:  

```python
from app import app, db
import models
import views

if __name__ == '__main__':
    app.run()
```

### Creating the Entry table

In order to start working with the Entry model, we first need to create a table for it in our database. Luckily, Flask-SQLAlchemy comes with a nice helper for doing just this. Create a new sub-folder named scripts in the blog project's app directory. Then create a file named create_db.py:  

```shell
(blog) $ cd app/
(blog) $ mkdir scripts
(blog) $ touch scripts/create_db.py
```

Add the following code to the create_db.py module. This function will automatically look at all the code that we have written and create a new table in our database for the Entry model based on our models:  

```python
import os, sys
sys.path.append(os.getcwd())
from main import db

if __name__ == '__main__':
    db.create_all()
```

Execute the script from inside the app/ directory. Make sure the virtualenv is active. If everything goes successfully, you should see no output.  

```shell
(blog) $ python create_db.py 
(blog) $
```

>#### Note
>If you encounter errors while creating the database tables, make sure you are in the app directory, with the virtualenv activated, when you run the script. Next, ensure that there are no typos in your SQLALCHEMY_DATABASE_URI setting.

### Working with the Entry model 使用Entry模型

Let's experiment with our new Entry model by saving a few blog entries. We will be doing this from the Python interactive shell. At this stage let's install IPython, a sophisticated shell with features such as tab-completion (that the default Python shell lacks).  

我们来通过保存几篇博文来体验下Entry模型。我们从Python的交互式shell中执行操作。本阶段我们来安装IPyton，一个

```shell
(blog) $ pip install ipython
```

Now check whether we are in the app directory and let's start the shell and create a couple of entries as follows:  

```shell
(blog) $ ipython

In []: from models import *  # First things first, import our Entry model and db object.
In []: db  # What is db?
Out[]: <SQLAlchemy engine='sqlite:////home/charles/projects/blog/app/blog.db'>
```

>###Note
>If you are familiar with the normal Python shell but not IPython, things may look a little different at first. The main thing to be aware of is that `In[]` refers to the code you type in, and `Out[]` is the output of the commands you put into the shell.

IPython has a neat feature that allows you to print detailed information about an object. This is done by typing in the object's name followed by a question-mark (?). Introspecting the Entry model provides a bit of information, including the argument signature and the string representing that object (known as the docstring) of the constructor.  

```shell
In []: Entry?  # What is Entry and how do we create it?
Type:       _BoundDeclarativeMeta
String Form:<class 'models.Entry'>
File:       /home/charles/projects/blog/app/models.py
Docstring:  <no docstring>
Constructor information:
 Definition:Entry(self, *args, **kwargs)
```

We can create `Entry` objects by passing column values in as the keyword-arguments. In the preceding example, it uses `**kwargs`; this is a shortcut for taking a `dict` object and using it as the values for defining the object, as shown next:  

```shell
In []: first_entry = Entry(title='First entry', body='This is the body of my first entry.')
```

In order to save our first entry, we will to add it to the database session. The session is simply an object that represents our actions on the database. Even after adding it to the session, it will not be saved to the database yet. In order to save the entry to the database, we need to commit our session:  

为了保存我们的第一篇文章，我们要添加它到数据库会话中。这个

```shell
In []: db.session.add(first_entry)
In []: first_entry.id is None  # No primary key, the entry has not been saved.
Out[]: True
In []: db.session.commit()
In []: first_entry.id
Out[]: 1
In []: first_entry.created_timestamp
Out[]: datetime.datetime(2014, 1, 25, 9, 49, 53, 1337)
```

As you can see from the preceding code examples, once we commit the session, a unique id will be assigned to our first entry and the created_timestamp will be set to the current time. Congratulations, you've created your first blog entry!  

如你在上面代码中所看的那样，一旦我们提交了会话，

Try adding a few more on your own. You can add multiple entry objects to the same session before committing, so give that a try as well.  

尝试着自己去添加更多内容。你可以在提交之前对相同的会话添加多个entry对象，所以请你多多尝试。  

>#### Note 注释
>At any point while you are experimenting, feel free to delete the blog.db file and re-run the create_db.py script to start over with a fresh database.

### Making changes to an existing entry 对现有文章的变更

In order to make changes to an existing Entry, simply make your edits and then commit. Let's retrieve our Entry using the id that was returned to us earlier, make some changes, and commit it. SQLAlchemy will know that it needs to be updated. Here is how you might make edits to the first entry:  


```shell
In []: first_entry = Entry.query.get(1)
In []: first_entry.body = 'This is the first entry, and I have made some edits.'
In []: db.session.commit()
```

And just like that your changes are saved.  

### Deleting an entry 删除文章

Deleting an entry is just as easy as creating one. Instead of calling db.session.add, we will call db.session.delete and pass in the Entry instance that we wish to remove.  

删除一篇文章和创建文章一样简单。和调用db.session.add相反，我们调用db.session.delete，然后传递我们希望移除的Entry实例。  

```shell
In []: bad_entry = Entry(title='bad entry', body='This is a lousy entry.')
In []: db.session.add(bad_entry)
In []: db.session.commit()  # Save the bad entry to the database.
In []: db.session.delete(bad_entry)
In []: db.session.commit()  # The bad entry is now deleted from the database.
```

## Retrieving blog entries 重新取回博文
While creating, updating, and deleting are fairly straightforward operations, the real fun starts when we look at ways to retrieve our entries. We'll start with the basics, and then work our way up to more interesting queries.  

尽管创建、更新和删除都是相当傻瓜式的操作，但在我们浏览重新取回博文时才能真正感觉到乐趣。我们从基础开始，然后运用更为有趣的查询。  

We will use a special attribute on our model class to make queries: Entry.query. This attribute exposes a variety of APIs for working with the collection of entries in the database.  

Let's simply retrieve a list of all the entries in the Entry table:  

我们来简单的重新去取回Entry表中的所有博文列表：  

```shell
In []: entries = Entry.query.all()
In []: entries  # What are our entries?
Out[]: [<Entry u'First entry'>, <Entry u'Second entry'>, <Entry u'Third entry'>, <Entry u'Fourth entry'>]
```

As you can see, in this example the query returns a list of Entry instances that we created. When no explicit ordering is specified, the entries are returned to us in an arbitrary order chosen by the database. Let's specify that we want the entries returned to us in an alphabetical order by title:  

如你所见，这个例子中查询操作返回了一个我们创建的Entry实例的列表。

```shell
In []: Entry.query.order_by(Entry.title.asc()).all()
Out []:
[<Entry u'First entry'>,
 <Entry u'Fourth entry'>,
 <Entry u'Second entry'>,
 <Entry u'Third entry'>]
```

Shown next is how you would list your entries in reverse-chronological order, based on when they were last updated:  

```shell
In []: oldest_to_newest = Entry.query.order_by(Entry.modified_timestamp.desc()).all()
Out []:
[<Entry: Fourth entry>,
 <Entry: Third entry>,
 <Entry: Second entry>,
 <Entry: First entry>]
```

### Filtering the list of entries 过滤文章列表

It is very useful to be able to retrieve the entire collection of blog entries, but what if we want to filter the list? We could always retrieve the entire collection and then filter it in Python using a loop, but that would be very inefficient. Instead we will rely on the database to do the filtering for us, and simply specify the conditions for which entries should be returned. In the following example, we will specify that we want to filter by entries where the title equals 'First entry'.  

```shell
In []: Entry.query.filter(Entry.title == 'First entry').all()
Out[]: [<Entry u'First entry'>]
```

If this seems somewhat magical to you, it's because it really is! SQLAlchemy uses operator overloading to convert expressions such as `<Model>.<column> == <some value>`into an abstracted object called `BinaryExpression`. When you are ready to execute your query, these data-structures are then translated into SQL.  

>#### Note
>A BinaryExpression is simply an object that represents the logical comparison and is produced by over riding the standards methods that are typically called on an object when comparing values in Python.

In order to retrieve a single entry, you have two options: .first() and .one(). Their differences and similarities are summarized in the following table:  

table:omit  

Let's try the same query as before but, instead of calling .all(), we will call .first() to retrieve a single Entry instance:  

```shell
In []: Entry.query.filter(Entry.title == 'First entry').first()
Out[]: <Entry u'First entry'>
```

Notice how previously .all() returned a list containing the object, whereas .first() returned just the object itself.  

### Special lookups 特殊查询

In the previous example we tested for equality, but there are many other types of lookups possible. In the following table, we have listed some that you may find useful. A complete list can be found in the SQLAlchemy documentation.  

table:omit  

### Combining expressions 联合表达式

The expressions listed in the preceding table can be combined using bitwise operators to produce arbitrarily complex expressions. Let's say we want to retrieve all blog entries that have the word `Python` or `Flask` in the title. To accomplish this, we will create two `contains` expressions, then combine them using Python's bitwise OR operator, which is a pipe `|` character, unlike a lot of other languages that use a double pipe `||` character:  

上面表中列出的表达式可以使用比特位运算符来生成具有任意复杂度的表达式。假如我们想要重新取回所有标题中含有`Python` or `Flask`的博客文章。要完成这个操作，我们创建两个“包含”表达式，然后使用Python的OR比特位运算符来合并它们，这里使用的管道符号`|`，而不像其他很多语言那样，使用一对儿管道符号`||` ：  

```python
Entry.query.filter(Entry.title.contains('Python') | Entry.title.contains('Flask'))
```

Using bitwise operators, we can come up with some pretty complex expressions. Try to figure out what the following example is asking for:  

使用比特位运算符，我们可以拥有更多优雅的复杂表达式。请试着指出下面例子到底在查询什么内容？：  

```python
Entry.query.filter(
    (Entry.title.contains('Python') | Entry.title.contains('Flask')) &
    (Entry.created_timestamp > (datetime.date.today() - datetime.timedelta(days=30)))
)
```

As you probably guessed, this query returns all entries where the title contains either Python or Flask, and that were created within the last 30 days. We are using Python's bitwise OR and AND operators to combine the sub-expressions. For any query you produce, you can view the generated SQL by printing the query as follows:  

你可能猜到了，这个查询返回了所有标题中含有Python或者是含有Flask的全部文章，而且是过去30天之内创建的文章。我们使用Python的比特位运算符OR合AND来联合子表达式。

```
In []: query = Entry.query.filter(
    (Entry.title.contains('Python') | Entry.title.contains('Flask')) &
    (Entry.created_timestamp > (datetime.date.today() - datetime.timedelta(days=30)))
)
In []: print str(query)

SELECT entry.id AS entry_id, ...
FROM entry 
WHERE (
    (entry.title LIKE '%%' || :title_1 || '%%') OR (entry.title LIKE '%%' || :title_2 || '%%') ) AND entry.created_timestamp > :created_timestamp_1
```

#### Negation 否定式
There is one more piece to discuss, which is negation. If we wanted to get a list of all blog entries that did not contain Python or Flask in the title, how would we do that? SQLAlchemy provides two ways to create these types of expressions, using either Python's unary negation operator (~) or by calling db.not_(). This is how you would construct this query with SQLAlchemy:  

这里需啊详细讨论下，什么是否定式。假如我们想要获得

Using unary negation:  

使用一元否定式；  

```shell
In []: Entry.query.filter(~(Entry.title.contains('Python') | Entry.title.contains('Flask')))
```

Using db.not_():  

使用db.not_()：  

```shell
In []: Entry.query.filter(db.not_(Entry.title.contains('Python') | Entry.title.contains('Flask')))
```

#### Operator precedence 优先运算符
Not all operations are considered equal to the Python interpreter. This is like in math class, where we learned that expressions such as `2 + 3 * 4` are equal to 14 and not 20, because the multiplication operation occurs first. In Python, bitwise operators all have a higher precedence than things such as equality tests, so this means that, when you are building your query expression, you have to pay attention to the parentheses. Let's look at some example Python expressions and see the corresponding query:  

table:omit  

If you find yourself struggling with operator precedence, it's a safe bet to put parentheses around any comparison that uses ==, !=, <, <=, >, and >=.  

## Building a tagging system 构建标签系统

Tags are a lightweight taxonomy system that is perfect for blogs. Tags allow you to apply multiple categories to a blog post and allow multiple posts to be related to one another outside their category. On my own blog I use tags to organize the posts, so that people interested in reading my posts about Flask need only look under the "Flask" tag and find all the relevant posts. As per the spec that we discussed in Chapter 1, Creating Your First Flask Application, each blog entry can have as few or as many tags as you want, so a post about Flask might be tagged with both Flask and Python. Similarly, each tag (for example, Python) can have multiple entries associated with it. In database parlance, this is called a many-to-many relationship.  

标签是一个轻量级的非常适合博客的分类系统。标签允许你对博客文章应用多个分类，还运训多篇文章关联到另外一个外部分类。在我自己的博客上，我使用标签来组织文章，所以

In order to model this, we must first create a model to store tags. This model will store the names of tags we use, so after we've added a few tags the table might look something like the following one:  

table:omit  

Let's open `models.py` and add a definition for the Tag model. Add the following class at the end of the file, below the Entry class:  

我们打开models.py，并定义Tag模型。将下面的类添加到Entry类的末尾：  

```python
class Tag(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64))
    slug = db.Column(db.String(64), unique=True)

    def __init__(self, *args, **kwargs):
        super(Tag, self).__init__(*args, **kwargs)
        self.slug = slugify(self.name)

    def __repr__(self):
        return '<Tag %s>' % self.name
```

You've seen all of this before. We've added a primary key, which will be managed by the database, and a single column to store the name of the tag. The name column is marked as unique, so each tag will only be represented by a single row in this table, regardless of how many blog entries it appears on.  

Now that we have models for both blog entries and tags, we need a third model to store the relationships between the two. When we wish to signify that a blog entry is tagged with a particular tag, we will store a reference in this table. The following is a diagram of what is happening at the database table level:  

img:omit  

Since we will never be accessing this intermediary table directly (SQLAlchemy will handle it for us transparently), we will not create a model for it but will simply specify a table to store the mapping. Open models.py and add the following highlighted code:  

```python
import datetime, re

from app import db

def slugify(s):
    return re.sub('[^\w]+', '-', s).lower()

entry_tags = db.Table('entry_tags',
    db.Column('tag_id', db.Integer, db.ForeignKey('tag.id')),
    db.Column('entry_id', db.Integer, db.ForeignKey('entry.id'))
)

class Entry(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
    slug = db.Column(db.String(100), unique=True)
    body = db.Column(db.Text)
    created_timestamp = db.Column(db.DateTime, default=datetime.datetime.now)
    modified_timestamp = db.Column(
        db.DateTime,
        default=datetime.datetime.now,
        onupdate=datetime.datetime.now)

    tags = db.relationship('Tag', secondary=entry_tags,
        backref=db.backref('entries', lazy='dynamic'))

    def __init__(self, *args, **kwargs):
        super(Entry, self).__init__(*args, **kwargs)
        self.generate_slug()

    def generate_slug(self):
        self.slug = ''
        if self.title:
            self.slug = slugify(self.title)

    def __repr__(self):
        return '<Entry %s>' % self.title

class Tag(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64))
    slug = db.Column(db.String(64), unique=True)

    def __init__(self, *args, **kwargs):
        super(Tag, self).__init__(*args, **kwargs)
        self.slug = slugify(self.name)

    def __repr__(self):
        return '<Tag %s>' % self.name
```

By creating the entry_tags table, we have established a link between the Entry and Tag models. SQLAlchemy provides a high-level API for working with this relationship, the aptly-named db.relationship function. This function creates a new property on the Entry model that allows us to easily read and write the tags for a given blog entry. There is a lot going on in these two lines of code so let's take a closer look:  

```python
tags = db.relationship('Tag', secondary=entry_tags,
    backref=db.backref('entries', lazy='dynamic'))
```

We are setting the tags attribute of the `Entry` class equal to the return value of the db.relationship function. The first two arguments, `'Tag'` and `secondary=entry_tags`, instruct SQLAlchemy that we are going to be querying the Tag model via the `entry_tags` table. The third argument creates a back-reference, allowing us to go from the Tag model back to the associated list of blog entries. By specifying lazy='dynamic', we instruct SQLAlchemy that, instead of it loading all the associated entries for us, we want a Query object instead.  

### Adding and removing tags from entries 从文章中添加和删除标签

Let's use the IPython shell to see how this works. Close your current shell and re-run the scripts/create_db.py script. This step is necessary since we added two new tables. Now re-open IPython:  

```shell
(blog) $ python scripts/create_db.py
(blog) $ ipython
In []: from models import *
In []: Tag.query.all()
Out[]: []
```

There are currently no tags in the database, so let's create a couple of them:  

```shell
In []: python = Tag(name='python')
In []: flask = Tag(name='flask')
In []: db.session.add_all([python, flask])
In []: db.session.commit()
```

Now let's load up some example entries. In my database there are four:  

```shell
In []: Entry.query.all()
Out[]:
[<Entry Py
thon entry>,
 <Entry Flask entry>,
 <Entry More flask>,
 <Entry Django entry>]
In []: python_entry, flask_entry, more_flask, django_entry = _
```

>#### Note
>In IPython, you can use an underscore (_) to reference the return-value of the previous line.

To add tags to an entry, simply assign them to the entry's tags attribute. It's that easy!  

```shell
In []: python_entry.tags = [python]
In []: flask_entry.tags = [python, flask]
In []: db.session.commit()
```

We can work with an entry's list of tags just like a normal Python list, so the usual .append() and .remove() methods will also work:  

```shell
In []: kittens = Tag(name='kittens')
In []: python_entry.tags.append(kittens)
In []: db.session.commit()
In []: python_entry.tags
Out[]: [<Tag python>, <Tag kittens>]
In []: python_entry.tags.remove(kittens)
In []: db.session.commit()
In []: python_entry.tags
Out[]: [<Tag python>]
```

### Using backrefs

When we created the tags attribute on the Entry model, you will recall we passed in a backref argument. Let's use IPython to see how the back-reference is used.  

```shell
In []: python  # The python variable is just a tag.
Out[]: <Tag python>
In []: python.entries
Out[]: <sqlalchemy.orm.dynamic.AppenderBaseQuery at 0x332ff90>
In []: python.entries.all()
Out[]: [<Entry Flask entry>, <Entry Python entry>]
```

Unlike the `Entry.tags` reference, the back-reference is specified as lazy='dynamic'. This means that, unlike entry.tags, which gives us a list of tags, we will not receive a list of entries every time we access tag.entries. Why is this? Typically, when the result-set is larger than a few items, it is more useful to treat the backref argument as a query, which can be filtered, ordered, and so on. For example, what if we wanted to show the latest entry tagged with python?  

```shell
In []: python.entries.order_by(Entry.created_timestamp.desc()).first()
Out[]: <Entry Flask entry>
```

>#### Note
>The SQLAlchemy documentation contains an excellent overview of the various values that you can use for the lazy argument. You can find them online at http://docs.sqlalchemy.org/en/rel_0_9/orm/relationships.html#sqlalchemy.orm.relationship.params.lazy

 
## Making changes to the schema 对表应用改变

The final topic we will discuss in this chapter is how to make modifications to an existing Model definition. From the project specification, we know we would like to be able to save drafts of our blog entries. Right now we don't have any way to tell whether an entry is a draft or not, so we will need to add a column that lets us store the status of our entry. Unfortunately, while db.create_all() works perfectly for creating tables, it will not automatically modify an existing table; to do this we need to use migrations.  

本章钟最后要讨论的话题是如何对已有模型定义进行修改。从项目中的详述中，我们得知我们能够保存博客文章的草稿。

### Adding Flask-Migrate to our project 对项目添加Flask-Migrate

We will use Flask-Migrate to help us automatically update our database whenever we change the schema. In the blog virtualenv, install Flask-Migrate using pip:  

我们使用Flask-Migrate来帮助我们在改变表时自动地更新数据。请在虚拟环境blog中，使用pip安装Flask-Migrate：  

```shell
(blog) $ pip install flask-migrate
```

>#### Note 注释
>The author of SQLAlchemy has a project called alembic; Flask-Migrate makes use of this and integrates it with Flask directly, making things easier.
>SQLAlchemy的作者拥有一个称作alembic的项目；Flask-Migrate使用了它并直接和Flask进行了集成，一遍易于使用。  

Next we will add a Migrate helper to our app. We will also create a script manager for our app. The script manager allows us to execute special commands within the context of our app, directly from the command-line. We will be using the script manager to execute the migrate command. Open app.py and make the following additions:  

接下来我们对应用添加一个迁移助手。我们还对应用创建了一个脚本管理器。脚本管理器允许我们在应用的上下文中从命令行执行特殊的命令。我们会使用这个脚本管理器来执行迁移命令。打开app.py然后添加如下内容：  

```python
from flask import Flask
from flask.ext.migrate import Migrate, MigrateCommand
from flask.ext.script import Manager
from flask.ext.sqlalchemy import SQLAlchemy

from config import Configuration

app = Flask(__name__)
app.config.from_object(Configuration)
db = SQLAlchemy(app)
migrate = Migrate(app, db)

manager = Manager(app)
manager.add_command('db', MigrateCommand)
```

In order to use the manager, we will add a new file named manage.py along with app.py. Add the following code to manage.py:  

为了使用管理器，我们在app.py旁边添加了一个新的称作 manage.py的文件。请添加下面的代码manage.py：  

```python
from app import manager
from main import *

if __name__ == '__main__':
    manager.run()
```

This looks very similar to main.py, the key difference being, instead of calling app.run(), we are calling manager.run().  

看上去和main.py很像，

>#### Note 注释
>Django has a similar, although auto-generated, manage.py file that serves a similar function.

### Creating the initial migration 创建首次迁移

Before we can start changing our schema, we need to create a record of its current state. To do this, run the following commands from inside your blog's app directory. The first command will create a migrations directory inside the app folder that will track the changes we make to our schema. The second command db migrate will create a snapshot of our current schema so that future changes can be compared to it.  

在开始对表应用改变之前，我们需要创建一个包含当前状态的记录。要完成这项操作，请在blog应用目录中运行下面的命令。

```shell
(blog) $ python manage.py db init

  Creating directory /home/charles/projects/blog/app/migrations ... done
  ...
(blog) $ python manage.py db migrate
INFO  [alembic.migration] Context impl SQLiteImpl.
INFO  [alembic.migration] Will assume non-transactional DDL.
  Generating /home/charles/projects/blog/app/migrations/versions/535133f91f00_.py ... done
```

Finally, we will run db upgrade to run the migration that will indicate to the migration system that everything is up-to-date:  

最后，我们运行db upgrade以运行向迁移系统标示更新所有内容的迁移动作：  

```shell
(blog) $ python manage.py db upgrade
INFO  [alembic.migration] Context impl SQLiteImpl.
INFO  [alembic.migration] Will assume non-transactional DDL.
INFO  [alembic.migration] Running upgrade None -> 535133f91f00, empty message
```

### Adding a status column 添加状态列

Now that we have a snapshot of our current schema, we can start making changes. We will be adding a new column, named status, that will store an integer value corresponding to a particular status. Although there are only two statuses at the moment (PUBLIC and DRAFT), using an integer instead of a Boolean gives us the option to easily add more statuses in the future. Open models.py and make the following additions to the Entry model:  

```python
class Entry(db.Model):
    STATUS_PUBLIC = 0
    STATUS_DRAFT = 1

    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
    slug = db.Column(db.String(100), unique=True)
    body = db.Column(db.Text)
    status = db.Column(db.SmallInteger, default=STATUS_PUBLIC)
    created_timestamp = db.Column(db.DateTime, default=datetime.datetime.now)
    ...
```

From the command-line, we will once again be running `db migrate` to generate the migration script. You can see from the command's output that it found our new column!  

```shell
(blog) $ python manage.py db migrate
INFO  [alembic.migration] Context impl SQLiteImpl.
INFO  [alembic.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added column 'entry.status'
  Generating /home/charl
es/projects/blog/app/migrations/versions/2c8e81936cad_.py ... done
```

Because we have blog entries in the database, we need to make a small modification to the auto-generated migration to ensure the statuses for the existing entries are initialized to the proper value. To do this, open up the migration file (mine is migrations/versions/2c8e81936cad_.py) and change the following line:  

引文，在数据库中我们拥有博客文章，我们需要

```python
op.add_column('entry', sa.Column('status', sa.SmallInteger(), nullable=True))
```

Replacing `nullable=True` with `server_default='0'` tells the migration script to not set the column to null by default, but instead to use `0`.  

```python
op.add_column('entry', sa.Column('status', sa.SmallInteger(), server_default='0'))
```

Finally, run db upgrade to run the migration and create the status column.  

最后，

```shell
(blog) $ python manage.py db upgrade
INFO  [alembic.migration] Context impl SQLiteImpl.
INFO  [alembic.migration] Will assume non-transactional DDL.
INFO  [alembic.migration] Running upgrade 535133f91f00 -> 2c8e81936cad, empty message
```

Congratulations, your `Entry` model now has a status field!  

恭喜了，你的`Entry`模型现在拥有了status字段！

 
## Summary

By now you should be familiar with using SQLAlchemy to work with a relational database. We covered the benefits of using a relational database and an ORM, configured a Flask application to connect to a relational database, and created SQLAlchemy models. All this allowed us to create relationships between our data and perform queries. To top it off, we also used a migration tool to handle future database schema changes.  

In Chapter 3, Templates and Views we will set aside the interactive interpreter and start creating views to display blog entries in the web browser. We will put all our SQLAlchemy knowledge to work by creating interesting lists of blog entries, as well as a simple search feature. We will build a set of templates to make the blogging site visually appealing, and learn how to use the Jinja2 templating language to eliminate repetitive HTML coding. It will be a fun chapter!  



