---
name: f5-nginx-scale-upstream
description: Add, drain and remove servers in an NGINX Plus upstream group at runtime, without a configuration reload — the safe scale-out and scale-in flow.
api: NGINX Plus REST API
spec: openapi/f5-nginx-plus-api-openapi.yml
version: '9.0'
base_url: https://{nginx-host}/api/9
operations:
  - getAPIEndpoints
  - getHttpUpstreams
  - getHttpUpstreamName
  - getHttpUpstreamServers
  - postHttpUpstreamServer
  - patchHttpUpstreamPeer
  - deleteHttpUpstreamServer
generated: '2026-09-07'
method: generated
source: Grounded in operationIds present in openapi/f5-nginx-plus-api-openapi.yml, harvested verbatim
  from https://demo.nginx.com/swagger-ui/nginx_api.yaml on 2026-09-07.
---

# Scale an NGINX Plus upstream at runtime

NGINX Plus lets you change the membership of an upstream group through the API while traffic is
flowing. No reload, no dropped connections. This is the flow for adding capacity and for taking it
away again cleanly.

## Before you start

Two preconditions decide whether any of this will work, and both fail loudly.

1. **The API must be in write mode.** The `api` directive in the nginx.conf location serving this
   endpoint must be `api write=on;`. Without it every POST, PATCH and DELETE below returns
   **405** with internal error code `MethodDisabled`. This is a deployment decision the operator
   made, not something you can change through the API.
2. **The upstream must be dynamically configurable.** It needs a `zone` directive. A statically
   configured upstream returns **400** `UpstreamStatic` on any membership change.

There is no authentication in this contract — access control lives in the nginx.conf location
(`allow`/`deny`, `auth_basic`, client certificates). Whatever the operator put there is what you
must satisfy.

## Steps

### 1. Confirm the API version

    GET /api/            → getAPIEndpoints

Returns the array of versions this build supports. Use the highest one it lists. Calling an
unsupported `/api/{n}` returns **404** `UnknownVersion`, which looks like a missing resource and
is not.

### 2. Find the upstream

    GET /api/9/http/upstreams/                    → getHttpUpstreams
    GET /api/9/http/upstreams/{name}/             → getHttpUpstreamName

`getHttpUpstreams` returns a map keyed by upstream name, not an array. There is no pagination —
you get all of them.

### 3. Read the current members before you change anything

    GET /api/9/http/upstreams/{name}/servers/     → getHttpUpstreamServers

Keep this response. It is your rollback material: there is no version history on this API, so the
only record of the previous membership is the one you just took.

**The `id` field is not stable.** NGINX assigns peer IDs, and they are reassigned on a
configuration reload. Never cache an ID across a deploy — re-read the collection and match on
`server` (the address) instead.

### 4. Add the new server

    POST /api/9/http/upstreams/{name}/servers/    → postHttpUpstreamServer

Body carries at least `server` (address), plus optional `weight`, `max_conns`, `max_fails`,
`fail_timeout`, `slow_start`, `backup`, `down`.

This operation is **not idempotent**. A replay returns **409** `EntryExists` rather than
converging. If a POST times out, re-read `getHttpUpstreamServers` and check before retrying —
do not blind-retry.

Add it `down: true` first if you want to health-check it before it takes traffic, then PATCH it
up. Or set `slow_start` so NGINX ramps it in rather than hitting it with a full share
immediately.

Common failures here: **400** `UpstreamBadAddress`, `UpstreamBadWeight`, `UpstreamBadMaxConns`,
`UpstreamBadMaxFails`, `UpstreamBadFailTimeout`, `UpstreamBadSlowStart`,
`UpstreamConfFormatError` (unknown parameter or missing `server`), `UpstreamConfNoResolver` (you
gave a hostname and the upstream block has no `resolver`), and `UpstreamOutOfMemory` (the
upstream's shared memory zone is full — see below).

### 5. Verify

    GET /api/9/http/upstreams/{name}/             → getHttpUpstreamName

Check the new peer appears and its `state` becomes `up`. Watch `health_checks.unhealthy` and
`fails` before declaring success.

## Taking a server out

Drain first, then remove. Deleting a live peer drops its in-flight connections.

### 1. Drain it

    PATCH /api/9/http/upstreams/{name}/servers/{id}    → patchHttpUpstreamPeer

Body `{"drain": true}`. Existing sessions finish; no new ones are assigned.

### 2. Wait for connections to fall

    GET /api/9/http/upstreams/{name}/servers/{id}      → getHttpUpstreamPeer

Poll `active` until it reaches zero, or until your drain deadline expires.

### 3. Remove it

    DELETE /api/9/http/upstreams/{name}/servers/{id}   → deleteHttpUpstreamServer

**400** `UpstreamServerImmutable` means the server was defined statically in the configuration
file and cannot be removed through the API — that one needs a config change and a reload.
**400** `UpstreamServerWeightImmutable` means the peer came from SRV resolution and its weight
comes from DNS.

## What you cannot undo

Runtime membership changes live in memory. Unless the upstream is backed by a `state` file,
everything you did here is lost on an NGINX restart. That is not a rollback mechanism — it is a
different failure mode, and it can silently revert your change hours later. Confirm whether the
upstream has a `state` file before you treat an API change as durable.

Statistics resets (`DELETE` on any of the zone endpoints) have no undo at all. Do not call them
as a diagnostic step.

## Shared memory

NGINX Plus R37 added roughly 1KB of shared memory per upstream server for response-time
histograms. Upstream `zone` allocations sized before R37 may need a 25-30% increase. When you hit
the ceiling, adding a server fails with **400** `UpstreamOutOfMemory` — a size problem wearing a
validation error's clothes.
