# Chapter 2. Relational Databases with SQLAlchemy

Relational databases are the bedrock upon which almost every modern Web application is built. Learning to think about your application in terms of tables and relationships is one of the keys to a clean, well-designed project. As you will see in this chapter, the data model you choose early on will affect almost every facet of the code that follows. We will be using SQLAlchemy, a powerful object relational mapper that allows us to abstract away the complexities of multiple database engines, to work with the database directly from within Python.  

In this chapter, we shall:  

本章，我们学习以下内容：  

- Present a brief overview of the benefits of using a relational database
- Introduce SQLAlchemy, the Python SQL Toolkit and Object Relational Mapper
- Configure our Flask application to use SQLAlchemy
- Write a model class to represent blog entries
- Learn how to save and retrieve blog entries from the database
- Perform queries – sorting, filtering, and aggregation
- Build a tagging system for blog entries
- Create schema migrations using Alembic

- 使用关系型数据库的现有优点简要概述
- 介绍SQLAlchemy，Python SQL 工具套件以及对象关系映射
- 配置Flask应用以使用SQLALchemy
- 编写一个模型类以表示博客文章
- 学习如何从数据库保存并重新取回文章
- 执行查询——排序，过滤，聚合
- 为博客文章构建一个标签系统
- 使用Alembic创建模型迁移

## Why use a relational database? 为什么使用关系数据库？

Our application's database is much more than a simple record of things that we need to save for future retrieval. If all we needed to do was save and retrieve data, we could easily use flat text files. The fact is, though, that we want to be able to perform interesting queries on our data. What's more, we want to do this efficiently and without reinventing the wheel. While non-relational databases (sometimes known as NoSQL databases) are very popular and have their place in the world of the web, relational databases long ago solved the common problems of filtering, sorting, aggregating, and joining tabular data. Relational databases allow us to define sets of data in a structured way that maintains the consistency of our data. Using relational databases also gives us, the developers, the freedom to focus on the parts of our app that matter.  

我们应用的数据库不仅仅是一条简单的内容记录，我们需要为了将来重新取回执行保存。如果我们要做的事情只有保存和重新取回数据，那么我们可以轻松地使用普通文本文件。而实际情况是，虽然我们想要对数据执行感兴趣的查询。

In addition to efficiently performing ad hoc queries, a relational database server will also do the following:  

此外，为了更有效率地执行

- Ensure that our data conforms to the rules set forth in the schema
- Allow multiple people to access the database concurrently, while at the same time guaranteeing the consistency of the underlying data
- Ensure that data, once saved, is not lost even in the event of an application crash

- 

Relational databases and SQL, the programming language used with relational databases, are topics worthy of an entire book. Because this book is devoted to teaching you how to build apps with Flask, I will show you how to use a tool that has been widely adopted by the Python community for working with databases, namely, SQLAlchemy.  

关系型数据库和SQL，

>#### Note
>SQLAlchemy abstracts away many of the complications of writing SQL queries, but there is no substitute for a deep understanding of SQL and the relational model. For that reason, if you are new to SQL, I would recommend that you check out the colorful book Learn SQL the Hard Way, Zed Shaw available online for free at http://sql.learncodethehardway.org/.

>#### 注释
>SQLAlchemy 抽象出了很多负责的SQL查询，但是对SQL和关系型数据库的深入理解是无法替代的。因此，你是SQL方面的新手，我推荐你

## Introducing SQLAlchemy 介绍SQLAlchemy

SQLAlchemy is an extremely powerful library for working with relational databases in Python. Instead of writing SQL queries by hand, we can use normal Python objects to represent database tables and execute queries. There are a number of benefits to this approach, as follows:  

在Python中SQLAlchemy是一个极其强大的能够和关系型数据库交互的库。与手工编写SQL查询相反，我们可以使用普通Python对象来表示数据库表，以及执行查询。如下，该方法有很多的优点：  

- Your application can be developed entirely in Python.
- Subtle differences between database engines are abstracted away. This allows you to do things just like a lightweight database, for instance, use SQLite for local development and testing, then switch to the databases designed for high loads (such as PostgreSQL) in production.
- Database errors are less common because there are now two layers between your application and the database server: the Python interpreter itself (this will catch the obvious syntax errors), and SQLAlchemy, which has well-defined APIs and its own layer of error-checking.
- Your database code may become more efficient, thanks to SQLAlchemy's unit-of-work model that helps reduce unnecessary round-trips to the database. SQLAlchemy also has facilities for efficiently pre-fetching related objects known as eager loading.
- **Object Relational Mapping (ORM)** makes your code more maintainable, an aspiration known as **don't repeat yourself**, (**DRY**). Suppose you add a column to a model. With SQLAlchemy it will be available whenever you use that model. If, on the other hand, you had hand-written SQL queries strewn throughout your app, you would need to update each query, one at a time, to ensure that you were including the new column.
SQLAlchemy can help you avoid SQL injection vulnerabilities.
- Excellent library support: As you will see in later chapters, there are a multitude of useful libraries that can work directly with your SQLAlchemy models to provide things such as maintenance interfaces and RESTful APIs.

- 你的应用可以完全使用Python开发。
- 数据库引擎之间的细微差被抽象了出来。它允许你像轻量级数据库那样执行操作，例如，为本地开发和测试使用SQLite，然后切换到生产环境中的拥有高负载设计的数据库（比如PostgreSQL）。
- 数据错误通常会减少，因为在应用和数据库服务器之间存在两个层级：Python解释器自身和有着良好设计的API和自有层次的错误检查的SQLAlchemy。
- 你的数据库代码变得更加高效，要感谢SQLAlchemy的单元协作模型，它检查了对数据库的非必要往返开销。SQLAlchemy还拥有被称作贪婪载入的高效率的关联对象预获取工具。  
- Object Relational Mapping (ORM)是你的代码更具有可维护性，
- 良好的库支持：就像你在稍后章节所见到的，有大量的可以直接作用于SQLAlchemy模型以提供诸如维护接口和RESTful API的有用的库。

I hope you're excited after reading this list. If all the items in this list don't make sense to you right now, don't worry. As you work through this chapter and the subsequent ones, these benefits will become more apparent and meaningful.  

我希望你在阅读过这个列表后能觉得激动。如果现在你对这个列表中的所有项不能够理解，请不要担心。

Now that we have discussed some of the benefits of using SQLAlchemy, let's install it and start coding.  

注意我们已经讨论了一些使用SQLAlchemy的好处，我们开安装它，然后开始编写代码。  

>#### Note
>If you'd like to learn more about SQLAlchemy, there is a chapter devoted entirely to its design in The Architecture of Open-Source Applications, available online for free at http://aosabook.org/en/sqlalchemy.html.

>#### 注释
>如果你想要学习关于SQLAlchemy的更多内容，

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

你可以通过打开一个Python解释器来检查你的安装受否成功，然后见检查SQLAlchemy的版本；注意你的确切版本号看上是不同的，  

```shell
$ python
>>> import sqlalchemy
>>> sqlalchemy.__version__
'0.9.0b2'
```

### Using SQLAlchemy in our Flask app 在我们Flask应用中使用SQLAlchemy

SQLAlchemy works very well with Flask on its own, but the author of Flask has released a special Flask extension named Flask-SQLAlchemy that provides helpers with many common tasks, and can save us from having to re-invent the wheel later on. Let's use pip to install this extension:  

SQLAlchemy自己能够和Flask非常好结合，但是Flask的作者发行了一个特殊的称作Flask-SQLAlchemy的特殊Flask扩展，它提供了很多常见任务的辅助，而且免去了我们之后必须重复造轮子的问题。我们使用pip安装这个扩展：  

```shell
(blog) $ pip install flask-sqlalchemy
…
Successfully installed flask-sqlalchemy
```

Flask provides a standard interface for the developers who are interested in building extensions. As the framework has grown in popularity, the number of high-quality extensions has increased. If you'd like to take a look at some of the more popular extensions, there is a curated list available on the Flask project website at http://flask.pocoo.org/extensions/.  

Flask为有兴趣构建扩展的开发者提供了一个标准接口。随着框架流行度的增长，很多高质量的扩展得以增加。如果你希望看一看一些更为流行的扩展，在Flask项目网站上有一个准确的列表 http://flask.pocoo.org/extensions/ 。  

### Choosing a database engine 选择数据引擎

SQLAlchemy supports a multitude of popular database dialects, including SQLite, MySQL, and PostgreSQL. Depending on the database you would like to use, you may need to install an additional Python package containing a database driver. Listed next are several popular databases supported by SQLAlchemy and the corresponding pip-installable driver. Some databases have multiple driver options, so I have listed the most popular one first.  

SQLAlchemy支持许多的流行数据库分支，包括SQLite, MySQL, 和 PostgreSQL。根据你所选择使用的数据库不同，你或许需要安装额外的包含了数据库驱动的Python包。接下来列出了SQLAlchemy支持的多个流行的数据库，以及对应的可以使用pip安装的驱动。部分数据库有多个驱动选择，所以首先列出了对流行的那个。  

table:omit  

SQLite comes as standard with Python and does not require a separate server process, so it is perfect for getting up-and-running quickly. For simplicity in the examples that follow, I will demonstrate how to configure the blog app for use with SQLite. If you have a different database in mind that you would like to use for the blog project, feel free to use `pip` to install the necessary driver package at this time.  

SQLite是作为Python标准出现的，而且不要求独立的服务器进程，所以在快速即开即用方面非常完美。为了简化下面的例子，我会说明如何使用SQLite配置blog引用。如果你有在思想上希望对blog项目使用的不同数据库，

### Connecting to the database 连接数据

Using your favorite text editor, open the config.py module for our blog project (~/projects/blog/app/config.py). We are going to add a SQLAlchemy-specific setting to instruct Flask-SQLAlchemy how to connect to our database. The new lines are highlighted in the following:  

使用你喜欢的文本编辑器，然后打开博客项目（~/projects/blog/app/config.py）中的config.py模块。我们要添加一个SQLAlchemy专用的设置来教Flask-SQLAlchemy如何对数据库进行连接。如下是要插入的代码：  

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

因为SQLite数据被存储在本地，所以我们唯一需要提供的信心就是数据库文件的路径。换句话说，如果你想要连接本地正在运行的PostgreSQL，你的URI看起来是这个样子：  

```
postgresql://postgres:secretpassword@localhost:5432/blog_db
```

>#### Note
>If you're having trouble connecting to your database, try consulting the SQLAlchemy documentation on database URIs: http://docs.sqlalchemy.org/en/rel_0_9/core/engines.html.

>#### 注释
>如果有连接数据的问题，那么请尝试查阅SQLAlchemy文档中的数据库部分：http://docs.sqlalchemy.org/en/rel_0_9/core/engines.html.

Now that we've specified how to connect to the database, let's create the object responsible for actually managing our database connections. This object is provided by the Flask-SQLAlchemy extension and is conveniently named SQLAlchemy. Open app.py and make the following additions:  

现在，我们已经说明了如何连接到数据库，让我们创建实际上负责管理数据库连接的对象。这个对象由Flask-SQLAlchemy扩展提供，为了方便就叫做SQLAlchemy。打开app.py然后加入以下额外内容：  

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

模型是一张我们希望存储在数据库中数据的表的数据表现。这些模型拥有称作列的属性，它们代表了数据中的项。所以，如果我们创建一个Person模型，我们可以为姓、名，生日，和家庭住址，头发颜色，等等设置列。因为的我们的意图在于创建一个表示博客文章的模型，所以我们要设置东西有title和body。  

>#### Note
>Note that we don't say a People model or Entries model – models are singular even though they commonly represent many different objects.  


>#### 注释
>注意，我们不说People模型或者Entries模型，因为模型是单个的，尽管通常可以用来表示很多不同的对象。  

With SQLAlchemy, creating a model is as easy as defining a class and specifying a number of attributes assigned to that class. Let's start with a very basic model for our blog entries. Create a new file named models.py in the blog project's app/ directory and enter the following code:  

使用SQLAlchemy，创建一个模型就和定义一个类，然后定义一组类属性一样容易。我们从一个非常简单的博文模型开始。在blog项目的app／目录中创建一个名叫models.py的文件，然后输入下面的代码：  

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

这里发生了很多事情，所以让我们从导入开始，然后一路走下去。我们以导入标准库datetime和re模块开始。我们会使用datetime来获取当前的日期和时间，re则执行一字符串操作。结下的导入语句带来了创建在app.py中的db对象。你可以回忆一下，db对象是一个SQLAlchemy类的实例，它是Flask-SQLAlchemy扩展的一部分。db对象对我们需要构建Entry模型的类的访问，

Before the Entry model, we define a helper function slugify, which we will use to give our blog entries some nice URLs (used in Chapter 3, Templates and Views). The slugify function takes a string such as A post about Flask and uses a regular expression to turn a string that is human-readable in to a URL, and so returns a-post-about-flask.  

在Entry模型之前，我们定义了一个辅助函数slugify，我们使用它给我们的博客文章带来一些美观的URL（被用在了第三章，模板和视图）。slugify函数接受一个

Next is the Entry model. Our Entry model is a normal class that extends db.Model. By extending db.Model, our Entry class will inherit a variety of helpers that we'll use to query the database.  

接下来是Entry模型。我们的Entry模型是一个普通扩展了db.Model的类。通过扩展db.Model，我们的Entry类继承了多种可以用于数据库查询的辅助。  

The attributes of the Entry model, are a simple mapping of the names and data that we wish to store in the database and are listed as follows:  

Entry模型的属性被简单地映射为我们希望存储在数据库中的名称和数据，一如下面所示：  

- id: This is the primary key for our database table. This value is set for us automatically by the database when we create a new blog entry, usually an auto-incrementing number for each new entry. While we will not explicitly set this value, a primary key comes in handy when you want to refer one model to another, as you'll see later in the chapter.
- title: The title for a blog entry, stored as a String column with a maximum length of 100.
- slug: The URL-friendly representation of the title, stored as a String column with a maximum length of 100. This column also specifies unique=True, so that no two entries can share the same slug.
- body: The actual content of the post, stored in a Text column. This differs from the String type of the Title and Slug as you can store as much text as you like in this field.
- created_timestamp: The time a blog entry was created, stored in a DateTime column. We instruct SQLAlchemy to automatically populate this column with the current time by default when an entry is first saved.
- modified_timestamp: The time a blog entry was last updated. SQLAlchemy will automatically update this column with the current time whenever we save an entry.

- id: 数据库表的主键。该值在我们创建新博客文章时由数据库自动地设置，通常为每篇新文章设置一个自增长的数字。
- title：博客文章的标题，
- slug：表示标题的用户友好URL
- body：文章的真实内容，存储在一个文本列中。
- created_timestamp：博客文章被创建的时间，顺过DateTime列进行存储。我们告诉SQLAlchemy使用当前时间自动地产生这个列
- modified_timestamp：博客文章更新的最后时间。不论何时保存文章，SQLAlchemy都会自动地使用当前时间更新这个列。

>#### Note
>For short strings such as titles or names of things, the String column is appropriate, but when the text may be especially long it is better to use a Text column, as we did for the entry body.

>#### 注释
>为了简化诸如标题和或者名称这类东西，String列的使用是合适的，当时当文章特别长时，最后好使用Text列，就像我们之前对文章的body所做的那样。  

We've overridden the constructor for the class (`__init__`) so that, when a new model is created, it automatically sets the slug for us based on the title.  

我们重写了类的构造器`__init__`，所以在❤新模型被创建时，它会自动地为我们对标题设置slug。  

The last piece is the `__repr__` method that is used to generate a helpful representation of instances of our `Entry` class. The specific meaning of `__repr__` is not important but allows you to reference the object that the program is working with, when debugging.  

最后部分是`__repr__`方法，它用来生成`Entry`类实例的辅助显示内容。

A final bit of code needs to be added to main.py, the entry-point to our application, to ensure that the models are imported. Add the highlighted changes to main.py as follows:  

最后一点代码需要加入到程序的入口main.py，请确保models的导入。如下，对main.py添加高亮的变更内容：  

```python
from app import app, db
import models
import views

if __name__ == '__main__':
    app.run()
```

### Creating the Entry table 创建Entry表

In order to start working with the Entry model, we first need to create a table for it in our database. Luckily, Flask-SQLAlchemy comes with a nice helper for doing just this. Create a new sub-folder named scripts in the blog project's app directory. Then create a file named create_db.py:  

为了开始使用Entry模型，我们首先需要在数据库中为它创建一个表。幸运的是，Flask-SQLAlchemy带来了一个非常好拥有辅助来完成这项操作。在blog项目的app目录中创建一个名叫scripts的子文件夹。然后创建一个称作create_db.py的文件：  

```shell
(blog) $ cd app/
(blog) $ mkdir scripts
(blog) $ touch scripts/create_db.py
```

Add the following code to the create_db.py module. This function will automatically look at all the code that we have written and create a new table in our database for the Entry model based on our models:  

将下面的代码添加到create_db.py模块。该函数会自动地

```python
import os, sys
sys.path.append(os.getcwd())
from main import db

if __name__ == '__main__':
    db.create_all()
```

Execute the script from inside the app/ directory. Make sure the virtualenv is active. If everything goes successfully, you should see no output.  

从app目录执行这个脚本。确保virtualenv是激活的。如果一切顺利，你是看不到输出的。  

```shell
(blog) $ python create_db.py 
(blog) $
```

>#### Note
>If you encounter errors while creating the database tables, make sure you are in the app directory, with the virtualenv activated, when you run the script. Next, ensure that there are no typos in your SQLALCHEMY_DATABASE_URI setting.

>#### 注释
>当你运行这个脚本时，如果在创建数据库表时遇到了错误，请确保你位于app目录中，并使用激活了virtualenv。解析来，确保在你的SQLALCHEMY_DATABASE_URI设置中没有拼写错误。

### Working with the Entry model 使用Entry模型

Let's experiment with our new Entry model by saving a few blog entries. We will be doing this from the Python interactive shell. At this stage let's install IPython, a sophisticated shell with features such as tab-completion (that the default Python shell lacks).  

我们来通过保存几篇博文来体验下Entry模型。我们从Python的交互式shell中执行操作。本阶段我们来安装IPython，一个复杂精密的拥有诸如tab补全（这也是默认Python shell所缺少的）功能的shell。  

```shell
(blog) $ pip install ipython
```

Now check whether we are in the app directory and let's start the shell and create a couple of entries as follows:  

现在，检查我们时候在app目录中，让我们启动shell，并创建两篇文章，一如下面所示：  

```shell
(blog) $ ipython

In []: from models import *  # First things first, import our Entry model and db object.
In []: db  # What is db?
Out[]: <SQLAlchemy engine='sqlite:////home/charles/projects/blog/app/blog.db'>
```

>#### Note
>If you are familiar with the normal Python shell but not IPython, things may look a little different at first. The main thing to be aware of is that `In[]` refers to the code you type in, and `Out[]` is the output of the commands you put into the shell.

>#### 注释
>如果你熟悉不同的Python shell而不是IPython，那么有些地方起初看来有些不同。主要注意的事情是，`In[]`引用了你输的代码，而`Out[]`则是输入到shell中的命令的输出。

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

我们创建了`Entry`对象

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

为了对一个现有的Entry做出改变，请简单编辑之后提交。我们使用之前返回来的id重新取回Entry，应用变更，然后提交它。SQLAlchemy

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

我们对模型使用一个特殊属性来进行查询：Entry.query。该属性暴露了

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

能够重新取回完整的博客文章集合非常有用，但是如果我们想要过滤列表呢？我们可以重新取回整个集合，然后使用循环在Python中过滤它，不过这样做会非常低效。

```shell
In []: Entry.query.filter(Entry.title == 'First entry').all()
Out[]: [<Entry u'First entry'>]
```

If this seems somewhat magical to you, it's because it really is! SQLAlchemy uses operator overloading to convert expressions such as `<Model>.<column> == <some value>`into an abstracted object called `BinaryExpression`. When you are ready to execute your query, these data-structures are then translated into SQL.  

如果你看上去觉得有些魔幻，这是因为实际情况就是这样！

>#### Note
>A BinaryExpression is simply an object that represents the logical comparison and is produced by over riding the standards methods that are typically called on an object when comparing values in Python.

In order to retrieve a single entry, you have two options: .first() and .one(). Their differences and similarities are summarized in the following table:  

为了重新取回单篇文章，你有两个选择： .first() 和 .one()。它们的区别和相似点汇总在了下面的表格中：  

table:omit  

Let's try the same query as before but, instead of calling .all(), we will call .first() to retrieve a single Entry instance:  

```shell
In []: Entry.query.filter(Entry.title == 'First entry').first()
Out[]: <Entry u'First entry'>
```

Notice how previously .all() returned a list containing the object, whereas .first() returned just the object itself.  

### Special lookups 特殊查询

In the previous example we tested for equality, but there are many other types of lookups possible. In the following table, we have listed some that you may find useful. A complete list can be found in the SQLAlchemy documentation.  

在之前的例子中，我们测试了相等行，但还有很多其他类型可能查询类型。下表中，我们列出了一些你对你有用的查询类型。完整的列表可以在SQLAlchemy的文档中找到。  

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

对Python解释器来说不是全部的运算都认为是相等的。这就像在数学课里，我们学习了这样的表达式`2 + 3 * 4`等于14而不是20，因为乘运算首先出现。在Python中比特位运算

table:omit  

If you find yourself struggling with operator precedence, it's a safe bet to put parentheses around any comparison that uses ==, !=, <, <=, >, and >=.  

如果你发现自己正在挣扎于运算符处理过程，那么

## Building a tagging system 构建标签系统

Tags are a lightweight taxonomy system that is perfect for blogs. Tags allow you to apply multiple categories to a blog post and allow multiple posts to be related to one another outside their category. On my own blog I use tags to organize the posts, so that people interested in reading my posts about Flask need only look under the "Flask" tag and find all the relevant posts. As per the spec that we discussed in Chapter 1, Creating Your First Flask Application, each blog entry can have as few or as many tags as you want, so a post about Flask might be tagged with both Flask and Python. Similarly, each tag (for example, Python) can have multiple entries associated with it. In database parlance, this is called a many-to-many relationship.  

标签是一个轻量级的非常适合博客的分类系统。标签允许你对博客文章应用多个分类，还运训多篇文章关联到另外一个外部分类。在我自己的博客上，我使用标签来组织文章，所以

In order to model this, we must first create a model to store tags. This model will store the names of tags we use, so after we've added a few tags the table might look something like the following one:  

为了对此进行建模，我们首先必须创建一个存储标签的模型。整个模型存储了我们要使用的标签名称，所以我们之后添加的几个标签的表格就像下面这个一样：  

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

这一切之前你都见识过了。我们添加一个由数据库来管理的主键，以及一个存储标签名称的单个列。name列被标记为唯一，所以每个标签仅表示这张表中的单个行，且无视有多少博客文章出现。  

Now that we have models for both blog entries and tags, we need a third model to store the relationships between the two. When we wish to signify that a blog entry is tagged with a particular tag, we will store a reference in this table. The following is a diagram of what is happening at the database table level:  

现在，我们同时拥有了博客文章和标签的模型，我们需要第三个模型来存储两者致歉的关系。在我们希望表示一篇博客文章使用特殊标签进行标记时，我们要在这个表中存储一个引用。下面是一个数据库表层面上所发生的事情：  

![img](images/) 

Since we will never be accessing this intermediary table directly (SQLAlchemy will handle it for us transparently), we will not create a model for it but will simply specify a table to store the mapping. Open models.py and add the following highlighted code:  

因为我们根本不要直接的访问这个中间表（明显地SQLAlchemy会为我们处理它的），所以我们不会为此创建一个模型，而是简单地指定一个表来存储映射。打开models.py并加入以下高亮的代码：  

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

通过创建entry_tags表，我们在模型Entry和Tag之间建立了一个链接。SQLAlchemy提供了一个高级的API来处理这个关系，刚好叫做db.relationship的函数。这个函数在Entry模型上创建了一个新属性，它允许我们很容易的读或者写给定的博客文章。这两行代码做了很多事情，所以让我们来看个仔细：  

```python
tags = db.relationship('Tag', secondary=entry_tags,
    backref=db.backref('entries', lazy='dynamic'))
```

We are setting the tags attribute of the `Entry` class equal to the return value of the db.relationship function. The first two arguments, `Tag` and `secondary=entry_tags`, instruct SQLAlchemy that we are going to be querying the Tag model via the `entry_tags` table. The third argument creates a back-reference, allowing us to go from the Tag model back to the associated list of blog entries. By specifying lazy='dynamic', we instruct SQLAlchemy that, instead of it loading all the associated entries for us, we want a Query object instead.  

我们设置`Entry`类的tags属性等于函数db.relationship的返回值。头两个参数，`Tag` 和 `secondary=entry_tags`告诉SQLAlchemy我们要通过`entry_tags`表查询Tag模型。第三个参数创建了一个后向引用，它允许我们从Tag模型回溯到关联的博客文章列表。通过指定lazy='dynamic'，我们告诉SQLAlchemy，与载入所有关联文章相反，我们想要载入一个Query对象。  

### Adding and removing tags from entries 从文章中添加和删除标签

Let's use the IPython shell to see how this works. Close your current shell and re-run the scripts/create_db.py script. This step is necessary since we added two new tables. Now re-open IPython:  

我们使用IPython shell来看看它是如何工作的。关闭当前的shell，然后重新运行scripts/create_db.py脚本。此步骤是必须的，因为我们添加了两个新表。现在重新打开I python：  

```shell
(blog) $ python scripts/create_db.py
(blog) $ ipython
In []: from models import *
In []: Tag.query.all()
Out[]: []
```

There are currently no tags in the database, so let's create a couple of them:  

在数据库中目前还有没有标签，所以我们创建一对标签：  

```shell
In []: python = Tag(name='python')
In []: flask = Tag(name='flask')
In []: db.session.add_all([python, flask])
In []: db.session.commit()
```

Now let's load up some example entries. In my database there are four:  

现在我们载入部分示例文章。在我的数据库中有四个；  

```shell
In []: Entry.query.all()
Out[]:
[<Entry Python entry>,
 <Entry Flask entry>,
 <Entry More flask>,
 <Entry Django entry>]
In []: python_entry, flask_entry, more_flask, django_entry = _
```

>#### Note
>In IPython, you can use an underscore (_) to reference the return-value of the previous line.

>#### 注释
>在IPython中，你可以使用下划线引用前一行的返回值

To add tags to an entry, simply assign them to the entry's tags attribute. It's that easy!  

为了对一个文章添加标签，简单的对这些文章赋予entry的标签属性即可。就是这么简单！

```shell
In []: python_entry.tags = [python]
In []: flask_entry.tags = [python, flask]
In []: db.session.commit()
```

We can work with an entry's list of tags just like a normal Python list, so the usual .append() and .remove() methods will also work:  

我可以像使用普通的Python列表那样使用标签的文章列表，所以常用的.append()和.remove()方法也是可用的：  

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

### Using backrefs 使用后向引用

When we created the tags attribute on the Entry model, you will recall we passed in a backref argument. Let's use IPython to see how the back-reference is used.  

当我们在Entry模型上创建tags属性时，你要重新调用传递到backref中的参数。我们使用IPython来看看后向引用是如何使用的。  

```shell
In []: python  # The python variable is just a tag. 该python变量仅是一个标签而已。
Out[]: <Tag python>
In []: python.entries
Out[]: <sqlalchemy.orm.dynamic.AppenderBaseQuery at 0x332ff90>
In []: python.entries.all()
Out[]: [<Entry Flask entry>, <Entry Python entry>]
```

Unlike the `Entry.tags` reference, the back-reference is specified as lazy='dynamic'. This means that, unlike entry.tags, which gives us a list of tags, we will not receive a list of entries every time we access tag.entries. Why is this? Typically, when the result-set is larger than a few items, it is more useful to treat the backref argument as a query, which can be filtered, ordered, and so on. For example, what if we wanted to show the latest entry tagged with python?  

不同于`Entry.tags`的引用，后向引用是通过lazy='dynamic'指定的。这就意味着，不同于访问entry.tags返回给我们一个标签列表，每次我们访问tag.entries时却不会收到文章列表。为什么会这样子？通常在结果集合大于多个项时，把backref参数当作一次查询更为有用，这个结果集可以被过滤，排序，等等。例如，如果我们想要展示含有python标记的最新文章该怎么做呢？  

```shell
In []: python.entries.order_by(Entry.created_timestamp.desc()).first()
Out[]: <Entry Flask entry>
```

>#### Note
>The SQLAlchemy documentation contains an excellent overview of the various values that you can use for the lazy argument. You can find them online at http://docs.sqlalchemy.org/en/rel_0_9/orm/relationships.html#sqlalchemy.orm.relationship.params.lazy

>#### 注释
>SQLAlchemy文档包含了一个很棒的可用于惰性参数的各种值的概述。你可以在线查找它们http://docs.sqlalchemy.org/en/rel_0_9/orm/relationships.html#sqlalchemy.orm.relationship.params.lazy

## Making changes to the schema 对表应用变更

The final topic we will discuss in this chapter is how to make modifications to an existing Model definition. From the project specification, we know we would like to be able to save drafts of our blog entries. Right now we don't have any way to tell whether an entry is a draft or not, so we will need to add a column that lets us store the status of our entry. Unfortunately, while db.create_all() works perfectly for creating tables, it will not automatically modify an existing table; to do this we need to use migrations.  

本章最后要讨论的话题是如何对一个现有模型定义进行修改。从项目中的详述中，我们得知我们能够保存博客文章的草稿。现在我们没有任何途径来说明一篇文章是否是草稿，送一我们需要添加一个列，以便让我们存储文章的状态。不幸的是，在db.create_all()可以完美地创建表时，它却不会自动地修改现有的表；要解决这个问题我需要运用迁移。  

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

为了使用manager，我们在app.py旁边添加了一个新的称作 manage.py的文件。请添加下面的代码manage.py：  

```python
from app import manager
from main import *

if __name__ == '__main__':
    manager.run()
```

This looks very similar to main.py, the key difference being, instead of calling app.run(), we are calling manager.run().  

看上去和main.py很像，关键的不同在于，与调用app.run()相仿，我们调用的是manager.run()。  

>#### Note 注释
>Django has a similar, although auto-generated, manage.py file that serves a similar function.

>Django拥有一个类似的，虽然是自动生成的一个文件manage.py，它提供了与之类似的函数。  

### Creating the initial migration 创建首次迁移

Before we can start changing our schema, we need to create a record of its current state. To do this, run the following commands from inside your blog's app directory. The first command will create a migrations directory inside the app folder that will track the changes we make to our schema. The second command db migrate will create a snapshot of our current schema so that future changes can be compared to it.  

在开始对表应用变更之前，我们需要创建一个包含当前状态的记录。因此，请在bloga`app`目录内运行下面的命令。第一个命令在`app`文件夹内部创建一个migrations目录，它将跟踪我们的对schema的变更。第二个命令`db migrate`为当前多的schema创建了一个快照，这样未来的变更可以与之相比较。  

```shell
(blog) $ python manage.py db init
Creating directory /home/charles/projects/blog/app/migrations 
... done
...
(blog) $ python manage.py db migrate
INFO  [alembic.migration] Context impl SQLiteImpl.
INFO  [alembic.migration] Will assume non-transactional DDL.
Generating /home/charles/projects/blog/app/migrations/versions/535133f91f00_.py 
... done
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

现在我们拥有了一个当前schema的快照，我们可以开始应用变更了。我们会添加一个新的列，名叫status，它将存储对应到一个特殊状态的蒸熟值。尽管此刻只有两个状态（PUBLIC和DRAFT），对选项使用整数而不是布尔值可以在未来轻松的添加更多状态。打开models.py，将下面的附加内容加入到Entry模型：  

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

在命令行，我们可以再次运行`db migrate`来生成迁移脚本。你可以从命令输入结果中看到新加入的列被找到了！  

```shell
(blog) $ python manage.py db migrate
INFO  [alembic.migration] Context impl SQLiteImpl.
INFO  [alembic.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added column 'entry.status'
  Generating /home/charl
es/projects/blog/app/migrations/versions/2c8e81936cad_.py ... done
```

Because we have blog entries in the database, we need to make a small modification to the auto-generated migration to ensure the statuses for the existing entries are initialized to the proper value. To do this, open up the migration file (mine is migrations/versions/2c8e81936cad_.py) and change the following line:  

因为，在数据库中我们拥有博客文章，所以我们需要对自动生成的迁移做出一点改动以保证现有的文章被初始化到了合适的值。为此，打开迁移文件（我的是migrations/versions/2c8e81936cad_.py）然后更改下面的行：  

```python
op.add_column('entry', sa.Column('status', sa.SmallInteger(), nullable=True))
```

Replacing `nullable=True` with `server_default='0'` tells the migration script to not set the column to null by default, but instead to use `0`.  

使用`server_default='0'`替换`nullable=True`，高数迁移脚本不要把列默认设置为null，而是与之相反设置为0.  

```python
op.add_column('entry', sa.Column('status', sa.SmallInteger(), server_default='0'))
```

Finally, run db upgrade to run the migration and create the status column.  

最后，运行`db upgrade`以执行迁移，创建状态列。  

```shell
(blog) $ python manage.py db upgrade
INFO  [alembic.migration] Context impl SQLiteImpl.
INFO  [alembic.migration] Will assume non-transactional DDL.
INFO  [alembic.migration] Running upgrade 535133f91f00 -> 2c8e81936cad, empty message
```

Congratulations, your `Entry` model now has a status field!  

恭喜啊，你的`Entry`模型现在拥有了status字段！

## Summary

By now you should be familiar with using SQLAlchemy to work with a relational database. We covered the benefits of using a relational database and an ORM, configured a Flask application to connect to a relational database, and created SQLAlchemy models. All this allowed us to create relationships between our data and perform queries. To top it off, we also used a migration tool to handle future database schema changes.  

现在你应该熟悉了使用SQLAlchemy操作关系型数据库。我们学习了使用关系型数据库和ORM的好处，配置一个Flask应用链接关系型数据库，还创建了SQLAlchemy模型。这些操作允许我们在数据和查询之间建立关系。为了解决限制 ，我们还使用迁移工具处理了未来数据库schema的更改。  

In Chapter 3, Templates and Views we will set aside the interactive interpreter and start creating views to display blog entries in the web browser. We will put all our SQLAlchemy knowledge to work by creating interesting lists of blog entries, as well as a simple search feature. We will build a set of templates to make the blogging site visually appealing, and learn how to use the Jinja2 templating language to eliminate repetitive HTML coding. It will be a fun chapter!  
在第三章，模板和视图我们会设置交互解释器的旁白，并在web浏览器中为了现实博客文章开始创建视图。通过创建有趣的博客文章列表，还有简单的搜索功能，我们会把所有的SQLAlchemy派上用场。为了让博客网站的视觉吸引我们会构建一组模板，学习如何使用Jinja2模板语言消除重复的HTML编码工作。这是会是有趣的一章！
