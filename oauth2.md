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
