# brainlock-php

Official PHP SDK for [BrainLock](https://brainlock.id) — memory-based identity proof for partner apps. Zero hard dependencies; uses the `curl` and `openssl` extensions that ship with PHP 8.1+.

> **Status: pre-release.** API surface is stable for Connect (`v0.4.0`+) and Verify (`v0.5.0`+). Watch for the `v1.0` tag.

## Two products on one SDK

BrainLock ships as two distinct products that share the same protocol primitives:

- **BrainLock Connect** — passwordless sign-in / sign-up. Same shape as "Sign in with Apple". Returns a fixed identity bundle (first name, last name, email, picture-if-uploaded).
- **BrainLock Verify** — per-action approval for high-stakes moments (money transfers, password changes, seed-phrase reveals, admin operations, document e-signatures). Returns an action receipt — what the user authorized + an audit-log id.

One install, one configure step, then a different call depending on which product you reach for.

## Install

Drop the single file into your project — no Composer package, no autoload setup:

```bash
curl -O https://raw.githubusercontent.com/xtiaan3/brainlock-php/main/src/BrainLock.php
```

```php
require 'BrainLock.php';
```

That's it. No `composer require`, no `vendor/` directory. The SDK is one file (~45 KB) and uses only the curl + openssl extensions that ship with PHP.

## Configure once

```php
BrainLock::configure([
    'api_key'      => $_ENV['BRAINLOCK_API_KEY'],     // bl_live_… or bl_test_…
    'callback_url' => 'https://yourapp.com/auth/callback',
]);
```

## Connect — sign in

```php
// On your "Sign in with BrainLock" button:
BrainLock::connect([
    'user_id' => session_id(),     // your stable user id (or session id on first sign-in)
]);
// Never reached — connect() redirects and exits.
```

```php
// In your callback handler:
try {
    $identity = BrainLock::verifyConnectToken($_GET['token']);
} catch (BrainLockException $e) {
    error_log('BrainLock verify failed: ' . $e->getMessage());
    header('Location: /signin?error=token_invalid');
    exit;
}

// $identity = [
//     'sub'             => '...',                        // BL user id (rotates — use email as your dedupe key)
//     'first_name'      => 'Christiaan',
//     'last_name'       => 'Rendle',
//     'email'           => 'c@example.com',
//     'picture'         => 'https://...',                // one-shot presigned URL; download once
//     'verification_id' => 'verif_01J6XK...',            // audit-log id — store it
//     'verified'        => true,
//     'biometric_used'  => true,
// ]
```

## Verify — approve a specific action

```php
// On a high-stakes action (e.g. user clicks "Send $5,000"):
BrainLock::verifyAction([
    'user_id' => $currentUser->id,
    'action'  => 'transfer_funds',
    'context' => [
        'amount_cents' => 500000,
        'currency'     => 'USD',
        'recipient'    => 'alice@example.com',
    ],
    'security_level' => 'elevated',
]);
// Never reached — verifyAction() redirects and exits.
```

```php
// In your callback handler:
try {
    $receipt = BrainLock::verifyActionToken($_GET['token']);
} catch (BrainLockException $e) {
    error_log('BrainLock verify failed: ' . $e->getMessage());
    header('Location: /transfers?error=token_invalid');
    exit;
}

// CRITICAL: confirm the returned action matches what you started.
if (($receipt['action'] ?? '') !== 'transfer_funds') {
    header('Location: /transfers?error=action_mismatch');
    exit;
}

// $receipt = [
//     'sub'             => '...',
//     'action'          => 'transfer_funds',             // echo
//     'context'         => [...],                        // echo
//     'verification_id' => 'verif_01J6XK...',
//     'verified'        => true,
//     'biometric_used'  => false,
// ]

// Run the action. Store $receipt['verification_id'] alongside the transfer.
executeTransfer(...);
```

Verify is always a top-level redirect — no iframe / popup transport.

## Cross-site SSO is free

Once a user has signed in via BrainLock anywhere on a device — direct on brainlock.id, or through any partner — every subsequent BrainLock sign-in on the same device flashes through in ~600ms with a consent moment instead of a fresh ceremony. The brainlock.id session cookie set during step one carries across every other "Sign in with BrainLock" integration automatically. Zero extra wiring on your side.

## Trusted-device + biometric shortcut works automatically

If the user has enrolled biometrics on a trusted device, BrainLock recognizes the device when they hit the sign-in page and offers a one-tap "Sign in with biometrics" alternative. This applies to your partner integration with zero work from you — your users get the polish you'd otherwise have to build yourself.

## Experimental: iframe transport (Connect only)

A `mode: 'iframe'` option in `configure()` loads the Connect UI in a same-origin iframe via a server-side proxy. It looks slicker for first-time sign-in but **breaks cross-site SSO** (each partner gets a partitioned BrainLock session that doesn't transfer) and **defeats the trusted-device + biometric shortcut**. We don't recommend it for most cases. Verify is always redirect — the iframe option doesn't apply.

## Public methods

| Method | Use |
|---|---|
| `BrainLock::configure(array $config)` | One-time setup; required before any other call. |
| `BrainLock::connect(array $opts)` | Kick off a Connect sign-in. Redirects and exits. |
| `BrainLock::startSession(array $opts)` | Lower-level: create a Connect session, return its `redirect_url`. Use when you want to drive the redirect yourself. |
| `BrainLock::verifyConnectToken(string $token)` | Validate a Connect callback JWT. Returns the identity bundle. |
| `BrainLock::verifyAction(array $opts)` | Kick off a Verify approval. Redirects and exits. |
| `BrainLock::verifyActionToken(string $token)` | Validate a Verify callback JWT. Returns the action receipt. |
| `BrainLock::handleEmbed()` | Same-origin proxy forwarder for the optional iframe transport. |

A `BrainLockException` is thrown on signature failure, expired tokens, malformed JWTs, missing config, network errors, and intent mismatch (e.g. a Verify token redeemed by `verifyConnectToken()`).

## Reference implementation

See [tangocash](https://github.com/xtiaan3/tangocash) for the canonical example app — a peer-to-peer wallet built on this SDK.

## Full reference

- API spec: https://brainlock.id/developer/docs/api-v1
- SDK doc: https://brainlock.id/developer/sdks/php
- Audit log: https://brainlock.id/developer/docs/api-v1#audit-log

## License

MIT — see [LICENSE](LICENSE).
