# 🔐 OAuth / OIDC Testing Standards (Subscription Domain)
**Path:** `.cursor/rules/testing/oauth-testing.mdc`

Testing rules for OAuth2 / OIDC flows (Auth0 / Okta / Custom IdP) in a **Subscription API** environment.

Goal: deterministic, fast, secure tests validating token flows without hitting the real IdP.

---
## 🎯 Purpose
- Validate subscription-specific auth flows **without external network calls**.
- Ensure correct permission / scope enforcement (e.g., `subscriptions:write`).
- Cover positive + security edge cases deterministically.
- Support CI-friendly test runs.

---
## 🏛 Layer Scope
| Layer | Responsibility |
|--------|----------------|
| **Domain** | Represent identity via Value Objects (`UserId`, `SubscriptionId`, `ActorScope`). No OAuth logic here. |
| **Application** | Applies authorization policies when executing commands (e.g., `ActivateSubscription`). |
| **Infrastructure** | OAuth client, JWKS, discovery, HTTP integration, token verification. |

> The Domain should never know OAuth exists.

---
## 🧪 Test Environment Rules
- No external HTTP calls — **fake discovery, JWKS, token exchange, userinfo**.
- Freeze time (`Carbon::setTestNow()`) to validate `iat` / `exp` logic.
- Generate ephemeral RSA keys for token signing.

---
## 🧰 Faking the IdP (Laravel HTTP Client)
```php
Http::fake([
  'https://idp.example.com/.well-known/openid-configuration' => Http::response([
    'issuer' => 'https://idp.example.com',
    'token_endpoint' => 'https://idp.example.com/oauth/token',
    'jwks_uri' => 'https://idp.example.com/.well-known/jwks.json',
    'userinfo_endpoint' => 'https://idp.example.com/userinfo',
  ], 200),

  'https://idp.example.com/oauth/token' => Http::response([
    'access_token' => 'ACCESS.TOKEN',
    'id_token' => $signedIdToken,
    'token_type' => 'Bearer',
    'expires_in' => 3600,
  ], 200),

  'https://idp.example.com/.well-known/jwks.json' => Http::response(['keys' => [$publicJwk]], 200),
]);
```

---
## 🔐 Building Signed Tokens (Subscription Example)
```php
$claims = [
  'iss' => 'https://idp.example.com',
  'aud' => 'subscriptions-api',
  'sub' => 'user-123',
  'scope' => 'subscriptions:write subscriptions:activate',
  'subscription_id' => 'sub_001',
  'iat' => $now,
  'exp' => $now + 3600,
];
$signedIdToken = JWT::encode($claims, $privateKeyPem, 'RS256', 'kid-1');
```
> Include domain-specific claims when relevant (e.g., `subscription_id`).

---
## ✅ Happy Path (User activates subscription)
Goal: user logs in → IdP redirects back → handler activates subscription.

**Feature Test**
```php
Carbon::setTestNow('2025-01-01');
$this->withSession(['oauth.state' => 'state123', 'oauth.nonce' => 'nonce123']);

$response = $this->get('/oauth/callback?code=abc&state=state123');

$response->assertRedirect('/app/subscriptions');
$this->assertAuthenticated();
```

**Application Handler example**
```php
class ActivateSubscriptionHandler
{
    public function handle(ActivateSubscription $cmd): void
    {
        $this->policy->canActivate($cmd->actorId);
        $subscription = $this->repo->get($cmd->subscriptionId);
        $subscription->activate();
        $this->repo->save($subscription);
    }
}
```
> OAuth proves identity; **Application layer decides permissions**.

---
## 🛡 Security Edge Cases
- Invalid `state` → reject redirect
- Missing / mismatched `nonce`
- Expired token (`exp < now`)
- Future `iat` beyond allowed skew
- Audience mismatch (`aud !== subscriptions-api`)
- Missing required scopes (e.g., no `subscriptions:write`)

```php
$response = $this->get('/oauth/callback?code=ok&state=wrong');
$response->assertSessionHas('oauth.error');
```

---
## 🚫 Anti-Patterns
| Problem | Instead |
|----------|---------|
| Hitting real IdP in CI | Fake OAuth HTTP endpoints |
| Authorization inside controller | Application Handler + Policy |
| Trusting email claim without verification | Validate `email_verified` + domain lookup |
| Accepting tokens without checking `aud`/`scope` | Enforce required scopes per API route |

---
## ✅ Final Checklist
- IdP is fully **faked** (no network calls)
- Time frozen (`Carbon::setTestNow()`)
- Signed tokens include correct `aud`, `iat`, `exp`, `scope`
- Policies verify subscription-level access
- Application handlers perform orchestration
- Domain remains OAuth-agnostic

---
> OAuth authenticates the user.  
> The **Domain rules** decide if they are allowed to activate a subscription.