# Krateo AuthN Helm Chart

This is a [Helm Chart](https://helm.sh/docs/topics/charts/) for [Krateo AuthN](https://github.com/krateoplatformops/authn).

## Requirements

authn signs JWTs asymmetrically with **RS256** and publishes the matching public
key as a JWKS at `GET /.well-known/jwks.json`. Before installing, a Secret holding
the **PEM-encoded RSA private key** must exist in the release namespace.

By default the chart expects a Secret named `jwt-sign-key` with the key under
`private.pem` (configurable via `jwt.signKeySecretName` / `jwt.signKeySecretKey`).
The Secret is mounted into the pod as a file at `/etc/authn/jwt/private.pem` and
exposed to the process via `JWT_SIGN_KEY_FILE` — it is **not** injected as an
environment variable value.

Generate a keypair and create the Secret:

```sh
# 1. Generate a 2048-bit RSA private key (the public half is derived by authn).
openssl genrsa -out private.pem 2048

# 2. Create the Secret from that file. The key name (private.pem) must match
#    .Values.jwt.signKeySecretKey.
kubectl create secret generic jwt-sign-key \
  --namespace <authn-namespace> \
  --from-file=private.pem=./private.pem
```

You must also set a stable key ID (`kid`) via `jwt.kid` (default
`krateo-authn-key-1`). It is stamped into every token header and the JWKS, and
validators use it to select the key — keep it constant for the key's lifetime.

> **Migrating from HS256?** Earlier versions used a shared symmetric secret
> (`stringData.JWT_SIGN_KEY`). That is no longer supported: replace the Secret
> with the PEM private key shown above. See
> [`authn/docs/jwt-jwks.md`](https://github.com/krateoplatformops/authn/blob/main/docs/jwt-jwks.md)
> for the full JWT/JWKS reference, including agentgateway and Snowplow wiring.

## How to install

```sh
helm repo add krateo https://charts.krateo.io
helm repo update krateo
helm install authn krateo/authn
```
