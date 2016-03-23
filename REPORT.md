# OAuth 2.0 Server on MEAN stack and LAMP stack

## Introduction

## Comparison

### Linux

Everybody runs their server on Linux.

### HTTP Server: Node.js vs. Apache 2

For the server layer, the two stack are very different. LAMP relies on the very successful Apache server, while with MEAN stack, we need to use Express framework to build one.

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

#### Project Structure

### Database
#### MySQL vs. MongoDB

MySQL is the most famous relational database, while MongoDB is the most famous non-relational database (document-oriented database). The difference between them is a big topic, which is out of the scope of this report. Here, I will talk about the difference when we try to access them from our code.

#### Mongoose vs. PDO MySQL Driver

MEAN stack uses an MongoDB object modeling framework called Mongoose to do CURD operations. It provide object like interface to access the data, hiding the database command detail from server application developing.

PHP Data Object (PDO) is a interface for accessing data in PHP. PDO itself does not do anything, it is just a interface. PDO drivers do the actual work. Since we use MySQL database, so relatively we need to use a MySQL PDO Driver to do the CURD operations. Comparing with Mongoose, PDO drivers use more original sql command to access the database.

\* A PDO driver for MySQL database called ```pdo_mysql.so``` should be automatiically installed when you install PHP. If not, install it using
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
PDO:
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
PDO:
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
PDO:
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

Javascript for Node.js and PHP 5.3+ are different from each other in many ways.

#### Module Importation




### Express.js vs. Slim 3
#### MVC pattern


#### RESTful - Router
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


### OAuth 2.0 Framework
#### [OAuth 2.0 Server PHP](http://bshaffer.github.io/oauth2-server-php-docs/)
#### [OAuth2orize](https://github.com/jaredhanson/oauth2orize)

### Angular.js

## Does Stack really matters?
### web service
### docker
### Maybe A structure  
