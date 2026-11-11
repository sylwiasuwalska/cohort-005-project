# Which hosted vendor fits Cadence's in-app Notifications?

Research for [issue #2](https://github.com/sylwiasuwalska/cohort-005-project/issues/2), part of the [In-app Notifications map](https://github.com/sylwiasuwalska/cohort-005-project/issues/1). Sources were read on 2026-11-11. Pricing and roadmaps change, so re-check the linked pages before you sign up.

Cadence context: React 19.2 + React Router 8 (server actions run in Node), SQLite via Drizzle, its own session-based users switched through the DevUI panel. Scope is in-app only, with email possible later.

## Recommendation

**Novu Cloud (EU region).** It is the only vendor that meets every criterion on its free plan: 10,000 workflow runs a month, EU data residency in Frankfurt, a prebuilt React `<Inbox />` plus headless hooks, a Node SDK, real-time sockets, HMAC-based user identification with no vendor auth, and email in the same workflows. The first paid tier starts at $30/month.

**Runners-up**

1. **Knock**: the most polished in-app feed and docs, with the same 10k free messages. But EU data residency is only on the roadmap ("by the end of 2026"), and the first paid tier is $250/month.
2. **SuprSend**: an EU workspace region, 10k free notifications, JWT user tokens and a $100/month first paid tier. Docs are thinner, and EU availability per plan is unclear.

**Not shortlisted**: Courier, because EU residency needs the Business plan and feed tokens need a call to Courier's API. MagicBell, because its free tier is 1,000 deliveries and it documents no EU region.

## Shortlist

Starting candidates were Knock, Novu, MagicBell and SuprSend. Courier was added because it ships a first-class in-app Inbox and publishes a free tier. Every vendor here has a hosted in-app feed as a product, not only as a channel adapter.

## Comparison

| Criterion | Novu | Knock | SuprSend | Courier | MagicBell |
|---|---|---|---|---|---|
| Free tier | 10k workflow runs/mo, all channels [1] | 10k messages/mo [6] | 10k notifications/mo, feed carries SuprSend branding [11] | 10k sends/mo [15] | 1k deliveries/mo [19] |
| First paid tier | Pro from $30/mo, 30k runs, $1.20/1k extra [1] | Starter $250/mo, 50k msgs, $0.005 extra [6] | Essentials $100/mo, 50k, $2/1k extra [11] | Business $0.005/send, pay as you go [15] | Startup $249/mo, 50k, $0.0025 extra [19] |
| React feed component | `@novu/react` `<Inbox />` (bell, feed, preferences), React 18+/19 [2] | `@knocklabs/react` `NotificationFeedPopover`, `NotificationIconButton` [7] | `@suprsend/react`, drop-in and headless [12] | `@trycourier/courier-react` v9, React 18+ [16] | `@magicbell/react` `Inbox`, `FloatingInbox` [20] |
| Headless option | Hooks in `@novu/react`, `@novu/js` [2] | `@knocklabs/client`, `useNotifications` hook [7] (full headless API not checked) | Yes, headless mode in `@suprsend/react` [12] | Custom views, low-level APIs [16] | Not documented on the React page [20] |
| Node SDK | `@novu/api`, `novu.trigger({ workflowId, to, payload })` [3] | `@knocklabs/node`, trigger with `recipients` + `data` [8] | `@suprsend/node-sdk` [13] | Yes (not checked in detail) | `magicbell-js/project-client`, `broadcasts.createBroadcast` [21] |
| Real-time | Socket (`wss://eu.socket.novu.co`) [4][5] | Socket connections with reconnect [7] | **Unverified** | "Messages arrive in real time" [16]; transport not named | Ably appears in the API (`connection_id`) [22] |
| Identify users without vendor auth | HMAC-SHA256 of `subscriberId` with secret key, computed on your server [4] | RS256 JWT signed locally, `signUserToken` in Node SDK ("enhanced security mode") [9] | ES256 JWT signed locally with a signing key [14] | Backend calls Courier `/auth/issue-token` (network call per token) [17] | HS256 JWT signed locally with the project secret [23] |
| EU data residency | Yes, Frankfurt, **all plans**, including Free [1][5] | **No**: "We expect to support EU data residency by the end of 2026" [10] | EU workspace region exists [13]; plan availability **unverified** (pricing lists custom regions under Enterprise [11]) | Business and Enterprise only [18] | **None documented** [22] |
| Room for email later | Email on Free, same workflow [1] | Email channel; each channel message billed [6] | Unlimited channels on Free [11] | Email core product [15] | Email channel, each delivery billed [19] |

## Notes per vendor

### Novu (recommended)

- **Pricing**: billing counts workflow runs, not per-channel messages. Adding an email step to a workflow later should therefore not double the bill the way per-message pricing does [1]. Free limits: 5 workflows, 2 environments, 3 team members and 24-hour activity-feed retention [1]. Five workflows may get tight as the number of Notification kinds grows. Pro removes the limits and the Novu branding.
- **Security**: turn on "Security HMAC encryption" for the In-App integration in production. The server computes `subscriberHash = HMAC-SHA256(NOVU_SECRET_KEY, subscriberId)`. With HMAC on, the Inbox will not load without the hash [4]. That maps directly to a Cadence user id plus a hash computed in a loader.
- **EU**: the API key is bound to the region it was created in, and data cannot be moved between regions later [5]. Create the account in the EU from day one. Set `serverURL: "https://eu.api.novu.co"` in `@novu/api` [3], and set `apiUrl` / `socketUrl` on `<Inbox />` [4].

### Knock (runner-up 1)

- Feeds are on all plans, with unlimited retention. The Free plan shows Knock branding [6].
- A message is "a single message delivered to a single user on a single channel". In-app plus email is two messages [6].
- Enhanced security mode signs a JWT with your private signing key (`KNOCK_SIGNING_KEY`). No call to Knock is needed [9].
- The deal-breaker today is EU residency, which is roadmap only [10]. If EU residency isn't a hard requirement, Knock is a close call with Novu on developer experience.

### SuprSend (runner-up 2)

- Inbox auth moved from HMAC to signed JWT user tokens (ES256) [12][14].
- "Each workspace has a data region, including the EU" [13]. The pricing page only lists "Custom Data Center Region" under Enterprise [11], so it is not clear whether a Free workspace can be in the EU.
- Batched or digested messages count as one notification. Messages blocked by opt-out are not billed [11].

### Courier

- Feed JWTs are minted by calling `https://api.eu.courier.com/auth/issue-token` with the API key, which adds a network hop to every token [17].
- The EU datacenter is on Business and Enterprise only [18]. A Courier blog post says Enterprise only, so its sources disagree.

### MagicBell

- The free tier is 1,000 deliveries a month, and every channel counts separately [19].
- No EU host is documented. The API base URL is `api.magicbell.com/v2` [22].

## Unverified or open

- **SuprSend**: whether the EU region is available on Free or Essentials; the real-time transport.
- **Novu**: Inbox notification retention on Free (only the 24h *activity feed* retention is stated); whether the 5-workflow cap fits Cadence's list of Notification kinds.
- **Knock**: whether EU residency shipped after the docs were read; the full headless client surface.
- **Courier**: Node SDK details were not checked.
- **MagicBell**: real-time transport (Ably is only inferred from an API field); whether a headless React option exists.
- **All vendors**: pricing was read from marketing pages, not contracts.

## Sources

1. Novu pricing: https://novu.co/pricing
2. Novu Inbox overview: https://docs.novu.co/platform/inbox/overview
3. Novu TypeScript server SDK: https://docs.novu.co/platform/sdks/server/typescript
4. Novu Inbox, prepare for production (HMAC, EU URLs): https://docs.novu.co/platform/inbox/prepare-for-production
5. Novu security and data residency: https://docs.novu.co/platform/additional-resources/security
6. Knock pricing: https://knock.app/pricing
7. Knock React feed: https://docs.knock.app/in-app-ui/react/feed
8. Knock Node SDK: https://github.com/knocklabs/knock-node
9. Knock in-app security and authentication: https://docs.knock.app/in-app-ui/security-and-authentication
10. Knock plans (EU residency roadmap): https://docs.knock.app/manage-your-account/knock-plans
11. SuprSend pricing: https://www.suprsend.com/pricing
12. SuprSend React Inbox: https://docs.suprsend.com/docs/inbox-react
13. SuprSend docs home (regions, SDKs): https://docs.suprsend.com/
14. SuprSend client authentication: https://docs.suprsend.com/docs/client-authentication
15. Courier pricing: https://www.courier.com/pricing
16. Courier Inbox overview: https://www.courier.com/docs/platform/inbox/inbox-overview
17. Courier Inbox authentication: https://www.courier.com/docs/platform/inbox/authentication
18. Courier datacenters: https://www.courier.com/docs/workspaces/datacenters
19. MagicBell pricing: https://www.magicbell.com/pricing
20. MagicBell React library: https://www.magicbell.com/docs/libraries/magicbell-react
21. MagicBell JS library: https://www.magicbell.com/docs/libraries/magicbell-js
22. MagicBell API reference: https://www.magicbell.com/docs/api/reference
23. MagicBell user JWT authentication: https://www.magicbell.com/docs/api/authentication/user
