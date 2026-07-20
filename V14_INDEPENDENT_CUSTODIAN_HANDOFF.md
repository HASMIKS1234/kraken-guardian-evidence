# Kraken Guardian V14 Independent Custodian Handoff

Status: owner action required; paper-only; no trading or outcome access

## Why you are being asked

Kraken Guardian needs one independent person or organization to retain a
cryptographic copy of its research evidence. Your role is to prove that the
producer cannot silently rewrite or approve its own final evidence.

This role does not involve trading, exchange accounts, money, API keys,
strategy results, or access to private market data.

## Independence requirements

Accept this role only if all of the following are true:

- You are not the Kraken Guardian producer or its automation.
- You control a GitHub account or organization that the producer cannot use.
- The producer will have no write access to the repository or protected branch.
- You can keep one Ed25519 private key exclusively on hardware you control.
- You will review each future signature request before signing it.

If any requirement is false, decline the role. Creating a second producer-owned
account does not satisfy independence.

## What to create

1. Create a private GitHub repository under your independent account or
   organization.
2. Create a protected branch dedicated to Kraken Guardian evidence.
3. Permit create-only retention below:

   `artifacts/v14-finality-claims`

4. Do not grant the producer, `edagher92-coder`, or the producing automation
   write access.
5. On your own device, generate an Ed25519 keypair using a trusted local tool.
6. Keep the private key, GitHub credentials, recovery codes, and repository
   token entirely outside the producer's devices and repository.

## Return exactly four non-secret values

Send the producer only this completed block:

```text
REPOSITORY=OWNER/REPOSITORY
BRANCH=PROTECTED_BRANCH
REVIEWER_ID=YOUR_STABLE_REVIEWER_NAME
RAW_32_BYTE_PUBLIC_KEY_BASE64=CANONICAL_BASE64_PUBLIC_KEY
```

The public key is safe to share. The private key is not.

Also state, in your own words, that:

```text
I control the repository and branch independently. The Kraken Guardian
producer has no write access. The Ed25519 private key remains only in my
custody. The evidence path is protected for create-only retention.
```

## Never send

- The Ed25519 private key or seed.
- GitHub passwords, tokens, cookies, sessions, or recovery codes.
- Exchange credentials or wallet details.
- Remote desktop access to your signed-in account.
- A signature you have not reviewed.

## What happens after you respond

The producer runs an offline preflight using the four public values. That check
does not contact your repository and grants no operational permission. A
two-party review then verifies the proposed identity lock. Only after that
review may its exact hashes be pinned in code.

Future evidence requests will provide exact canonical bytes and hashes for you
to inspect and sign on your device. A valid signature proves retention only.
It never authorizes trading, exchange contact, strategy promotion, outcome
access, margin, leverage, transfers, or withdrawals.

## Completion condition

Your setup is complete only when both parties independently confirm the
repository boundary, public-key fingerprint, canonical lock bytes, and exact
hashes. The producer's software must continue to report every operational
permission as false at this stage.
