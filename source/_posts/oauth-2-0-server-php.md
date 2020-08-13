---
title: OAuth 2.0 Server  PHP
tags: []
id: '279'
categories:
  - - all-blog
    - PHP
date: 2018-08-15 18:15:45
---

### 要求

此库需要PHP 5.3.9+。然而，有一个稳定的版本和开发分支的PHP 5.2.x-5.3.8为好。

### 安装

该库遵循zend PSR-0标准。由于这个原因，存在许多可以自动加载此库的自动加载器，但是如果您不使用它，则可以注册OAuth2\\Autoloader：

```php
require_once('/path/to/oauth2-server-php/src/OAuth2/Autoloader.php');
OAuth2\Autoloader::register();
```

使用Composer 执行以下命令：

```shell
composer.phar require bshaffer/oauth2-server-php "^1.10"
```

这会将需求添加到composer.json并安装库。 **强烈建议您检查v1.10.0标记，以确保您的应用程序不会破坏向后兼容性问题。但是，如果您希望保持开发的最前沿，可以将其设置为dev-master相反。**

### 开始使用此库

```php
$storage = new OAuth2\Storage\Pdo(array('dsn' => $dsn, 'username' => $username, 'password' => $password));
$server = new OAuth2\Server($storage);
$server->addGrantType(new OAuth2\GrantType\AuthorizationCode($storage)); // or any grant type you like!
$server->handleTokenRequest(OAuth2\Request::createFromGlobals())->send();
```

### 用户关联

Once you’ve authenticated a user and issued an access token (such as with an Authorize Controller), you’ll probably want to know which user an access token applies to when it is used. You can do this by using the optional user\_id parameter of handleAuthorizeRequest:

```php
$userid = 1234; // A value on your server that identifies the user
$server->handleAuthorizeRequest($request, $response, $is_authorized, $userid);
```

That will save the user ID into the database with the access token. When the token is used by a client, you can retrieve the associated ID:

```php
if (!$server->verifyResourceRequest(OAuth2\Request::createFromGlobals())) {
    $server->getResponse()->send();
    die;
}

$token = $server->getAccessTokenData(OAuth2\Request::createFromGlobals());
echo "User ID associated with this token is {$token['user_id']}";
```