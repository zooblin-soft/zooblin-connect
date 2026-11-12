# Zooblin Connect

Zooblin Connect is Zooblin's white-label customer service platform for its clients. It starts from Chatwoot Community Edition and adds Zooblin branding, client provisioning, and an independently developed voice integration. The source fork remains attributed to Chatwoot in `LICENSE`.

## Product scope

- Shared agent inbox for website chat, email, WhatsApp, and other channels supported by Community Edition.
- Customer profile and conversation history linked to a Zooblin tenant and, where authorized, a Zooblin customer ID.
- Inbound and outbound voice calls, queueing, assignment, dispositions, call notes, and operational reporting. These are product requirements; they are **not implemented by this fork yet**.
- A branded agent workspace and customer-facing channels for each white-label client.

## Client isolation

Start with a separate Zooblin Connect deployment per client. Chatwoot's installation name, logo, and brand links are installation-wide, so one shared deployment cannot give each client a complete independent brand. Separate deployments also provide clear database, storage, secret, backup, and webhook boundaries. Each deployment may have multiple Chatwoot accounts for that client's internal teams.

Provisioning should create a dedicated hostname, PostgreSQL database, Redis namespace or instance, object storage prefix or bucket, encryption secrets, and backup policy. Zooblin's control plane should map its tenant ID to the deployment and never accept an account ID supplied by an untrusted client as proof of tenant ownership.

## Voice architecture

The voice provider has not been chosen. Keep its SDK, credentials, numbers, call control, and audio media in a separate voice adapter. The Chatwoot API channel can create contacts and conversations; account webhooks can reflect agent activity. Use these interfaces for the call timeline, assignment, notes, and final disposition. A call record should include provider call ID, tenant ID, Chatwoot account and conversation IDs, direction, status, timestamps, assigned agent, and optional recording reference.

The first implementation should cover incoming call routing, agent accept/decline, outgoing calls, missed-call callbacks, and call completion. Add recording only after the client has defined consent, retention, access, and deletion rules. Verify provider webhooks, enforce idempotency by provider event ID, and scope every API token and phone number to one client deployment. The browser calling UI will need the chosen provider's media SDK or a separate softphone; Chatwoot's message API does not carry live audio.

## Community Edition build

The product Docker image must be built from this repository with `docker/Dockerfile`. `.dockerignore` excludes `enterprise/` and `spec/enterprise/` from the build context, and the runtime sets `CW_EDITION=ce` and `DISABLE_ENTERPRISE=true`. Do not deploy the upstream `chatwoot/chatwoot:latest` image as Zooblin Connect; it would not contain Zooblin changes. Keep the Chatwoot MIT copyright and license notice in source distributions, and review third-party dependencies before distributing an image.

The repository is an upstream fork, so the Git history contains Chatwoot's separately licensed `enterprise/` tree. That tree is not part of the Zooblin Connect product build. Do not copy, modify, enable, or market Enterprise features without a separate license. In particular, the proposed voice adapter must be an independent implementation.

## Release gates

1. Set Zooblin Connect's production hostname, mail sender, storage, database, Redis, and secrets.
2. Set client-specific legal terms and privacy URLs; the upstream defaults in `config/installation_config.yml` are placeholders and must not be published for a client.
3. Verify the Community Edition image contains no `enterprise/` files and that the active installation name and logos are the intended client brand.
4. Test tenant separation, inbound and outbound calls, webhook retries, agent handoff, call history, and backup restore in staging.
5. Review call recording, retention, and telecom requirements in each operating market before launch.

## Upstream and licensing

- Upstream source: https://github.com/chatwoot/chatwoot
- Community license: `LICENSE`
- Enterprise license: `enterprise/LICENSE` (source reference only; excluded from the product image)
- Chatwoot API channel: https://www.chatwoot.com/hc/user-guide/articles/1677839703-how-to-create-an-api-channel-inbox
- Chatwoot webhooks: https://www.chatwoot.com/hc/user-guide/articles/1677693021-how-to-use-webhooks
