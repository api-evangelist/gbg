---
name: GBG
description: Use when building identity verification journeys, configuring
  verification modules, integrating APIs for customer onboarding, or deploying
  hosted verification workflows. Agents should reach for this skill when
  designing verification workflows, troubleshooting journey execution,
  configuring modules, managing API clients, or investigating verification
  results.
metadata:
  mintlify-proj: gbg
  version: "1.0"
---

# GBG-GO Skill Reference

## Product summary

GBG-GO is an identity verification platform that helps businesses verify customer identities, prevent fraud, and meet regulatory requirements. It provides a **Journey builder** for designing verification workflows, a library of **modules** (document verification, biometrics, compliance screening, data verification), and two delivery methods: **API-first integration** (you build the frontend) and **Hosted journeys** (platform-managed screens). Key files and concepts: journeys are published to Preview or Production environments with unique resource IDs and versions; modules are configured with outcomes that drive evaluation decisions; interactions define what data end users must provide. Primary docs: https://docs.go.gbgplc.com

## When to use

Reach for this skill when:
- **Building journeys**: Creating or modifying customer verification workflows in the Journey builder
- **Configuring modules**: Adding document verification, biometrics, compliance screening, or data verification modules to a journey
- **API integration**: Starting journeys, fetching interactions, submitting verification data, or monitoring journey progress via REST API
- **Hosted journeys**: Deploying platform-managed verification screens (low-code or no-code delivery)
- **Troubleshooting**: Debugging API errors, credential issues, module dependencies, or journey execution failures
- **Investigation**: Reviewing submitted customer data, exporting sessions, or changing verification decisions
- **Account setup**: Creating API clients, managing user roles, or configuring RBAC permissions

## Quick reference

### Journey workflow
| Step | Action | Key output |
|------|--------|-----------|
| 1. Create | Open Journey builder, name journey, select API + hosted (latest) | Draft journey |
| 2. Add modules | Browse and add modules (Document, Biometrics, Data, Compliance, etc.) | Configured modules |
| 3. Configure | Click each module, set outcomes and rules | Module outcomes |
| 4. Evaluation | Add Evaluation module, define Accept/Reject/Manual Review conditions | Decision logic |
| 5. Publish | Click "Publish to Preview" or "Publish to Production" | Resource ID + Version |
| 6. Integrate | Use resource ID + version in API calls or hosted journey links | Live journey |

### API execution flow (interaction-based)
```
1. Authenticate → Get access token (Bearer token)
2. Start journey → POST /journey/start with resourceId@version
3. Fetch interaction → GET /fetch/interaction to see required domain elements
4. Submit data → POST /submit/interaction with collected data
5. Check state → GET /journey/state/fetch to detect completion
6. Repeat steps 3-5 until journey.status = COMPLETED
```

### Key API endpoints
| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/auth/token` | POST | Generate access token (Bearer) |
| `/journey/start` | POST | Create journey instance, get instanceId |
| `/fetch/interaction` | GET | Retrieve current interaction and required domain elements |
| `/submit/interaction` | POST | Submit collected interaction data |
| `/journey/state/fetch` | GET | Poll journey status and results |

### Module categories
| Category | Examples | Use case |
|----------|----------|----------|
| Documents | Document Classification, Authentication, Extraction, Verification | Identity document checks |
| Biometrics | Liveness Verification, Facematch, Biometric Onboarding | Face and liveness checks |
| Data | Data Verification, Dual Match | Personal data validation |
| Compliance | PEPs and Sanctions | Regulatory screening |
| Digital Identities | Email Insights, Phone Insights, Secure OTV | Digital identity verification |
| Utilities | Manual Review, Webhook, User Invite | Custom workflows |

### Domain elements (common data units)
- `FullName`, `DateOfBirth`, `Address` (personal details)
- `PrimaryDocument`, `SecondaryDocument` (identity documents)
- `Selfie`, `AnchorImage` (biometric data)
- `Email`, `PhoneNumber` (contact info)

### Hosted journey delivery methods
| Method | Code required | Start point | Use case |
|--------|---------------|------------|----------|
| **Low-code** | Yes (single API call) | Your server calls `/journey/start`, receives `instanceUrl` | Custom app integration |
| **No-code** | No | GBG-GO platform sends invite link | Support agents, one-off verification |

### Regional API endpoints
- Europe: `https://eu.platform.go.gbgplc.com/v2/captain/`
- Americas: `https://us.platform.go.gbgplc.com/v2/captain/`
- Australasia: `https://au.platform.go.gbgplc.com/v2/captain/`

## Decision guidance

### When to use API-first vs Hosted journeys

| Scenario | API-first | Hosted journeys |
|----------|-----------|-----------------|
| Full control over UI needed | ✓ | ✗ |
| Minimal frontend development | ✗ | ✓ |
| Custom branding/styling | ✓ | ✓ (themes) |
| Mobile app integration | ✓ (with Bridge SDKs) | ✓ (with Bridge SDKs) |
| One-off customer verification | ✗ | ✓ (no-code) |
| High-volume integration | ✓ | ✓ (low-code) |

### When to use specific journey version vs @latest

| Scenario | Use specific version | Use @latest |
|----------|---------------------|------------|
| Production environment | ✓ (always) | ✗ |
| Testing new changes | ✗ | ✓ (acceptable) |
| Performance critical | ✓ | ✗ |
| Predictable behavior needed | ✓ | ✗ |

### Module ordering best practices

| Scenario | Order |
|----------|-------|
| Document + Biometrics | Document first → Liveness → Facematch |
| NameMatch dependency | DocumentExtraction → NameMatch |
| Compliance screening | Document verification → PEPs/Sanctions → Data verification |

## Workflow

### Create and publish a journey
1. Log in to GBG-GO platform
2. Click "Create a new journey" → Select "API + hosted (latest)"
3. Click edit icon, enter meaningful name (e.g., "KYC Verification - New Customers")
4. Click "Browse and add modules" in left sidebar
5. Search for or scroll to find modules (Document Classification, Liveness, etc.)
6. Click module → "Add to journey"
7. Click each module to configure outcomes and rules
8. Add Evaluation module for decision logic (Accept/Reject/Manual Review)
9. Click "Publish to Preview" to test
10. Click Dashboard to note Resource ID and Version
11. Test with API using Preview resource ID
12. Click "Publish to Production" when ready

### Execute a journey via API
1. Create API client: Account management → Clients tab → Create client → Copy Client ID and Secret
2. Request access token: POST `/auth/token` with client credentials
3. Start journey: POST `/journey/start` with `resourceId@version` and optional prefill data
4. Store `instanceId` from response
5. Fetch interaction: GET `/fetch/interaction?instanceId=...` to see required domain elements
6. Collect data from end user matching the schema
7. Submit interaction: POST `/submit/interaction` with collected data
8. Repeat steps 5-7 until `journey.status` = COMPLETED
9. Fetch final state: GET `/journey/state/fetch?instanceId=...` to retrieve results

### Review verification results
1. Log in to GBG-GO platform
2. Click "Investigation" in left menu
3. Verify you're on correct environment (Preview/Production tab) and region
4. Search for session by Session ID, Name, Email, or apply filters
5. Click Session ID to open details
6. Review tabs: Processing (module outcomes), User Data, Documents & Biometrics, Audit Log
7. Expand any module to see detailed checks and raw responses
8. (Optional) Change decision: Select new decision, add note, click "Change decision"
9. (Optional) Export as PDF: Click "Export" → "Export to PDF"

### Deploy a hosted journey (low-code)
1. Create and publish journey to Preview or Production
2. Note the Resource ID and Version from Dashboard
3. On your server, call POST `/journey/start` with `resourceId@version` and `page` configuration
4. Extract `instanceUrl` from response
5. Redirect end user to `instanceUrl`
6. Platform renders verification screens; end user completes verification
7. End user is redirected back to your app
8. Poll `/journey/state/fetch` to monitor progress and retrieve results

## Common gotchas

- **Using @latest in production**: Causes unpredictable delays and behavior changes. Always use specific version numbers.
- **Missing module credentials**: If journey completes with no results, modules lack credentials. Contact GBG account manager to update.
- **NameMatch without DocumentExtraction**: NameMatch requires DocumentExtraction as a dependency. Add DocumentExtraction before NameMatch.
- **Facematch requires both images**: In adaptive mode, Facematch needs both selfie and anchor image. Submitting only one causes failure.
- **Expired access tokens**: Tokens expire after a set period. Implement token refresh logic; don't hardcode tokens.
- **Instance URL expiration**: Hosted journey instance URLs are short-lived. Redirect users immediately after receiving the URL.
- **Webhook module timing**: Webhook fires before journey completes. Implement a short delay before calling `/journey/state/fetch` to ensure state is finalized.
- **Missing required fields in prefill**: If prefilling data, ensure all required fields match the schema exactly. Missing fields create pending tasks.
- **Interaction loop assumptions**: Don't assume task order. Always fetch the current interaction to discover what data is needed next.
- **Data retention policies**: Sessions older than your organization's retention period disappear from Investigation. Check settings if sessions are missing.
- **Credentials not stored in client-side code**: Never embed Client ID or Client Secret in frontend code. Always authenticate from your server.

## Verification checklist

Before submitting work with GBG-GO:

- [ ] Journey is published to Preview or Production (not in draft)
- [ ] Resource ID and Version are correct (not using @latest in production)
- [ ] All required modules have valid credentials (test with `/journey/state/fetch`)
- [ ] Module dependencies are satisfied (e.g., DocumentExtraction before NameMatch)
- [ ] Evaluation rules are configured (Accept/Reject/Manual Review paths defined)
- [ ] API client exists and has correct roles/permissions
- [ ] Access token is being refreshed before expiration
- [ ] Interaction data matches the schema (validate locally before submitting)
- [ ] Error handling includes X-Request-ID for support troubleshooting
- [ ] Hosted journey instance URL is used immediately (not stored for later)
- [ ] Investigation portal shows expected sessions in correct environment/region
- [ ] Decision changes are logged with notes for audit trail

## Resources

**Comprehensive navigation**: https://docs.go.gbgplc.com/llms.txt

**Critical documentation pages**:
1. [Journey builder overview](https://docs.go.gbgplc.com/docs/go-v2/platform/journey-builder/overview) — Core workflow for creating and publishing journeys
2. [API-first integration overview](https://docs.go.gbgplc.com/docs/go-v2/developer-integration/overview) — Complete API execution flow and SDK options
3. [Hosted journeys overview](https://docs.go.gbgplc.com/docs/go-v2/low-code-builder/introduction/hosted-journey) — Low-code and no-code delivery methods

---

> For additional documentation and navigation, see: https://docs.go.gbgplc.com/llms.txt