# brainlock-php

Official PHP SDK for [BrainLock](https://brainlock.id) — passwordless identity for partner apps.

> **Status: under construction.** Watch for the `v1.0` tag.

## Install

Via Composer (recommended):

```bash
composer require brainlock/sdk
```

Or drop a single file into your project — no Composer required.

## Usage

```php
require 'vendor/autoload.php';

BrainLock::configure([
    'api_key'      => $_ENV['BRAINLOCK_API_KEY'],
    'callback_url' => 'https://example.com/auth/callback',
]);

// On your sign-in button:
BrainLock::connect(['user_id' => session_id()]);

// On your callback endpoint:
$identity = BrainLock::verify($_GET['token']);
// $identity = ['sub' => '...', 'name' => '...', 'email' => '...', 'picture' => '...']
```

The signature is locked. The implementation underneath will evolve until `v1.0`.

## Transport modes

The SDK picks the right cross-origin transport for each user automatically. You don't need to write any client-side JS or detect browser versions.

| Mode         | What it does                                                                                        |
|--------------|-----------------------------------------------------------------------------------------------------|
| `iframe` (default) | BrainLock UI loads in a full-viewport iframe over your page. Modern browsers (Safari 18.2+, Chrome 118+, Firefox 130+, Edge 118+) get this. The bundled launcher does a capability check and silently falls back to `redirect` for older browsers. |
| `redirect`   | Classic full-page navigation to brainlock.id and back. Works everywhere. Use this explicitly if you'd rather not have the in-page iframe.                |

Set it via `BrainLock::configure(['mode' => 'redirect', ...])` to force one. Otherwise we pick `iframe` and you get the best experience the user's browser can support.

## What BrainLock Connect always returns

Every successful sign-in returns the same fixed identity bundle:

| Field     | Always returned?                     |
|-----------|--------------------------------------|
| `sub`     | Yes — stable user identifier         |
| `name`    | Yes — first + last                   |
| `email`   | Yes                                  |
| `picture` | Only if the user has uploaded one    |

There are no scopes to request. The bundle is the bundle.

## Reference implementation

See [tangocash](https://github.com/xtiaan3/tangocash) for the canonical example app — a peer-to-peer wallet built on this SDK.

## License

MIT — see [LICENSE](LICENSE).
