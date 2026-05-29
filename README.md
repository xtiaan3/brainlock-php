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

## How it works

`connect()` issues a 302 redirect to `brainlock.id/auth/<session>`. The user completes the sign-in ceremony on brainlock.id in the same browser window — real address bar, no popup, no iframe. BrainLock then redirects the browser back to your `callback_url` with `?token=<JWT>`. Your callback calls `verify()` to validate the token and extract the identity.

Exactly the shape of "Sign in with Google" / Apple / Microsoft.

### Cross-site SSO is free

Once a user has signed in via BrainLock anywhere on a device — direct on brainlock.id, or through any partner — every subsequent BrainLock sign-in on the same device **flashes through in ~600ms** with a consent moment instead of a fresh ceremony. The brainlock.id session cookie set during step one carries across every other "Sign in with BrainLock" integration automatically. Zero extra wiring on your side.

### Trusted-device + biometric shortcut works automatically

If the user has enrolled biometrics on a trusted device, BrainLock recognizes the device when they hit the sign-in page and offers a one-tap "Sign in with biometrics" alternative. This applies to your partner integration with zero work from you — your users get the polish you'd otherwise have to build yourself.

### Experimental: iframe transport

There's also a `mode: 'iframe'` option that loads the BrainLock UI in a same-origin iframe via a server-side proxy. It looks slicker for first-time sign-in but **breaks cross-site SSO** (each partner gets a partitioned BrainLock session that doesn't transfer) and **defeats the trusted-device + biometric shortcut**. We don't recommend it. Full tradeoffs in [brainlock-go/docs/CONNECT_TRANSPORTS.md](https://github.com/xtiaan3/brainlock-go/blob/main/docs/CONNECT_TRANSPORTS.md).

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
