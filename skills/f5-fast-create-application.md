---
name: f5-fast-create-application
description: Create a BIG-IP application from a FAST template — discover templates, read their parameter schema, render the AS3 declaration without deploying it, then deploy and track the task.
api: F5 BIG-IP FAST
spec: openapi/f5-big-ip-fast-openapi.yml
version: 1.26.0
base_url: https://{bigip}/mgmt/shared/fast
operations:
  - getFASTInfo
  - getFASTTemplateSets
  - getFASTTemplates
  - getFASTTemplateBySetAndTemplateName
  - postFASTRender
  - postFASTApplications
  - getFASTTasks
  - getFASTTaskById
  - getFASTApplication
  - deleteFASTApplication
generated: '2026-09-07'
method: generated
source: Grounded in operationIds present in openapi/f5-big-ip-fast-openapi.yml, harvested verbatim
  from github.com/F5Networks/f5-appsvcs-templates/blob/master/docs/openapi.yml on 2026-09-07.
---

# Create a BIG-IP application from a FAST template

FAST is the guardrail layer over AS3. Instead of authoring a full AS3 declaration, you pick a
template, supply parameters, and FAST renders and deploys the declaration for you. For an agent
this is the better entry point: the template's parameter schema is a real, machine-readable input
contract, which raw AS3 does not give you.

## Authenticate

HTTP Basic over TLS against a BIG-IP administrative user, or the `X-F5-Auth-Token` header from
`POST /mgmt/shared/authn/login`. The base URL is the caller's own BIG-IP — note that the published
spec declares `http://localhost:8100/mgmt/shared/fast`, which is the restnoded loopback used during
development, not a host you can reach.

## Steps

### 1. Confirm FAST is installed and current

    GET /info                                          → getFASTInfo

### 2. Find a template

    GET /templatesets                                  → getFASTTemplateSets
    GET /templates                                     → getFASTTemplates

Templates are addressed as `{setName}/{templateName}`. The bundled sets cover the common shapes
(HTTP, HTTPS, TCP, UDP, DNS, microservices).

### 3. Read the template's parameter schema

    GET /templates/{setName}/{templateName}            → getFASTTemplateBySetAndTemplateName

This returns the template's parameter definition. **Treat it as the input contract** — it tells you
which parameters exist, which are required, and what types and enums they accept. Validate your
parameters against it before rendering. This is the step that makes FAST safe to drive
programmatically.

### 4. Render without deploying

    POST /render                                       → postFASTRender

Send the template name and your parameters. FAST returns the AS3 declaration it *would* deploy,
and deploys nothing. This is FAST's rehearsal step and the equivalent of AS3's
`controls.dryRun` — do not skip it.

Read the returned declaration. It tells you exactly what will be created on the device, including
anything the template defaults in that you did not ask for.

### 5. Deploy

    POST /applications                                 → postFASTApplications

Returns a task ID (a UUID — FAST is the one F5 API that uses opaque identifiers; everything else
addresses objects by configuration name).

### 6. Track the task

    GET /tasks/{taskId}                                → getFASTTaskById
    GET /tasks                                         → getFASTTasks

Poll until terminal. The POST returning cleanly is not the deployment succeeding: FAST hands the
rendered declaration to AS3, and the AS3 outcome surfaces on the task.

### 7. Verify

    GET /applications/{tenantName}/{appName}           → getFASTApplication
    GET /applications                                  → getFASTApplications

## Changing an application

    PATCH /applications/{tenantName}/{appName}         → updateFASTApplication
    PUT  /applications                                 → putFASTApplications

Re-render first (step 4) so you can see the resulting declaration before you commit to it.

## Removing an application

    DELETE /applications/{tenantName}/{appName}        → deleteFASTApplication
    DELETE /applications                               → deleteFASTApplications

`deleteFASTApplications` removes **all** FAST-managed applications. Be certain which of the two you
are calling.

## Undo

FAST keeps no version history. Deletion is the only reversal it offers. Because FAST deploys
through AS3, the practical undo for a FAST-created application is at the AS3 layer — retrieve a
prior declaration with `GET /declare?age={n}` on `/mgmt/shared/appsvcs` and re-deploy it. See
`skills/f5-as3-deploy-application.md`. AS3's default retention is only 4 declarations, so capture
your own baseline before you start.

## Errors

FAST declares 400, 404, 422 and 500 with `FastResponse*` schemas but no enumerated error-code
vocabulary — you get a status and a message. A **422** on deploy is usually a parameter that
passed the template schema but produced an AS3 declaration AS3 rejected; the rendered output from
step 4 is where to look. See `errors/f5-problem-types.yml`.

## Template sets

    POST /templatesets                                 → postFASTTemplateSets
    POST /offbox-templatesets                          → postFASTOffboxTemplates
    DELETE /templatesets/{setName}                     → deleteFASTTemplateSetByName

Installing a template set changes what every other caller on this device can deploy. Treat it as a
platform change, not an application change.
