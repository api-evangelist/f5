---
name: f5-do-onboard-device
description: Onboard a BIG-IP with Declarative Onboarding — capture the original configuration first, apply the declaration asynchronously, and read the task rather than trusting the response code.
api: F5 BIG-IP Declarative Onboarding
spec: openapi/f5-big-ip-declarative-onboarding-openapi.yml
version: 1.47.0
base_url: https://{bigip}/mgmt/shared/declarative-onboarding
operations:
  - getMostRecentTask
  - getConfig
  - getAllConfigs
  - getInspect
  - postDeclaration
  - getTask
  - getAllTasks
generated: '2026-09-07'
method: generated
source: Grounded in operationIds present in openapi/f5-big-ip-declarative-onboarding-openapi.yml,
  harvested verbatim from
  github.com/F5Networks/f5-declarative-onboarding/blob/main/src/schema/latest/openapi.yaml on 2026-09-07.
---

# Onboard a BIG-IP with Declarative Onboarding

DO sets up the device itself: hostname, licensing, DNS and NTP, VLANs, self-IPs, routes, user
accounts, provisioning of modules. It is the layer beneath AS3 — DO establishes the device, AS3
then configures application services on it. Run DO first or AS3 has nothing to configure.

**This is the highest-consequence write surface in the F5 estate.** A DO declaration can change
management addressing, re-provision modules and restart services. It can cut you off from the
device you are configuring. Nothing here should run unattended.

## Authenticate

HTTP Basic over TLS against a BIG-IP administrative user, or `X-F5-Auth-Token` from
`POST /mgmt/shared/authn/login`. The declared server in the spec is `192.0.2.1` — the RFC 5737
documentation address, a placeholder for the caller's own device.

## Steps

### 1. Check DO's state

    GET /                                    → getMostRecentTask

Returns the status of the most recently deployed configuration request. If something is already
running, wait.

### 2. Capture the current configuration — both of them

    GET /inspect                             → getInspect
    GET /config                              → getAllConfigs
    GET /config/{machineId}                  → getConfig

These are different things and you want both.

- `getInspect` returns the device's **current** configuration.
- `getConfig` returns the device's stored **original** pre-onboarding configuration, which DO
  captured the first time it ran.

Store both responses off-device. DO declares no operation that re-applies the original
configuration — it will hand it to you, but it will not put it back for you. Your restore path is
manual, and it starts with having the document.

### 3. Apply the declaration

    POST /                                   → postDeclaration

There is no dry-run mode on DO. There is no `controls.dryRun` equivalent. Whatever validation you
want to do, you do before you send.

Prefer async operation and expect the connection to be disturbed: if your declaration touches
management addressing or provisioning, the device may stop answering on the address you called it
on. Plan the out-of-band path before you POST.

### 4. Read the task, not the status code

    GET /task                                → getAllTasks
    GET /task/{taskId}                       → getTask

**A 200 from `POST /` does not mean the declaration succeeded.** DO reports the real outcome as
`result.code` / `result.status` / `result.message` on the task. Poll until terminal and branch on
the task, not on the HTTP status.

### 5. Verify

    GET /inspect                             → getInspect

Diff against what you intended.

## Do not call this

    DELETE /config/{machineId}               → deleteConfig

This destroys DO's stored copy of the device's **original** configuration. F5 describes it as a
recovery action for when DO "has gotten into an unusable state". It is irreversible, and after it
runs, the pre-onboarding baseline from step 2 no longer exists on the device. If you did not take
your own copy, it is gone.

## Errors

| Status | What it means |
|---|---|
| 404 | No task or config with that ID |
| 422 | Declaration failed validation |
| 500 | Processing error — read `result.message` on the task |

DO declares no error schema. See `errors/f5-problem-types.yml`.

## Sequencing with AS3

    DO  →  device identity, licensing, VLANs, self-IPs, provisioning
    AS3 →  virtual servers, pools, monitors, profiles, WAF policies

That ordering is the whole relationship between the two APIs. There is no field linking a DO
declaration to an AS3 declaration; the dependency is operational. See
`skills/f5-as3-deploy-application.md` for the next step.
