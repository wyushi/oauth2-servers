# Use Report of MEAN stack and LAMP stack

## Abstract

MEAN stack is one of the most recent and popular web solutions.
LAMP stack is one of the oldest and widely used ones. Although it's old, people still use it and it has been improved to have modern features.

In this report, I am going to compare my development experience of MEAN stack and LAMP stack.
Due to heavy setup and configuration processes of LAMP stack, I believe MEAN stack is easier to learn and work with, especially for new developers.

## Introduction

During passed a few years, Node.js becomes more and more popular because of its light weight and easiness of use.
And MEAN stack solution also becomes lots of developers' first choice for building web applications.

On the other hand, LAMP stack has been used for a long time to build web applications in passed decades.
The way it has been used changes through time, and it has been improved to have modern features as well.

This report will introduce both stacks, and go through the work experience of building modern server features on each of them.

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

In either stack, each letter of the four letters represents a software.
However, these softwares are not one-to-one matched as solutions for the same problem between stacks.
For example, in MEAN stack, A represents AngularJS [1], which is a front-end framework.
It does not depending on what technologies server is building on. It can work with either Node.js server or a Apache-PHP server.
Meanwhile, in LAMP stack, L represents Linux [2]. In most case, MEAN stack server also running on a Linux.
The stack layers can be reconstruct as below:

![stack](./images/stack.png)

For the database solutions, MongoDB [3] is a non-relational database, while MySQL [4] is a relational one.
The difference between non-relational databases and relational database is out the scope of this report.
So I will focus on the difference on how server framework interact with them.

In the following sections, I will compare two stacks in an order of building web applications, which including following topics:
  - [Environment setup](#environment-setup)
  - [Dependency management](#dependency-management)
  - [HTTP Server](#run-http-server)
  - [Serve static files](#serve-static-files)
  - [RESTful API](#expressjs-vs-slim-3)
  - [Middleware](#middleware)
  - [DB interactions](#database-interactions)
  - [Template Engine](#template-engine)

### Environment Setup

First of all, I need to setup a development environment for each stack.

The installations for either MEAN stack or LAMP stack would involve bunch of commands, reboots and configurations.
To make it easier, docker is used.
  - [rmukhia/im-meanjs](https://hub.docker.com/r/rmukhia/im-meanjs/) [5]
  - [linode/lamp](https://hub.docker.com/r/linode/lamp/) [6]

### Dependency Management

Secondly, I need to know how dependency is managed on each stack, such that I can install the libraries modules I need to build a application.

MEAN stack uses npm [7] to manage dependencies, while LAMP stack uses Composer [8].

#### Adding libraries

Both npm and Composer use a json file to let developer specify the dependencies and their versions.
For npm, this file is named ```package.json```, while it is named ```composer.json``` for Composer.

To install the libraries, for MEAN stack, run:
```bash
$ npm install
```
while for LAMP stack, run:
```bash
$ composer install
```

When running above commands, both npm and Composer would create a folder to store all the dependencies.
For npm, the folder is named ```node_modules```. For Composer, it is ```vendor```.

After creating ```vender``` directory, Composer also generates a JSON file called ```composer.lock```.
This file stores the dependencies and their exact versions number has been installed.
If this file presents, it will be used to install dependencies instead of ```composer.json```, to make sure dependencies version are exactly the same between different development environments.

#### \*Dependencies of Dependencies
Let's take a deeper look at ```node_modules``` directory and ```vendor``` directory.

![node_modules_structure](./images/node_modules.png)
![vendor_structure](./images/vendor.png)

As shown above, inside each module directory, there will be another ```package.json``` file and another ```node_modules``` directory. ```node_modules``` are nested. However, there is only one ```vendor``` folder among the whole project.

This means, for npm, each module library has its own dependencies, and the dependencies structure is like a tree.
For composer, the dependencies structure is flat, and libraries share dependencies.

### Run HTTP Server

After development environments and dependencies get setup, I can run a HTTP server on each stack.

#### Express Server

For MEAN stack, a HTTP server can be easily built by using Express framework [9].
(Node.js itself is not a server, it is a Javascript runtime.)

```javascript
var express = require('express');
var app = express();

app.listen(3000, function () {
  console.log('Server start listening on port 3000!');
});
```

#### Apache2 Server

For LAMP stack, Apache 2 [10] is used as a HTTP server.
With the use of docker and linode/lamp image, we could save some work on configuring Apache 2 server.

Under Linux, Apache 2 runs as a daemon. So it can be started using command line:
```
$ service apache2 start
```

### Serve Static Files

When HTTP servers are running, I want to use them to serve static files such as css files and browser side javascript files.

#### Express Server

For MEAN stack, Express framework uses a middleware to serve static files [11].

```javascript
app.use(express.static(path.resolve('./public')));
```

#### Apache2 Server

By default, if we go to http://example.com/script.php, Apache2 runs the script stored in ```root/script.php```, and response the result.
If there is no PHP script define inside ```script.php```, Apache2 will just return the content of the file.
This is exactly what we need to serve static files. So Apache server would serve static files by default.

### Express.js vs. Slim 3

Now the server will serve the static files defined in the URL path.
Next, I want to use some modern server features on each stack, such as RESTful API, middleware and Database interactions.

For MEAN stack, Express [9] is introduced as a web framework. To make the comporision fair for LAMP stack, and also to make the implementation easier, I use a PHP web framework called Slim 3 [12], helping building features on LAMP.

Express [9]
```javascript
var express = require('express');
var app = express();
```

Slim 3 [12]
```php
<?php
$app = new \Slim\App;
?>
```

#### Routes & Router

For a modern web server, the first thing is RESTful API, which at least different HTTP methods need to be separated for the same URI.
This is usually called routes or routers in web frameworks.

##### Express
```javascript
app.get('/users', function (req, res) {
  res.send('GET users');
});

app.post('/users', function (req, res) {
  res.send('POST users');
});

// or you can pack different methods handle into a controller
app.route('/users')
	.get(function (req, res) {
    res.send('GET users');
  })
	.post(function (req, res) {
    res.send('POST users');
  });
```

##### Slim 3
```php
<?php
$app->get('/users', function ($request, $response, $args) {
    return $response;
});

$app->post('/users', function ($request, $response, $args) {
    return $response;
});

// or you can pack different methods handle into a controller
$app->any('/users', 'UserController');
?>
```

With only this snippet of code, the LAMP server would not work as we expect.

As mentioned above, by default, Apache server directly serves the file or run the script in that file with the path specified in a URL.
However, for the RESTful API, a URL doesn't have to represent a real file locating somewhere in file system.
Instead, a URL would represents a resource, and it often stored in the database, and take some code to read.
For example, in above case, ```users``` is not a file, but a tables/resource store in the MySQL database.

Now, I need to tell Apache server when a URL like ```/users``` appears, don't try to find a file called ```users```, instead run a script called ```index.php```. Because in ```index.php```, I defined how to get user resource, which is the snippet of code above.

###### URL Rewriting

In LAMP stack, a configuration file called .htaccess is used to let Apache2 server know our special arrangement for this type of URLs.
This .htaccess file is stored in the root directory of the website, and contain following code [13]:

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```
It tells Apache server:
  - turn URL rewriting on
  - if the REQUEST_FILENAME is not found under this directory (as a file, or as a directory), run ```index.php``` instead.

Then the routes would work.

#### Middleware

The next feature I want to have is middleware.
Middleware works as extra layers of code server need to run before/after it runs your application code.

##### Express [14]
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
After done each middleware, developer need to explicitly call function ```next``` to run the next middleware.
And after all middleware are done and function ```next``` get called in last middleware, Express server start run the routes.

The design of middleware management would look like following graph in Express framework.
![express-middleware](./images/middleware-express.png)

##### Slim 3 [15]
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

Different from Express, the way Slim 3 handles middleware is treat them as wrappers.
Once adding a middleware, it wraps the exist middleware and app (routes) inside of itself.
This is why when defining a middleware, we need to put ```$response = $next($request, $response);``` clause in the middle of function, and call ``` retunr $response``` at the end.

The design of middleware management would look like following graph in Slim 3 framework. [15]
![slim-middleware](./images/middleware-slim.png)


### Database Interactions

Now the servers can serve static file, serve REST API and use middleware.
It time to let them serve data from databases.

For MEAN stack, a object modeling framework called Mongoose [16] is used to connect Node.js program with MongoDB.
Mongoose performs ORM (Object-relational mapping) to the database, providing OO objects interface for database interactions.
This means the data can be access in a manner of getting an object.

On the other side, for LAMP stack, PDO drivers (PHP Data Object drivers) [17] are used to accessing data from PHP scripts.
PDO is an interface, and it has different Implementation for different database. PDO use raw SQL command to access database.
In LAMP stack, a MySQL PDO driver is needed.

#### Installation

##### Mongoose
Use ```npm```
```
$ npm install mongoose
```
##### PDO Driver
A PDO driver for MySQL database called ```pdo_mysql.so``` is supposed to be automatically installed when installing PHP.
However, if it is not, install it using following command [18].
```
$ sudo apt-get install php5-mysql
```

After it is installed, it may not be activated for apache service.
To enable it, go to ```php.ini```, add or uncomment following lines [19].
```
extension=pdo.so
extension=pdo_mysql.so
```

#### Connect Database

##### Mongoose
```javascript
var connectString = 'mongodb://mongo:27017/learn-mean-auth-dev';
var db = mongoose.connect(connectString, function(err) {

});
```
##### PDO Driver
```php
<?php
$dbh = new PDO("mysql:host=$dbhost;dbname=$dbname", $dbuser, $dbpass);
$dbh->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
?>
```

##### Create Schema

With Mongoose and MEAN stack, we create a model schema directly in model module with javascript.

##### Mongoose
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

##### SQL Command

For LAMP stack, creating a schema means creating a table in database, and this should be done only once.
So it is better to do this with SQL command.
```sql
CREATE TABLE users (
username VARCHAR(255) NOT NULL,
...
CONSTRAINT username_pk PRIMARY KEY (username))
```

#### Create an object

After the table table is created, objects can be created based on the schema.

##### Mongoose
```javascript
var user = new User(data);
user.save(function(err, user) {

});
```

##### PDO Driver
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
And at last, we also need to make queries and get data out of database.

##### Mongoose
```javascript
User.find().sort('-created').exec(function(err, articles) {

});
```

##### PDO Driver
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

Since Mongoose provides ORM, it is much easier to do CURD operations with an object-like manner.
While with PDO drivers, raw SQL commands need to be written and embedded into PHP code.

### Template Engine

Although AngularJS can be used for client side application regardless the backend server's implementation.
Sometime a single page still need to be served. So I am still going have a template engine works on each stack.

For each stack, there are plenty of choices for template engines.
In my applications, I will use Swig [20] for MEAN stack and Twig [21] for LAMP stack.
They are as similar as their names.

#### Setup
##### Swig with Express [22]
```javascript
var swig = require('swig');
app.engine('html', swig.renderFile);
app.set('view engine', 'html');
app.set('views', path.join(__dirname, './templates'));
```

##### Twig with Slim 3 [23]
```php
<?php
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

Their templates languages are identical.

##### Swig Template
```html
<h1>{{ pagename|title }}</h1>
<ul>
{% for author in authors %}
  <li>{{ author }}</li>
{% endfor %}
</ul>
```

##### Twig Template
```html
<h1>{{ title }}</h1>
<ul>
{% for item in navigation %}
    <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
{% endfor %}
</ul>
```

#### Render Templates

##### Swig with Express [22]
```javascript
app.get('/', function (req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!'});
});
```
##### Twig with Slim 3 [23]
```php
<?php
$app->get('/', function ($request, $response, $args) {
  return $this->view->render($response, 'index.html', [
    'a_variable' => $foo
  ]);
});
?>
```

## Conclusion

In this report, I go through how to do some basic server features with both MEAN stack and LAMP stack.
At first, development environments and dependency management system are setup.
Then a HTTP server is run on each stack, and a few fundamental modern server feature are introduced and build on each each stack as well,
including RESTful API, middleware, database interactions and template engine.

Both stacks are managed to implement these modern server features, however, LAMP stack needs much more configurations than MEAN stack.
Due to the heavy setting up and configuration processes for LAMP stack, I personally recommend MEAN stack, especially for beginners.

## References

[1]"AngularJS — Superheroic JavaScript MVW Framework", Angularjs.org, 2016. [Online]. Available: https://angularjs.org/. [Accessed: 11- Apr- 2016].

[2]"Linux", Linux.com | The source for Linux information, 2016. [Online]. Available: https://www.linux.com/. [Accessed: 11- Apr- 2016].

[3]"MongoDB for GIANT Ideas | MongoDB", Mongodb.org, 2016. [Online]. Available: https://www.mongodb.org/. [Accessed: 11- Apr- 2016].

[4]"MySQL", Mysql.com, 2016. [Online]. Available: https://www.mysql.com/. [Accessed: 11- Apr- 2016].

[5]Hub.docker.com, 2016. [Online]. Available: https://hub.docker.com/r/rmukhia/im-meanjs/. [Accessed: 11- Apr- 2016].

[6]Hub.docker.com, 2016. [Online]. Available: https://hub.docker.com/r/linode/lamp/. [Accessed: 11- Apr- 2016].

[7]"npm", Npmjs.com, 2016. [Online]. Available: https://www.npmjs.com/. [Accessed: 11- Apr- 2016].

[8]"Composer", Getcomposer.org, 2016. [Online]. Available: https://getcomposer.org/. [Accessed: 11- Apr- 2016].

[9]"Express "Hello World" example", Expressjs.com, 2016. [Online]. Available: http://expressjs.com/en/starter/hello-world.html. [Accessed: 11- Apr- 2016].

[10]D. Group, "Welcome! - The Apache HTTP Server Project", Httpd.apache.org, 2016. [Online]. Available: https://httpd.apache.org/. [Accessed: 11- Apr- 2016].

[11]"Serve Static files with Middleware", Expressjs.com, 2016. [Online]. Available: http://expressjs.com/en/guide/using-middleware.html#middleware.built-in. [Accessed: 11- Apr- 2016].

[12]"Slim 3 Hello World", Slim Framework, 2016. [Online]. Available: http://www.slimframework.com/docs/#how-does-it-work. [Accessed: 11- Apr- 2016].

[13]"Apache Configuration for Slim 3", Slim Framework, 2016. [Online]. Available: http://www.slimframework.com/docs/start/web-servers.html#apache-configuration. [Accessed: 11- Apr- 2016].

[14]"Using Express middleware", Expressjs.com, 2016. [Online]. Available: http://expressjs.com/en/guide/using-middleware.html. [Accessed: 11- Apr- 2016].

[15]"Middleware", Slim Framework, 2016. [Online]. Available: http://www.slimframework.com/docs/concepts/middleware.html. [Accessed: 11- Apr- 2016].

[16]"Mongoose ODM v4.4.12", Mongoosejs.com, 2016. [Online]. Available: http://mongoosejs.com/. [Accessed: 11- Apr- 2016].

[17]"PHP: PDO Drivers - Manual", Php.net, 2016. [Online]. Available: http://php.net/manual/en/pdo.drivers.php. [Accessed: 11- Apr- 2016].

[18]H.  php5?, "How do I install and enable pdo_mysql and gd extensions for php5?", Askubuntu.com, 2016. [Online]. Available: http://askubuntu.com/questions/384677/how-do-i-install-and-enable-pdo-mysql-and-gd-extensions-for-php5. [Accessed: 11- Apr- 2016].

[19]H.  pdo_mysql, "How enable pdo_mysql", Serverfault.com, 2016. [Online]. Available: http://serverfault.com/questions/471282/how-enable-pdo-mysql. [Accessed: 11- Apr- 2016].

[20]"Swig - A Node.js and Browser JavaScript Template Engine", Paularmstrong.github.io, 2016. [Online]. Available: http://paularmstrong.github.io/swig/. [Accessed: 11- Apr- 2016].

[21]"Homepage - Twig - The flexible, fast, and secure PHP template engine", Twig.sensiolabs.org, 2016. [Online]. Available: http://twig.sensiolabs.org/. [Accessed: 11- Apr- 2016].

[22]C.  Express, "Cannot render swig templates in Express", Stackoverflow.com, 2016. [Online]. Available: http://stackoverflow.com/a/13907921. [Accessed: 11- Apr- 2016].

[23]"Templates", Slim Framework, 2016. [Online]. Available: http://www.slimframework.com/docs/features/templates.html. [Accessed: 11- Apr- 2016].
