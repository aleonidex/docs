---
title: What is an ID token
description: An ID token is a unique token issued to a service account and enabling it to authenticate with external systems that support OIDC authentication.
---

# Service account ID token

An [ID token](https://openid.net/specs/openid-connect-core-1_0.html#IDToken) is a unique short-lived token issued for a [service account](../users/service-accounts.md) and enabling it to authenticate with external systems that support [OIDC](https://openid.net/specs/openid-connect-core-1_0.html) authentication.

An ID token is generated by {{ yandex-cloud }} and used to get an access token in an external system. An ID token is valid for one hour. You cannot use an ID token for authentication when accessing the [{{ yandex-cloud }} API](../../../api-design-guide/index.yaml).

## ID token format {#token-format}

An ID token is a [JSON web token](https://en.wikipedia.org/wiki/JSON_Web_Token) (JWT) consisting of three parts separated by a dot, e.g., `<header>.<body>.<signature>`.

The ID token **header** includes the following attributes:

* `kid`: ID of the public signature key for token signature verification.
* `alg`: Signature algorithm.

The ID token **body** includes the following attributes:

* `iss`: URL of the {{ yandex-cloud }} OIDC provider, `https://auth.cloud.yandex.com`.
* `sub`: Service account [ID](../../operations/sa/get-id.md) provided in the `subject-id` parameter when creating the ID token.
* `aud`: Token target resource. Provide it in the `audience` parameter when creating the ID token. If left unspecified when creating the token, this value defaults to the service account ID from `subject-id`.
* `exp`: Token expiration time in [Unix Time](https://en.wikipedia.org/wiki/Unix_time) format. An ID token expires one hour after creation.
* `iat`: Token issue time in Unix Time format.
* `jti`: Unique ID token identifier that prevents the token reuse.

The **ID token signature** allows you to make sure the token has not been modified. To verify the signature, you need a public key. To [get](https://{{ auth-main-host }}/oauth/jwks/keys) a public key in {{ yandex-cloud }}, use the ID from the `kid` field of the token header.

To [get](../../operations/sa/get-id-token.md) an ID token for a service account, use the [{{ yandex-cloud }} CLI](../../../cli/index.yaml).