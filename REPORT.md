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
  - DB interaction
  - Template Engine

### Environment Setup

To build a 

The installations for either MEAN stack or LAMP stack on a Linux would involve bunch of commands, reboots and configurations.
To make life easier, I use docker and two popular docker images.

### Dependencies Management

#### npm & package.json

```bash
$ npm install
```
It will generate ```node_modules``` folder, and store all dependencies inside. To include modules, Node.js need to explicitly name the modules by using

```javascript
var widget = require('module-name');
```

#### Composer, composer.json & composer.lock
```bash
$ composer install
```
It will generate ```vendor``` folder, and store all dependencies inside.
Composer also generates a file inside ```vendor``` called ```autoload.php```. This file is inlcuded in ```index.php```, and it will auto load all the PHP modules in ```vendor``` folder.
After running install command, Composer also generates a JSON file called ```composer.lock```. This file stores a list of exact versions  it installed.
When a new development environment need to be setup, if ```composer.lock``` file is present, Composer will ignore ```composer.json``` file, and install the dependencies based on the lock file.
This mechanism ensures the program code could tight bind with its dependencies versions. At the same time, this also makes environment setup easier inside a team, in which their members need to have exactly the same environment to make sure their code could work together.

Thus, the dependencies would not update automatically. It need to be done manually.
```bash
$ composer update
```

#### node_modules vs. vendor
Let's take a close look at node_modules directory and vendor directory.
![node_modules_structure](./images/node_modules.png)
![vendor_structure](./images/vendor.png)

After installing finished, all the dependent libraries would be store in node_modules directory and vendor directory.

As shown above, inside each node_modules library directory, there will be a package.json file and another node_modules directory. Node_modules can be nested. This deeper node_modules directory stores the dependencies of this library. By this way, every module uses its own dependencies inside its own Node_modules directory.

Meanwhile, inside each vendor library directory, there is only a composer.json file, no composer.lock file and no vendor directory. This means the libraries of the project would not have their own dependencies, all libraries shares the same vendor directory. All dependencies are stored in the same folder, no matter which project or libraries they belong to. This mechanism saves space, but it requires all the dependencies are compatible to each other.

For example, our project depends on two library A, B. And both library A and B depend on an utils library called X. However, they are depends on different versions of X.
In this case, npm would create a node_modules directory for each of A and B, and install the correct version of X in side their own node_modules libraries.
On the other side, composer would install one version of X inside the vendor directory (where A and B also located there), and raise a warning.

### Run HTTP Server

For the server layer, the two stack are very different. LAMP relies on the very successful Apache server, while with MEAN stack, we need to use Express framework to build one.

#### Node.js Server

Unlike LAMP stack, an Apache2 server will be involved, serving files, and running scripts, Node.js is not a server instead it is a Javascript runtime. Which means, we need to build a HTTP server ourselves.

This may sound hard, but in fact, with Express framework, it could be just a few lines of code.

```javascript
var express = require('express');
var app = express();

app.listen(3000, function () {
  console.log('Server start listening on port 3000!');
});
```

#### Apache2 Server

##### Default URL rewriting Rule

By default, if we go to http://example.com/script.php, Apache2 runs the script stored in root/script.php, and response the result.

For example, the script.php looks like following:
```php
<?php
  echo "hello world";
?>
```
The response would be:
```
hello world
```
As we notice, this is not HTML. In order to make the page shows correct in browser, we need to add HTML tags to echo text.
```php
<?php
  echo "<h1>hello world</h1>";
?>
```

##### .htaccess Hypertext Access
.htaccess file is a configuration file used by Apache2 server to control the access of the server to the director .htaccess under and its subdirectory.

This is also where we are going to define the URL rewriting rules.

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```
- The first line tells the server to enable the URL rewriting under this directory.
- The second and third lines define two conditions under which the following rewriting rules would applied.
- The fourth line defines the rewriting rule.

Overall, this .htaccess file tells the Apache server, if the REQUEST_FILENAME is not found under this directory, run index.php instead. By this way, the router pattern becomes possible with Apache server.

### Serve Static Files

##### Express Server
```javascript
app.use(express.static(path.resolve('./public')));
```

##### Apache2 Server
Based on the request rewriting rules we talked about before, we can just put a folder called ```static``` under the public folder, and the Apache 2 server will serve files in ```static``` folder based on the relative path provided in the url.

File structure will be looked like below:
```
public_html
  static
    static_file_1
    static_file_2
    ...
```

### Express.js vs. Slim 3

#### Routes
Express:
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
Slim 3:
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

#### Middleware
Express:
```javascript
app.use('/users/:name', function (req, res, next) {
  ...
  next();
});
```
Slim 3:
```php
<?php
$mw = function ($request, $response, $next) use ($app) {
    ...
    return $response;
};

$app->get('/users/{name}', function ($request, $response, $args) {
    ...
    return $response;
})->add($mw);
?>
```

<!-- TODO: middleware design logic Comparison -->


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
