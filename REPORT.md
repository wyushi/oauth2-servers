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

### Dependencies Management: npm vs. Composer

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

#### Serve Static Files

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

### OAuth 2.0 Framework
#### [OAuth 2.0 Server PHP](http://bshaffer.github.io/oauth2-server-php-docs/)
#### [OAuth2orize](https://github.com/jaredhanson/oauth2orize)

#### OAuth 2.0 Server setup

##### OAuth2orize (Node.js):
```javascript
var oauth2orize = require('oauth2orize'),
    server = oauth2orize.createServer();

server.serializeClient(function(client, callback) {
  return callback(null, client._id);
});

server.deserializeClient(function(id, callback) {
  console.log('id: ' + id);
  User.findOne({ _id: id }, function (err, client) {
    if (err) { return callback(err); }
    return callback(null, client);
  });
});
```
##### OAuth 2.0 Server PHP

Database
```sql
CREATE TABLE oauth_clients (client_id VARCHAR(80) NOT NULL, client_secret VARCHAR(80), redirect_uri VARCHAR(2000) NOT NULL, grant_types VARCHAR(80), scope VARCHAR(100), user_id VARCHAR(80), CONSTRAINT clients_client_id_pk PRIMARY KEY (client_id));
CREATE TABLE oauth_access_tokens (access_token VARCHAR(40) NOT NULL, client_id VARCHAR(80) NOT NULL, user_id VARCHAR(255), expires TIMESTAMP NOT NULL, scope VARCHAR(2000), CONSTRAINT access_token_pk PRIMARY KEY (access_token));
CREATE TABLE oauth_authorization_codes (authorization_code VARCHAR(40) NOT NULL, client_id VARCHAR(80) NOT NULL, user_id VARCHAR(255), redirect_uri VARCHAR(2000), expires TIMESTAMP NOT NULL, scope VARCHAR(2000), CONSTRAINT auth_code_pk PRIMARY KEY (authorization_code));
CREATE TABLE oauth_refresh_tokens (refresh_token VARCHAR(40) NOT NULL, client_id VARCHAR(80) NOT NULL, user_id VARCHAR(255), expires TIMESTAMP NOT NULL, scope VARCHAR(2000), CONSTRAINT refresh_token_pk PRIMARY KEY (refresh_token));
CREATE TABLE oauth_users (username VARCHAR(255) NOT NULL, password VARCHAR(2000), first_name VARCHAR(255), last_name VARCHAR(255), CONSTRAINT username_pk PRIMARY KEY (username));
CREATE TABLE oauth_scopes (scope TEXT, is_default BOOLEAN);
CREATE TABLE oauth_jwt (client_id VARCHAR(80) NOT NULL, subject VARCHAR(80), public_key VARCHAR(2000), CONSTRAINT jwt_client_id_pk PRIMARY KEY (client_id));
```

Create server
```php
<?php
$storage = new OAuth2\Storage\Pdo(array('dsn' => $dsn, 'username' => $username, 'password' => $password));
$server = new OAuth2\Server($storage,
    [
        'access_lifetime' => 3600,
    ],
    [
        new GrantType\ClientCredentials($storage),
        new GrantType\AuthorizationCode($storage),
    ]
);
?>
```

#### Authorize
##### OAuth2orize (Node.js):
```javascript
// route
app.route('/oauth2/authorize')
    .get(users.requiresLogin, requireRole('user'), oauth2Controller.authorization)
    .post(users.requiresLogin, requireRole('user'), oauth2Controller.decision);

// get authorization page
exports.authorization = [
  server.authorization(function(clientId, redirectUri, callback) {
    User.findOne({ username: clientId }, function (err, client) {
      if (err) { return callback(err); }
      return callback(null, client, redirectUri);
    });
  }),
  function(req, res){
    res.render('dialog', {
      transactionID: req.oauth2.transactionID,
      user: req.user,
      client: req.oauth2.client
    });
  }
];

// handle request for authorization code
server.grant(oauth2orize.grant.code(function(client, redirectUri, user, ares, callback) {  
  var code = new Code({
    value: uid(16),
    clientId: client._id,
    redirectUri: redirectUri,
    userId: user._id
  });
  code.save(function (err) {
    if (err) { return callback(err); }
    callback(null, code.value);
  });
}));

exports.decision = [
  server.decision()
];
```

##### OAuth 2.0 Server PHP
```php
<?php
// get authorization page
$app->get('/authorize', function ($request, $response, $args) use ($app){
    $server = $app->oauthServer;
    $oauthReq = OAuth2\Request::createFromGlobals();
    $oauthRes = new OAuth2\Response();

    if (!$server->validateAuthorizeRequest($oauthReq, $oauthRes)) {
        $oauthRes->send();
        die;
    }
    exit('
    <form method="post">
      <label>Do You Authorize TestClient?</label><br />
      <input type="submit" name="authorized" value="yes">
      <input type="submit" name="authorized" value="no">
    </form>');
});

// handle request for authorization code
$app->post('/authorize', function ($request, $response, $args) use ($app){
    $server = $app->oauthServer;
    $oauthReq = OAuth2\Request::createFromGlobals();
    $oauthRes = new OAuth2\Response();

    if (!$server->validateAuthorizeRequest($oauthReq, $oauthRes)) {
        $oauthRes->send();
        die;
    }
    $username = $_SERVER['PHP_AUTH_USER'];
    $is_authorized = ($_POST['authorized'] === 'yes');
    $server->handleAuthorizeRequest($oauthReq, $oauthRes, $is_authorized, $username);
    $oauthRes->send();
    die;
});
?>
```

#### Exchange Access_token
##### OAuth2orize (Node.js):
```javascript

// router module
app.route('/oauth2/token')
    .post(users.isAuthenticated, requireRole('oauthClient'), oauth2Controller.token);

// controller module
server.exchange(oauth2orize.exchange.code(function(client, code, redirectUri, callback) {
  Code.findOne({ value: code }, function (err, authCode) {
    if (err) { return callback(err); }
    if (!authCode) { return callback(null, false); }
    if (client._id.toString() !== authCode.clientId) { return callback(null, false); }
    if (redirectUri !== authCode.redirectUri) { return callback(null, false); }

    authCode.remove(function (err) {
      if(err) { return callback(err); }
      var token = new Token({
        value: uid(256),
        clientId: authCode.clientId,
        userId: authCode.userId
      });
      token.save(function (err) {
        if (err) { return callback(err); }

        callback(null, token);
      });
    });
  });
}));

exports.token = [
  server.token(),
  server.errorHandler()
];
```
##### OAuth 2.0 Server PHP
```php
<?php
$app->post('/token', function ($request, $response, $args) use ($app) {
    $server = $app->oauthServer;
    $oauthReq = OAuth2\Request::createFromGlobals();
    $server->handleTokenRequest($oauthReq)->send();
});
?>
```

#### Access Resource
##### OAuth2orize (Node.js):
```javascript
'use strict';

var passport = require('passport'),
  url = require('url'),
  BearerStrategy = require('passport-http-bearer').Strategy,
  Token = require('mongoose').model('Token'),
  User = require('mongoose').model('User');

module.exports = function() {
  passport.use(new BearerStrategy (
    function(accessToken, callback) {
      console.log('bearer auth: ' + accessToken);
      Token.findOne({value: accessToken }, function (err, token) {
        if (err) { return callback(err); }
        if (!token) { return callback(null, false); }
        User.findOne({ _id: token.userId }, function (err, user) {
          if (err) { return callback(err); }
          if (!user) { return callback(null, false); }
          callback(null, user);
        });
      });
    }
  ));
};
```

```javascript
exports.isAuthenticated = passport.authenticate(['bearer', 'basic'],
  {session: false});
```

```javascript
app.route('/api/articles')
  .get(users.isAuthenticated, articles.list);
```
##### OAuth 2.0 Server PHP
```php
<?php
$mw = function ($request, $response, $next) use ($app) {
    $oauthReq = OAuth2\Request::createFromGlobals();
    $server = $app->oauthServer;

    if (!$server->verifyResourceRequest($oauthReq)) {
        $server->getResponse()->send();
        $response->getBody()->write(json_encode([
            "message"=>"not auth"
        ]));
        return $response;
    }
    $tokenData = $server->getAccessTokenData($oauthReq);
    $_SERVER['PHP_AUTH_USER'] = $tokenData['user_id'];

    $response = $next($request, $response);
    return $response;
};
?>
```

```php
<?php
$app->get('/users/{name}', function ($request, $response, $args) {
    $authUser = $_SERVER['PHP_AUTH_USER'];
    $model = new User();
    $username = $request->getAttribute('name');
    $user = $model->getUserByName($username);
    $response->getBody()->write(json_encode($user));
    return $response;
})->add($mw);
?>
```
### Angular.js

## Does Stack really matters?
### web service
### docker
### Maybe A structure  
