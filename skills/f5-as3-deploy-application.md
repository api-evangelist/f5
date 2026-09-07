---
name: f5-as3-deploy-application
description: Deploy an application delivery configuration to a BIG-IP with AS3, rehearsing it first with dry run and keeping a documented path back to the previous declaration.
api: F5 BIG-IP AS3
spec: openapi/f5-big-ip-as3-openapi.yml
version: 3.56.0
base_url: https://{bigip}/mgmt/shared/appsvcs
operations:
  - GET /info
  - GET /declare
  - POST /declare
  - GET /task
  - DELETE /declare/{tenant}/applications/{application}
generated: '2026-09-07'
method: generated
source: Grounded in the paths and parameters declared in openapi/f5-big-ip-as3-openapi.yml, harvested
  verbatim from github.com/F5Networks/f5-appsvcs-extension/blob/main/docs/openapi.yaml on 2026-09-07.
---

# Deploy an AS3 declaration safely

AS3 is declarative and convergent. You send the desired state of one or more tenants and AS3 makes
the BIG-IP match it. The consequence that catches people out: **anything not in your declaration is
removed from the tenants it covers.** A partial declaration is a deletion.

> **Note on operationIds.** The AS3 contract declares none. Every operation below is identified by
> method and path because that is all the published spec gives you. If you are generating a client,
> you will have to name them yourself.

## Authenticate

Either works against the customer's own device:

- HTTP Basic over TLS with a BIG-IP administrative user, or
- `POST /mgmt/shared/authn/login` with `{username, password, loginProviderName: "tmos"}`, then send
  the returned token in the `X-F5-Auth-Token` header. Default timeout is 1200 seconds.

There is no F5-hosted endpoint. The base URL is the caller's own BIG-IP.

## Steps

### 1. Check what AS3 you are talking to

    GET /info

Returns `version`, `release`, `schemaCurrent` and `schemaMinimum`. The path carries no version
segment, so this is the only way to know which classes and properties your declaration may use.
Do this before assuming a feature exists.

### 2. Save the current state

    GET /declare?show=base

Store the response. This is your baseline. `show=base` returns the declaration as deployed with
secrets encrypted; `show=full` populates schema defaults; `show=expanded` resolves every URL and
base64 reference to a static value and can be much larger.

Use `?filterClass={AS3Class}` to pull back only one class (once per request), and
`?age=list` to see what history AS3 is already holding.

### 3. Rehearse it

    POST /declare?controls.dryRun=true

This runs the declaration through every validation check and does **not** touch the device.
Available since AS3 3.30. It is the single most valuable thing in this API and there is no reason
to skip it.

Read the per-tenant `results[]`. A **422** means the JSON is well-formed but fails AS3 schema or
semantic validation — the message tells you which class and property.

### 4. Deploy

    POST /declare?async=true

Prefer `async=true`. A synchronous POST on a large declaration holds the connection open for
minutes, and AS3 answers **503** while another declaration is already in flight. Async returns
**202** with a request ID immediately.

Add `controls.logLevel` (RFC 5424 severities) and `controls.trace` / `controls.traceResponse` when
you need to debug. Be careful with traces: F5's own documentation warns they may contain sensitive
configuration data.

### 5. Poll to completion

    GET /task

Poll with the request ID from step 4 until the task reports a terminal state. **Do not treat the
202 as success.** The deployment result lives on the task, not on the acknowledgement.

### 6. Verify

    GET /declare?show=base

Diff against what you intended. Then check the application actually serves traffic — AS3 reporting
success means the configuration applied, not that the app is healthy.

## Rolling back

AS3 keeps a history of prior declarations and this is the documented undo:

    GET /declare?age=list        # index of retained declarations and their ages
    GET /declare?age=1           # the declaration deployed immediately before the current one
    POST /declare                # re-deploy that document

`age=0` is the most recently deployed; `age=1` through `age=15` are the prior ones. **By default
the list shows 4 declarations**, configurable up to 15 via `historyLimit` in the AS3 class.

That bound is the real constraint: if you deploy five times while chasing a problem on a
default-configured system, the good declaration you were trying to get back to is gone. Take your
own copy in step 2 and do not rely solely on AS3's history.

## Removing an application

    DELETE /declare/{tenant}/applications/{application}?controls.dryRun=true
    DELETE /declare/{tenant}/applications/{application}

`controls.dryRun` is wired to the DELETE path too, so a teardown can be rehearsed before it runs.
Use it.

## Idempotency

There is no `Idempotency-Key` header anywhere in AS3. What you get instead is convergence:
re-sending the same declaration converges to the same state rather than creating a second copy.
That covers you for a retried write on this API — but it does **not** hold for the NGINX Plus API
or for imperative iControl REST POSTs, where a replay creates or conflicts. Know which model you
are talking to before you retry.

## Errors

| Status | What it means | What to do |
|---|---|---|
| 404 | Tenant or application not found | Check the tenant/application path segments |
| 422 | Declaration failed AS3 validation | Read `results[]`; re-run with `controls.dryRun=true` |
| 500 | Processing error inside AS3 | Check `results[]` per tenant; may be partial |
| 503 | Another declaration in flight, or restnoded restarting | Back off and retry, or use `?async=true` |

AS3 declares no error schema, so you get a status code and a message string. See
`errors/f5-problem-types.yml`.
