# Harvest Provider for OAuth 2.0 Client
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)

This package provides Harvest OAuth 2.0 support for the PHP League's [OAuth 2.0 Client](https://github.com/thephpleague/oauth2-client).

## Installation

To install, use composer:

```
composer require xvilo/oauth2-harvest
```

## Usage

Usage is similar to the basic OAuth client, using `\Omines\OAuth2\Client\Provider\harvest` as the provider.

### Authorization Code Flow

```php
$provider = new Omines\OAuth2\Client\Provider\harvest([
    'clientId'          => '{harvest-client-id}',
    'clientSecret'      => '{harvest-client-secret}',
    'redirectUri'       => 'https://example.com/callback-url',
]);

if (!isset($_GET['code'])) {

    // If we don't have an authorization code then get one
    $authUrl = $provider->getAuthorizationUrl();
    $_SESSION['oauth2state'] = $provider->getState();
    header('Location: '.$authUrl);
    exit;

// Check given state against previously stored one to mitigate CSRF attack
} elseif (empty($_GET['state']) || ($_GET['state'] !== $_SESSION['oauth2state'])) {

    unset($_SESSION['oauth2state']);
    exit('Invalid state');

} else {

    // Try to get an access token (using the authorization code grant)
    $token = $provider->getAccessToken('authorization_code', [
        'code' => $_GET['code'],
    ]);

    // Optional: Now you have a token you can look up a users profile data
    try {

        // We got an access token, let's now get the user's details
        $user = $provider->getResourceOwner($token);

        // Use these details to create a new profile
        printf('Hello %s!', $user->getName());

    } catch (Exception $e) {

        // Failed to get user details
        exit('Oh dear...');
    }

    // Use this to interact with an API on the users behalf
    echo $token->getToken();
}
```

### Managing Scopes

When creating your Harvest authorization URL, you can specify the state and scopes your application may authorize.

```php
$options = [
    'scope' => ['read_user','openid'] // array or string
];

$authorizationUrl = $provider->getAuthorizationUrl($options);
```
If neither are defined, the provider will utilize internal defaults ```'api'```.

## Testing

```bash
$ ./vendor/bin/phpunit
```

## Contributing

Please see [CONTRIBUTING](https://github.com/xvilo/oauth2-harvest/blob/master/CONTRIBUTING.md) for details.


## Credits

This code is a modified fork from the [Omines GitLab Provider for OAuth 2.0 Client](https://github.com/omines/oauth2-gitlab) adapted
for harvest use, so many credits go to [Steven Maguire](https://github.com/stevenmaguire) and [Niels Keurentjes](https://github.com/curry684).

## Legal

This software was developed for personal use. It is shared with the general public under the permissive MIT license, without
any guarantee of fitness for any particular purpose. Refer to the included `LICENSE` file for more details.
