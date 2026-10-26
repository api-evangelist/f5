# F5 (f5)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

F5 is a global company that specializes in application delivery and security solutions for businesses. They provide products and services that help organizations efficiently and securely deliver applications to users across any network or cloud environment. F5's solutions are designed to optimize the performance, availability, and security of applications, ensuring a seamless user experience. The F5 portfolio includes BIG-IP for high-performance app and API delivery, NGINX for cloud-native delivery and security across modern apps and Kubernetes, and F5 Distributed Cloud Services for SaaS-based delivery, security, and networking across multicloud environments.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/f5/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/f5/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Consumer
- **Access:** 3rd-Party

## Tags

- Applications
- Security
- Load Balancing
- API Gateway
- WAF

## Timestamps

- **Created:** 2025-02-12
- **Modified:** 2026-04-28

## APIs

### F5 BIG-IP

BIG-IP delivers high-performance application and API delivery with load balancing, API gateway, DNS, and security services. The platform exposes programmable APIs including iControlREST for configuration and service management, iControl SOAP for legacy automation, iRules and iRulesLX for data-plane traffic manipulation, and TMSH scripting for control-plane automation. BIG-IQ provides centralized management for BIG-IP devices through its own well-documented REST API.

- **Human URL:** [https://clouddocs.f5.com/api/](https://clouddocs.f5.com/api/)

#### Tags

- Applications
- Security
- Load Balancing
- BIG-IP

#### Properties

- [Documentation](https://clouddocs.f5.com/api/)
- [i Control R E S T](https://clouddocs.f5.com/api/icontrol-rest/)
- [B I G- I Q  Management  A P I](https://clouddocs.f5.com/products/big-iq/mgmt-api/v0.0/)
- [i Rules](https://clouddocs.f5.com/api/irules/)
- [i Apps](https://clouddocs.f5.com/api/iapps/)
- [Postman Collection](collections/f5.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/f5.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### F5 NGINX

NGINX provides cloud-native delivery and security for modern applications, Kubernetes, and APIs. The NGINX product line includes NGINX Plus, NGINX One, NGINX Ingress Controller, and NGINX App Protect, with management and automation capabilities exposed through REST APIs for instance management, configuration, and observability across distributed deployments.

- **Human URL:** [https://docs.nginx.com/](https://docs.nginx.com/)

#### Tags

- Applications
- Kubernetes
- Cloud Native
- NGINX

#### Properties

- [Documentation](https://docs.nginx.com/)
- [N G I N X  Plus  A P I](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/)
- [N G I N X  One  Console](https://docs.nginx.com/nginx-one/)
- [N G I N X  Management  Suite](https://docs.nginx.com/nginx-management-suite/)
- [Postman Collection](collections/f5.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/f5.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### F5 Distributed Cloud Services

F5 Distributed Cloud Services delivers SaaS-based application delivery, security, and networking across multicloud and edge environments. The platform exposes a comprehensive REST API for managing tenants, virtual sites, load balancers, WAF policies, bot defense, API security, DDoS protection, and CDN configurations from a single control plane.

- **Human URL:** [https://docs.cloud.f5.com/docs-v2/api/](https://docs.cloud.f5.com/docs-v2/api/)

#### Tags

- Multicloud
- Edge
- Security
- SaaS

#### Properties

- [Documentation](https://docs.cloud.f5.com/docs-v2/api/)
- [API Reference](https://docs.cloud.f5.com/docs-v2/api-reference)
- [Getting Started](https://docs.cloud.f5.com/docs-v2/platform/concepts)
- [Postman Collection](collections/f5.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/f5.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [LinkedIn](https://www.linkedin.com/company/f5)
- [Website](https://www.f5.com)
- [Products](https://www.f5.com/products)
- [Developer  Portal](https://clouddocs.f5.com/)
- [Git Hub](https://github.com/F5Networks)
- [Blog](https://www.f5.com/company/blog)
- [Support](https://www.f5.com/services/support)
- [Community](https://community.f5.com/)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
