# Use Report of MEAN stack and LAMP stack and OAuth 2.0 Server Implementation on them

## Abstract

MEAN stack is one of the most recent and popular web solution widely used now.
Meanwhile LAMP stack is one of the oldest ones, however this does not prevent people from using it in a modern fashion.
In this report, I am going to compare my working experience with both MEAN stack and LAMP stack as a web developer.
Base on my experience, due to heavy setup and configuration process for LAMP stack, MEAN stack is easier to learn and work with, especially for new developers.

## Introduction

During at least passed 5 years, Node.js as a javascript server-side solution becomes more and more popular for its light weight and easiness of use.
And MEAN stack solution also becomes lots of developers' first choice for building web applications.

On the other hand, LAMP stack is one of the oldest solution used for web applications, which has been used more than 20 years.
However, it is not abandoned because of its age. It has been improving, and now it is used in a modern fashion to build web applications.

MEAN stack includes:
  - AngularJS (front-end framework)
  - Express (HTTP framework)
  - Node.js (Javascript Engine)
  - MongoDB (Database)

LAMP stack incldues:
  - PHP
  - Apache2 (server)
  - MySQL (Database)
  - Linux

In either stack, each letter represents a software. However, these softwares are not one-to-one connected between stacks.
For example, in MEAN stack, A represents AngularJS, which is a front-end framework running in client side.
It does not care what technologies server-side is using. It can work with either Node.js server or a Apache-PHP server.
Meanwhile, in LAMP stack, L represents Linux. In most case, MEAN stack server also running on a Linux.
The stack layers can be redraw as below.

![stack](./images/stack.png)

Furthermore, MongoDB is a non-relational database, while MySQL is a relational data. The difference between non-relational databases and relational database is out the scope of this report.
So we will talk about the difference on their interaction with server framework for two databases.

In the following sections, I will compare MEAN stack and LAMP stack, and address the difference in an order of learning process.
This means as a new developer to either stack, we will learn and compare both stacks on following point:
  - Environment setup
  - Dependency management
  - HTTP Server
  - Serve static files
  - RESTful API
  - Middleware
  - DB interactions
  - Template Engine

### Environment Setup

Before we start, the first thing we need to do is setting up the development environment.

The installations for either MEAN stack or LAMP stack on a Linux would involve bunch of commands, reboots and configurations.
To make life easier, I use docker and two popular docker images.
  - [rmukhia/im-meanjs](https://hub.docker.com/r/rmukhia/im-meanjs/)
  - [linode/lamp](https://hub.docker.com/r/linode/lamp/)

### Dependencies Management

After the development environments have been setup, we can start building our web app.
In order to build a web app, some libraries are often need to be install before we can actually write some code.

MEAN stack use npm to install, delete and manage the dependency libraries, while LAMP stack use Composer.

#### Add libraries

Both npm and composer use a json formatted file to let developer specify the dependencies libraries and their versions.
npm's dependencies specification file is called ```package.json```, while composer's is called ```composer.json```.

To install the libraries, for MEAN stack, run:
```bash
$ npm install
```
while for LAMP stack, run:
```bash
$ composer install
```

When running the above commands, both npm and composer would create a folder to store all the dependencies.
For MEAN stack, the folder is called ```node_modules```. For LAMP stack, it is celled ```vendor```.

Composer also generates a JSON file called ```composer.lock```. This file stores a list of exact versions  it installed.
When ```composer install``` got run, it will check if there is a ```composer.lock``` file first.
If there is, it will use ```composer.lock``` to install the dependencies, instead of using ```composer.json```, since the versions are more accurate.

#### \*Dependencies of Dependencies
Let's take a close look at node_modules directory and vendor directory.
![node_modules_structure](./images/node_modules.png)
![vendor_structure](./images/vendor.png)

As shown above, inside each node_modules library directory, there will be a ```package.json``` file and another ```node_modules``` directory. Node_modules are nested. However, there is only one ```vendor``` folder among the while project.

This means, for npm, each node_module library has its own dependencies, and the dependencies structure is like a tree. For composer, the vendor folder can not be nested, such that the dependencies structure is flat. vendor libraries need to share dependencies if they have.

### Run HTTP Server

Now we setup the environments and installed dependency modules. Finally, we can run a simple HTTP server.

#### Express Server

For MEAN stack, we are building a HTTP server using Express framework.
(Node.js is not a server, instead it is a Javascript runtime.)

```javascript
var express = require('express');
var app = express();

app.listen(3000, function () {
  console.log('Server start listening on port 3000!');
});
```

#### Apache2 Server

For LAMP stack, Apache 2 is used as a HTTP server.
With the use of docker and linode/lamp image, we could save some work on configuring Apache 2 server.
(PHP is also not a server. Apache 2 runs PHP scripts)

Under Linux, Apache 2 runs as a daemon. So it can be started using commnad line:
```
$ service apache2 start
```

### Serve Static Files

So far both servers are running, and we can serve something from those servers.
The very first thing we need to serve is static files, such as css files javascript files and even the whole AngularJS app.

##### Express Server

For MEAN stack, Express framework uses a middleware to serve the static files.

```javascript
app.use(express.static(path.resolve('./public')));
```

##### Apache2 Server

By default, if we go to http://example.com/script.php, Apache2 runs the script stored in ```root/script.php```, and response the result.
If there is no PHP script define inside ```script.php```, Apache2 will just return the content of the file.
This is exactly what we need to serve static files.

### Express.js vs. Slim 3

Now, we can use browser to go to ```path/to/file```, and the server will serve the static file.
Next, we are going to build some server features for both stack, such as RESTful API, middleware and Database interactions.
In order to do this easier, for LAMP stack, it is better to use some PHP micro framework. There are dozens of PHP framework.
Here, in this report, I am going to use one of the newest ones called Slim 3.

#### Routes & Router

At first, to make RESTful API, at least we need to separate different HTTP methods for the same URI.

##### Express:
```javascript
app.get('/users', function (req, res) {
  res.send('GET users');
});

app.post('/users', function (req, res) {
  res.send('POST users');
});

// or you can pack different methods handle into controllers
app.route('/users')
	.get(function (req, res) {
    res.send('GET users');
  })
	.post(function (req, res) {
    res.send('POST users');
  });
```

##### Slim 3:
```php
<?php
$app->get('/users', function ($request, $response, $args) {
    ...
    return $response;
});

$app->post('/users', function ($request, $response, $args) {
    ...
    return $response;
});

// or you can pack different methods handle into controllers
$app->any('/users', 'UserController');
?>
```

With only this snippet of code, the routes would not work as we expect.

As we mentioned above, by default, Apache server will directly serve the file or run the script with the path specified in the URL.
However, to make RESTful API, the URL doesn't have to represents a file or a script store as file locating somewhere in the file system.
More generally, a URL would represents a resource store in the database.
For example, in above case, ```users``` is not a file, but a tables/resource store in the database.

In this case, we need to let Apache server know that when it see URL like ```/users```, don't try to find a file called ```users```, instead run a script called ```index.php``` in which contains above code. (Let's assume above code is stored in a file called ```index.php```)

###### URL Rewriting

In LAMP stack, we use a configuration file called .htaccess to let Apache2 server know our special arrangement for this situation.
.htaccess is used to control the access of the server to the director .htaccess under and its subdirectory.
This is where we are going to define the URL rewriting rules we address above.

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```
- The first line tells the server to enable the URL rewriting under this directory.
- The second and third lines define two conditions under which the following rewriting rules would applied.
- The fourth line defines the rewriting rule.

Overall, this .htaccess file tells the Apache server, if the REQUEST_FILENAME is not found under this directory, run index.php instead.
By this way, the router pattern becomes possible with Apache server.

#### Middleware

There are always some code, for different reasons, we neither want to put it into routes nor want to call it from the routes.
(In most cases, the code is used multiple time or it is irrelevant to the routes business logic.)
Middleware is introduced. Middleware works as extra layers of code server need to run before/after it runs your routes code.

Express:
```javascript
// middleware for /users/:name
app.use('/users/:name', function (req, res, next) {
    // middleware 1
    next();
  }, function (req, res, next) {
    // middleware 2
    next();
  },
  ...
);
// routes for /users/:name
app.get('/users/:name', function (req, res) {
  res.status(200).send();
});
```

The way Express handle middleware is it chains them up.
After finishing each middleware, developer need to explicitly call function ```next``` to run the next middleware.
And after all middleware get called and ```next``` function get called in last middleware, express server start run the routes.

The design of middleware management would look like following graph in Express framework.
![express-middleware](./images/middleware-express.png)

Slim 3:
```php
<?php
$mw = function ($request, $response, $next) use ($app) {
    // ...
    $response = $next($request, $response);
    // ...
    return $response;
};

$app->get('/users/{name}', function ($request, $response, $args) {
    // ...
    return $response;
})->add($mw);
?>
```

The way Slim 3 framework use middleware is different.
It treats middleware as a wrapper. Every time adding a middleware, it wraps the exist middleware and app logic inside itself.
This is why when defining a middleware, we can put our code either before or after ```$response = $next($request, $response);``` clause.

The design of middleware management would look like following graph in Slim 3 framework.
![slim-middleware](./images/middleware-slim.png)


### Database Interactions



#### Mongoose vs. PDO MySQL Driver

MEAN stack uses an MongoDB object modeling framework called Mongoose to do CURD operations. It provide object like interface to access the data, hiding the database command detail from server application developing.

PHP Data Object (PDO) is a interface for accessing data in PHP. PDO itself does not do anything, it is just a interface. PDO drivers do the actual work. Since we use MySQL database, so relatively we need to use a MySQL PDO Driver to do the CURD operations. Comparing with Mongoose, PDO drivers use more original sql command to access the database.

\* A PDO driver for MySQL database called ```pdo_mysql.so``` should be automatically installed when you install PHP. If not, install it using
```
$ sudo apt-get install php5-mysql
```
After it is installed, it still may not been activated for your ```apache2``` service. To enable it, go to your ```php.ini```, add or uncomment following lines:
```
extension=pdo.so
extension=pdo_mysql.so
```

##### Connect Database
Mongoose:
```javascript
var connectString = 'mongodb://mongo:27017/learn-mean-auth-dev';
var db = mongoose.connect(connectString, function(err) {
	if (err) {
		console.error(chalk.red('Could not connect to MongoDB!'));
		console.log(chalk.red(err));
	}
});
```
PDO Driver:
```php
<?php
$dbh = new PDO("mysql:host=$dbhost;dbname=$dbname", $dbuser, $dbpass);
$dbh->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
?>
```
##### Create Schema
Using MEAN stack, with Mongoose, we create a model schema directly in model module with javascript.
```javascript
var mongoose = require('mongoose'),
	  Schema = mongoose.Schema,
    UserSchema = new Schema({
        username: {
    		type: String,
    		unique: 'testing error message',
    		required: 'Please fill in a username',
    		trim: true
    	},
      ...
    });
mongoose.model('User', UserSchema);
```

Using LAMP stack, a table need to be created in database, and this should only been done once. So it is better to do this directly with SQL command.
```sql
CREATE TABLE users (
username VARCHAR(255) NOT NULL,
...
CONSTRAINT username_pk PRIMARY KEY (username))
```
##### Create an object
After the table table is created, we can insert object into the database.

Mongoose:
```javascript
var user = new User(data);
user.save(function(err, user) {
	if (err) {
    // handle error
	}
});
```
PDO Driver:
```php
<?php
try {
    $sql = "INSERT INTO users (username, ...) VALUES (:username, ...)";
    $stmt = $dbh->prepare($sql);
    if ($stmt->execute($data)) {
        $data['id'] = $dbh->lastInsertId();
    }
} catch(PDOException $e) {
    return $e->getMessage();
}
?>
```
##### Query objects
And at last, we also need to get objects out of database.

Mongoose:
```javascript
User.find().sort('-created').exec(function(err, articles) {
  if (err) {
    // handle error
	}
});
```
PDO Driver:
```php
<?php
$r = array();
$sql = "SELECT * FROM users";
$stmt = $dbh->prepare($sql);
if ($stmt->execute()) {
    $r = $stmt->fetchAll(PDO::FETCH_ASSOC);
} else {
    $r = 0;
}
?>
```

### Javascript vs PHP

At first glimpse, JavaScript and PHP could be very similar, especially when using JavaScript with Express and using PHP with Slim 3. However, when we go further, we would find that the same concept in two different languages could be significant different.

#### Variable Scope and Closure
[StackOverflow Thread](http://stackoverflow.com/questions/7417430/javascript-closures-vs-php-closures-whats-the-difference)
```javascript
var a = 0;
var f = function () {
  console.log(a);
}
a = 1;
f();
```
As we all know the ```a = 1;``` clause will change the value of a, then when log a, it shows the new value.

```php
<?php
$a = 0;
$f = function () {
  echo $a;
}
$a = 2;
$f();
?>
```
When running this, PHP will complain that variable is undefined. Inside function $f, the outside scope is hide from inside function body. This could be a good thing. By this way, it is not easy for programmer to mass up variables in function scope with the scope outside. However, if we know what we are doing and we insist to use the variable defined outside inside the function body, PHP provide this ```use``` syntax to pass outside variable to inside.

```php
<? php
$a = 0;
$f = function () use ($a) {
  echo $a
}
$a = 2;
$f();
?>
```
### Template View Engine

#### Setup

##### Swig
```javascript
var swig = require('swig');
app.engine('html', swig.renderFile);
app.set('view engine', 'html');
app.set('views', path.join(__dirname, './templates'));
```

##### Twig
```php
<?php
// Create app
$app = new \Slim\App();

// Get container
$container = $app->getContainer();

// Register component on container
$container['view'] = function ($container) {
    $view = new \Slim\Views\Twig('path/to/templates', [
        'cache' => 'path/to/cache'
    ]);
    $view->addExtension(new \Slim\Views\TwigExtension(
        $container['router'],
        $container['request']->getUri()
    ));
    return $view;
};
?>
```
#### HTML Template

##### Swig
```html
<h1>{{ pagename|title }}</h1>
<ul>
{% for author in authors %}
  <li{% if loop.first %} class="first"{% endif %}>
    {{ author }}
  </li>
{% endfor %}
</ul>
```

##### Twig
```html
<h1>My Webpage</h1>
{{ a_variable }}
<ul id="navigation">
{% for item in navigation %}
    <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
{% endfor %}
</ul>
```

#### Render Tempaltes
```javascript
app.get('/', function (req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!'});
});
```

```php
<?php
$app->get('/demo', function ($request, $response, $args) {
  return $this->view->render($response, 'demo.html', [
    'a_variable' => $foo
  ]);
});
?>
```
