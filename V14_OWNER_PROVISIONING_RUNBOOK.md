# V14 Independent Reviewer Provisioning Runbook

Status: owner action required; paper-only; no endpoint or outcome access

Send `docs/V14_INDEPENDENT_CUSTODIAN_HANDOFF.md` to the proposed independent
custodian before collecting any public provisioning values.

This runbook provisions the public identity used to retain and independently
countersign V14 evidence. It does not authorize exchange contact, outcome
access, promotion, credentials, or execution.

## Boundary

- The independent reviewer repository must not be writable by the producing
  Kraken Guardian automation.
- Generate and retain the Ed25519 private key outside this repository and this
  environment. Never paste or commit the private key or repository token here.
- Only the canonical base64 encoding of the raw 32-byte public key and its
  SHA-256 fingerprint are non-secret inputs to this process.
- Production hash pins remain unset until a separate two-party review accepts
  the exact proposed child-lock bytes.

## 1. Owner-provisioned resources

Create an independently controlled GitHub repository and protected branch.
The branch must permit create-only retention under:

`artifacts/v14-finality-claims`

Record these non-secret values:

```text
OWNER/REPOSITORY
REVIEWER_BRANCH
REVIEWER_ID
RAW_32_BYTE_PUBLIC_KEY_BASE64
```

Keep any fine-grained repository token outside this environment.

## 2. Run the offline owner-boundary preflight

Before writing a child lock, run the combined preflight with all four explicit
owner attestations. The reviewer repository must be outside the producing
GitHub owner and installation. This command performs no network request, writes
nothing, accepts no private key or repository credential, and does not retain
the raw public-key value in its output.

```powershell
.\.venv\Scripts\python.exe scripts\preflight_v14_owner_provisioning.py `
  --repository "OWNER/REPOSITORY" `
  --branch "REVIEWER_BRANCH" `
  --reviewer-id "REVIEWER_ID" `
  --reviewer-public-key-base64 "RAW_32_BYTE_PUBLIC_KEY_BASE64" `
  --attest-independent-repository-control `
  --attest-private-key-external `
  --attest-producer-write-access-absent `
  --attest-protected-create-only-branch
```

The report must say
`state=OWNER_INPUTS_READY_FOR_TWO_PARTY_REVIEW` and
`ready_for_inert_child_lock_proposal=true`. It will still say
`independence_verified=false`, `authority_activated=false`, and every
operational permission remains false. Those are deliberate boundaries, not
missing configuration.

## 3. Verify the public key

From the repository root, derive and verify the fingerprint without writing an
artifact:

```powershell
.\.venv\Scripts\python.exe scripts\preflight_v14_external_finality_public_key.py `
  --public-key-base64 "RAW_32_BYTE_PUBLIC_KEY_BASE64"
```

The command must report `public_key_bytes=32` and `verified=true`. Record the
lowercase 64-character `reviewer_public_key_sha256` value. To check a proposed
fingerprint explicitly, repeat with:

```powershell
.\.venv\Scripts\python.exe scripts\preflight_v14_external_finality_public_key.py `
  --public-key-base64 "RAW_32_BYTE_PUBLIC_KEY_BASE64" `
  --expected-sha256 "REVIEWER_PUBLIC_KEY_SHA256"
```

This hashes the raw 32 public-key bytes, not PEM or OpenSSH text.

## 4. Create the inert child-lock proposal

Choose a fresh canonical UTC timestamp and create the proposal once:

```powershell
$CreatedUtc = Get-Date -AsUTC -Format "yyyy-MM-ddTHH:mm:ssZ"
.\.venv\Scripts\python.exe scripts\create_v14_external_finality_active_authority_lock.py `
  --output artifacts\v14_external_finality_active_authority_lock.json `
  --repository "OWNER/REPOSITORY" `
  --branch "REVIEWER_BRANCH" `
  --reviewer-id "REVIEWER_ID" `
  --reviewer-public-key-sha256 "REVIEWER_PUBLIC_KEY_SHA256" `
  --created-utc $CreatedUtc
```

The output is create-only. Do not delete and recreate it to obtain different
identities. Record both printed values:

```text
artifact_sha256=...
raw_file_sha256=...
```

The command must also report `operational_step4_eligible=false`.

## 5. Independent review and pinning

Submit the exact new artifact, these two hashes, the repository/branch/reviewer
identity, and the public-key fingerprint for two-party review. The reviewer
must confirm that:

1. the repository and branch are independently controlled and usable;
2. the fingerprint matches the retained raw 32-byte public key;
3. the artifact is canonical and binds the intended pending-lock identities;
4. every operational permission remains false;
5. the exact artifact and raw hashes are accepted.

Only after that review, make a separate reviewed code change setting these two
constants in `kraken_guardian/v14_external_finality_active_authority.py`:

```python
V14_EXTERNAL_FINALITY_ACTIVE_AUTHORITY_ARTIFACT_SHA256 = "..."
V14_EXTERNAL_FINALITY_ACTIVE_AUTHORITY_RAW_SHA256 = "..."
```

Do not add a private key or token.

## 6. Verify readiness

Run the focused contract suite:

```powershell
.\.venv\Scripts\python.exe -m pytest -q `
  tests\test_v14_external_finality_active_authority.py `
  tests\test_v14_external_finality_claim.py `
  tests\test_v14_external_finality_countersignature.py `
  tests\test_research_status.py
```

Then start the app and inspect `/api/research/status`. The child can be treated
as provisioned only when the active authority and independent countersignature
are verified. Even then, `endpoint_contact_allowed`, `operational_step4_eligible`,
`outcome_access_allowed`, and `execution_allowed` remain false until later,
separately reviewed protocol steps explicitly change them.
