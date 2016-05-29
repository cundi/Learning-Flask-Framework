# Chapter 1. Creating Your First Flask Application

Flask is fun. This bold declaration is one of the first things you see when you view the official Flask documentation and, over the course of this book, you will come to understand why so many Python developers agree.  

In this chapter we shall:  

- Briefly discuss the features of the Flask framework
- Set up a development environment and install Flask
- Implement a minimal Flask app and analyze how it works
- Experiment with commonly used APIs and the interactive debugger
- Start working on the blog project that will be progressively enhanced over the course of the book

## What is Flask?

Flask is a lightweight Web framework written in Python. Flask started out as an April fool's joke that became a highly popular underdog in the Python web framework world. It is now one of the most widely used Python web frameworks for start-ups, and is becoming commonly accepted as the perfect tool for quick and simple solutions in most businesses. At its core, it provides a set of powerful libraries for handling the most common web development tasks, such as:  

- URL routing that makes it easy to map URLs to your code
- Template rendering with Jinja2, one of the most powerful Python template engines
- Session management and securing cookies
- HTTP request parsing and flexible response handling
- Interactive web-based debugger
- Easy-to-use, flexible application configuration management

This book will teach you how to use these tools through practical, real-world examples. We will also discuss commonly used third-party libraries for things that are not included in Flask, such as database access and form validation. By the end of this book you will be ready to tackle your next big project with Flask.  

### With great freedom comes great responsibility

As the documentation states, Flask is fun, but it can also be challenging, especially when you are building a large application. Unlike other popular Python web frameworks, such as Django, Flask does not enforce ways of structuring your modules or your code. If you have experience with other web frameworks, you may be surprised how writing applications in Flask feels like writing Python as opposed to the framework boilerplate.  

This book will teach you to use Flask to write clean, expressive applications. As you progress through this book, you will not only become a proficient Flask developer but you will also become a stronger Python developer.  

## Setting up a development environment

Flask is written in Python, so before we can start writing Flask apps we must ensure that Python is installed. Most Linux distributions and recent versions of OSX come with Python pre-installed. The examples in this book will require Python 2.6 or 2.7. Instructions for installing Python can be found at http://www.python.org.  

If this is your first time using Python, there are a number of excellent resources available for free on the web. I would recommend Learn Python The Hard Way, by Zed Shaw, available for free online at http://learnpythonthehardway.org. Looking for more? You can find a large list of free Python resources at http://resrc.io/list/10/list-of-free-programming-books/#python.  

You can verify that Python is installed and that you have the correct version by running the Python interactive interpreter from a command prompt:  

```
$ python
Python 2.7.6 (default, Nov 26 2013, 12:52:49)
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

At the prompt (>>>) type exit() and hit Enter to leave the interpreter.  

### Supporting Python 3

This book will include code that is compatible with both Python 2 and Python 3 where possible. Unfortunately, since Python 3 is still relatively new as compared to Python 2, not all third-party packages used in this book are guaranteed to work seamlessly with Python 3. There is a lot of effort being put into making popular open-source libraries compatible with both versions but, at the time of writing, some libraries have still not been ported. For best results, ensure that the version of Python that you have installed on your system is 2.6 or above.  

## Installing Python packages

Now that you have ensured that Python is installed correctly, we will install some popular Python packages that will be used over the course of this book.  

We will be installing these packages system-wide but, once they are installed, we will be working exclusively in virtual environments.  

### Installing pip

The de-facto Python package installer is pip . We will use it throughout the book to install Flask and other third-party libraries.  

If you already have setuptools installed, you can install pip by simply running the following command:  

```shell
$ sudo easy_install pip
```

After completing the installation, verify that pip is installed correctly: 

```shell
$ pip --version
```

pip 1.2.1 from /usr/lib/python2.7/site-packages/pip-1.2.1-py2.7.egg (python 2.7)
The version numbers are likely to change, so for a definitive guide please consult the official instructions, which can be found at http://www.pip-installer.org/en/latest/installing.html.  

### Installing virtualenv

Once pip is installed, we can proceed to install the most important tool in any Python developer's toolkit: virtualenv. Virtualenv makes it easy to produce isolated Python environments, complete with their own copies of system and third-party packages.  

#### Why use virtualenv?
Virtualenv solves a number of problems related to package management. Imagine you have an old application that was built using a very early version of Flask, and you would like to build a new project using the most-recent version of Flask. If Flask was installed system-wide, you was be forced to either upgrade your old project or write your new project against the old Flask. If both projects were using virtualenv, then each could run its own version of Flask, with no conflicts or issues.  

Virtualenv makes it easy to control which versions of the third-party package is used by your project.  

Another consideration is that installing packages system-wide generally requires elevated privileges (sudo pip install foo). By using virtualenvs, you can create Python environments and install packages as a regular user. This is especially useful if you are deploying to a shared hosting environment or are in a situation where you do not have administrator privileges.  

#### Installing virtualenv with pip
We will use pip to install virtualenv; since it is a standard Python package, it can be installed just like any other Python package. To ensure that virtualenv is installed system-wide, run the following command (it requires elevated privileges):  

```shell
$ sudo pip install virtualenv
$ virtualenv --version
1.10.1
```

The version numbers are likely to change, so for a definitive guide please consult the official instructions at http://virtualenv.org.  

## Creating your first Flask app

Now that we have the proper tools installed, we're ready to create our first Flask app. To begin, create a directory somewhere convenient that will hold all of your Python projects. At the command prompt or terminal, navigate to your projects directory; mine is /home/charles/projects, or ~/projects for short on Unix-based systems.  

```shell
$ mkdir ~/projects
$ cd ~/projects
```

Now we will create a virtualenv. The commands below will create a new directory named hello_flask inside your projects folder that contains a complete, isolated Python environment.  

```shell
$ virtualenv hello_flask

New python executable in hello_flask/bin/python2.
Also creating executable in hello_flask/bin/python
Installing setuptools............done.
Installing pip...............done.
$ cd hello_flask
```

If you list the contents of the hello_flask directory, you will see that it has created several sub-directories, including a bin folder (Scripts on Windows) that contains copies of both Python and pip. The next step is to activate your new virtualenv. The instructions differ depending on whether you are using Windows or Mac OS/Linux. To activate your virtualenv refer to the following screenshot:  

img:omitt  

When you activate a virtualenv, your PATH environment variable is temporarily modified to

ensure that any packages you install or use are restricted to your virtualenv.

Installing Flask in your virtualenv

Now that we've verified that our virtualenv is set up correctly, we can install Flask.

When you are inside a virtualenv, you should never install packages with administrator privileges. If you receive a permission error when attempting to install Flask, double-check that you have activated your virtualenv correctly (you should see (hello_flask) in your command prompt).


(hello_flask) $ pip install Flask
You will see some text scroll by as pip downloads the Flask package and the related dependencies before installing it into your virtualenv. Flask depends on a couple of additional third-party libraries, which pip will automatically download and install for you. Let's verify that everything is installed properly:


(hello_flask) $ python
>>> import flask
>>> flask.__version__
'0.10.1'
>>> flask
<module 'flask' from '/home/charles/projects/hello_flask/lib/python2.7/site-packages/flask/__init__.pyc'>
Congratulations! You've installed Flask and now we are ready to start coding.

Hello, Flask!

Create a new file in the hello_flask virtualenv named app.py. Using your favorite text editor or IDE, enter the following code:

from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello, Flask!'

if __name__ == '__main__':
    app.run(debug=True)
Save the file and then execute app.py by running it from the command line. You will need to ensure that you have activated the hello_flask virtualenv:

```shell
$ cd ~/projects/hello_flask
(hello_flask) $ python app.py
* Running on http://127.0.0.1:5000/
```

Open your favorite web-browser and navigate to the URL displayed (http://127.0.0.1:5000). You should see the message Hello, Flask! displayed on a blank white page. By default, the Flask development server runs locally on 127.0.0.1, bound to port 5000.  

img:omit  

### Understanding the code

We just created a very basic Flask app. To understand what's happening let's take this code apart line-by-line.  

```python
from flask import Flask
```

Our app begins by importing the Flask class. This class represents a single WSGI application and is the central object in any Flask project.  

WSGI is the Python standard web server interface, defined in PEP 333. You can think of WSGI as a set of behaviors and methods that, when implemented, allow your web app to just work with a large number of webservers. Flask handles all the implementation details for you, so you can focus on writing you web app.  

```python
app = Flask(__name__)
```

In this line, we create an application instance in the variable app and pass it the name of our module. The variable app can of course be anything, however app is a common convention for most Flask applications. The application instance is the central registry for things such as views, URL routes, template configuration, and much more. We provide the name of the current module so that the application is able to find resources by looking inside the current folder. This will be important later when we want to render templates or serve static files.  

```python
@app.route('/')
def index():
    return 'Hello, Flask!'
```

In the preceding lines, we are instructing our Flask app to route all requests for / (the root URL) to this view function (index). A view is simply a function or a method that returns a response of some kind. Whenever you open a browser and navigate to the root URL of our app, Flask will call this view function and send the return value to the browser.  

There are a few things to note about these lines of code:  

- `@app.route` is a Python decorator from the app variable defined above. This decorator (app.route) wraps the following function, in this case,index, in order to route requests for a particular URL to a particular view. Index is chosen as the name for the function here, as it's the common name for the first page that a web server uses. Other examples could be homepage or main. Decorators are a rich and interesting subject for Python developers, so if you are not familiar with them, I recommend using your favorite search engine to find a good tutorial.  

- The `index` function takes no arguments. This might seem odd if you are coming from other web-frameworks and were expecting a request object or something similar. We will see in the following examples how to access values from the request.  

- The `index` function returns a plain string object. In later examples, we will see how to render templates to return HTML.  

The following lines execute our app using the built-in development server in debug mode. The 'if' statement is a common Python convention that ensures that the app will only be run when we run our script via python app.py, and will not run if we try to import this app from another Python file.  

```python
if __name__ == '__main__':
    app.run(debug=True)
```

### Routes and requests 路由和请求

Right now our Flask app isn't much fun, so let's look at the different ways in which we can add more interesting behavior to our web app. One common way is to add responsive behavior so that our app will look at values in the URL and handle them. Let's add a new route to our Hello Flask app called hello. This new route will display a greeting to the person whose name appears in the URL:   

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    return 'Hello, Flask!'


@app.route('/hello/<name>')
def hello(name):
    return 'Hello, %s' % name
```

