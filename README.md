# A2A Agent Card Key Service

<!-- 561-group-standards:badges:start -->
[![Red Cup Engineering](https://img.shields.io/badge/561%20Group-Red%20Cup%20Engineering-c0392b)](https://github.com/red-cup-engineering)
[![License: MIT](https://img.shields.io/badge/license-MIT-c0392b)](LICENSE)
[![Indigenous data: CARE](https://img.shields.io/badge/Indigenous%20data-CARE-4b6f44)](https://github.com/red-cup-engineering/.github/blob/main/GOVERNANCE.md)
[![Tips welcome](https://img.shields.io/badge/tips-welcome-d4a017)](https://github.com/561-group/.github/blob/main/TIPS.md)
<!-- 561-group-standards:badges:end -->

**Provision a signing key for an Agent-to-Agent (A2A) Agent Card.** This is
a small Node.js utility that creates an Ed25519 private signing key and its
corresponding public JSON Web Key Set (JWKS). A provider uses these to sign
its published Agent Card so a caller can verify the card's origin using
published public keys.

This service **does not** run an agent, publish the Agent Card, serve the
JWKS over HTTP, or act as a secret-management system.

## Where to start

The root package is marked `private: true`. Its usable command is the
local [key provisioner](src/provision-agent-card-key.mjs), declared in the
package manifest as `provision-agent-card-key`. Run it from a checkout
with dependencies available and with paths under separately managed
private/public storage, **not** a directory you plan to commit:

```sh
node src/provision-agent-card-key.mjs \
  --signing-key /secure/agent-card-signing.pem \
  --jwks /public/agent-card-jwks.json \
  --kid agent-card-key-1
```

Provide real paths appropriate to your host. It creates the directories
as needed, refuses to overwrite an existing file, writes the private key
as PKCS#8 PEM with mode `0600`, writes a public JWKS JSON file with mode
`0644`, and prints only the chosen key identifier (`kid`). Keep the
private key out of Git and under the host's secret-custody controls.
Protect the parent directories too; file permissions alone are not a
complete key-management policy.

## What happens next

The separate
[contracted A2A agent provider](https://github.com/red-cup-engineering/a2a-contracted-agent-service)
can consume the private key, public JWKS and key identifier when configured
by its host, then expose the public JWKS at
`/.well-known/jwks.json`. A remote party can use
[Agent Card discovery](https://github.com/red-cup-engineering/a2a-agent-card-discovery-service)
to check the published signature against those keys.

For tests, the [`./testing` export](src/testing.mjs) offers
`ephemeralAgentCardSigning(baseUrl, label)`. It creates a temporary
in-memory fixture configuration, **not** a durable production key.

## Status and boundaries

The repository's `package.json` declares version `1.0.2`; the code
provides one provisioner and one testing helper, not a generic public
key-management API. With workspace dependencies resolved, use
`npm run verify` to check syntax and run the package tests.

Generating a key is not proof that any Agent Card is published, the provider
is reachable, its advertised claims are true, or its runtime is authorized.
Those claims belong to the provider's own operating evidence.
