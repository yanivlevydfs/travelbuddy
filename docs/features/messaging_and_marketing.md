# Guest Messaging and Hotel Marketing

## Guest messaging channels

The HostOS messaging runtime supports WhatsApp Cloud API, Instagram messaging,
and web chat, with internal/phone/email channel types used by the shared
runtime. Channel capabilities differ: WhatsApp supports templates, Instagram
does not, and web chat replies through its HTTP response.

HostOS routes include:

- Signed WhatsApp and Instagram webhook endpoints.
- Instagram connect/disconnect and OAuth callback flows.
- WhatsApp onboarding, connection, claim, test-message, and template operations.
- Property conversation, session, message, and handler-management APIs.
- Public hotel web chat.

Routes are under `/api/v1/messaging/*` and `/api/v1/pms/*`, and are mounted only
when HostOS is enabled.

## AI employee runtime

The AI runtime is tool-based. It routes a turn to a role-specific employee,
selects an appropriate model, assembles property/session context, validates
tool inputs, checks permissions and business rules, and records the result.
Tools are grouped by tasks such as booking, communication, front desk, hotel
operations, marketing, pricing, sales, and stay management.

Important modules include `app/services/ai/runtime.py`,
`app/services/ai/registry.py`, `app/services/ai/permissions.py`,
`app/services/ai/autonomy.py`, and `app/services/ai/guardrails.py`.
The [agent architecture map](../agent_architecture_map.md) and
[agent arbitration](../agent_arbitration.md) describe the detailed runtime.

## Queue and state behavior

Inbound events are persisted and acknowledged before asynchronous processing.
Celery tasks process conversations and scheduled sweeps; Redis supports
broker/worker coordination and conversation locks. Inline processing is an
optional mode controlled by `MESSAGING_INLINE_PROCESSING`.

Per-hotel channel credentials are encrypted at rest. Configure a valid
`MESSAGING_CREDENTIALS_KEY` before allowing channel connections. Webhook
signatures and channel-specific policies are enforced by the corresponding
adapters.

## Marketing

HostOS marketing features include audience and consent management, reusable
creative, campaign execution, claims/guardrails, cancellation, and optional Meta
Ads operations. Campaign work runs through the `hotelos` task queue.

Google Business Profile routes are a separate optional feature, gated by
`ENABLE_GOOGLE_BUSINESS_PROFILE` in addition to `ENABLE_HOTEL_OS`; Google OAuth
must be configured. This integration is not mounted by default.
