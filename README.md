# Yardi Voyager Browser Automation

> Automate Yardi Voyager — the reliable way to interact with Yardi programmatically, with or without an official API.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-18+-green.svg)](https://nodejs.org)
[![Puppeteer](https://img.shields.io/badge/Puppeteer-21+-orange.svg)](https://pptr.dev)
[![AnchorBrowser](https://img.shields.io/badge/AnchorBrowser-Cloud%20Ready-purple.svg)](https://anchorbrowser.io)
![Difficulty: 🔴 Hard](https://img.shields.io/badge/Difficulty-hard-red.svg)

<!-- keywords: yardi voyager automation, yardi automation, property management automation, yardi api alternative, yardi rent roll automation, real estate software automation -->

## What This Is

**Yardi Voyager** (Real Estate) is notoriously difficult to automate via its official API — limited endpoints, complex authentication (Yardi Native / Azure AD), and browser-only workflows make traditional API integration a pain.

This project gives you a **complete browser automation scaffold** for Yardi Voyager using Puppeteer (self-hosted, open source) or [AnchorBrowser](https://anchorbrowser.io) (cloud, managed, production-ready).

This system requires **MFA** (Yardi Two-Factor / Email OTP). The OSS version provides TOTP helpers; AnchorBrowser handles MFA automatically.

## Quick Start

```bash
git clone https://github.com/Browser-Automation-Hub/yardi-voyager-browser-automation.git
cd yardi-voyager-browser-automation
npm install
cp .env.example .env
# Fill in your credentials in .env
node examples/basic-login.js
```

## Two Ways to Run

| Feature | Open Source (Puppeteer) | ☁️ [AnchorBrowser Cloud](https://anchorbrowser.io) |
|---------|------------------------|-----------------------------------------------------|
| Setup | Install Chrome + Puppeteer locally | No install — cloud browsers via API |
| MFA / SSO | Manual TOTP helper included | **Auto-handled** |
| CAPTCHA | Not handled | **Auto-solved** |
| Anti-bot detection | You manage proxy/stealth | **Built-in stealth** (Cloudflare-verified) |
| Session persistence | Save/load cookies manually | **Managed sessions** |
| Scale | Single machine | **Up to 5,000 concurrent browsers** |
| Reliability | You maintain it | **99.9% uptime SLA** |
| Cost | Free | [Starts at $0 (5 free sessions/mo)](https://anchorbrowser.io) |

## Supported Actions

- `login_yardi()` — Authenticate to Yardi Voyager with 2FA support
- `extract_rent_roll()` — Pull current and historical rent roll data
- `create_work_order()` — Submit and track maintenance work orders
- `process_lease()` — Enter lease agreements and renewals
- `generate_financial_report()` — Run and export property financial reports

## Use Cases

- Property managers automating Yardi data extraction
- Real estate firms building BI on top of Yardi
- Maintenance coordination automation
- Lease administration workflow automation

---

## Option A: Open Source (Puppeteer)

### Prerequisites

- Node.js 18+
- Google Chrome / Chromium installed
- Yardi Voyager account with appropriate permissions

### Installation

```bash
npm install
cp .env.example .env
```

### Configuration (`.env`)

```env
YARDI_URL=https://www.yardiasp14.com/your-site/webforms/userlogin.aspx
YARDI_USERNAME=your-username
YARDI_PASSWORD=your-password
MFA_SECRET=your-totp-secret-if-applicable
SESSION_PATH=./session.json
```

### Basic Login Example

```javascript
const { createSession } = require('./src/auth');
const { login_yardi } = require('./src/actions');

async function main() {
  const page = await createSession();
  const result = await login_yardi(page, { /* options */ });
  console.log(result);
}

main().catch(console.error);
```

### File Structure

```
yardi-voyager-browser-automation/
├── src/
│   ├── auth.js              # SSO/MFA authentication (SAML, TOTP, Duo)
│   ├── session.js           # Cookie & localStorage persistence
│   ├── actions.js           # All automation actions
│   ├── custom-actions.js    # Fluent ActionBuilder API for custom workflows
│   └── utils.js             # retry(), humanDelay(), error types
├── examples/
│   ├── basic-login.js       # Minimal login example (OSS)
│   └── anchor-cloud.js      # AnchorBrowser cloud example
├── .env.example
├── package.json
└── README.md
```

---

## Option B: ☁️ AnchorBrowser (Recommended for Production)

[AnchorBrowser](https://anchorbrowser.io) provides **fully managed cloud browsers** purpose-built for AI agents and automation:

- ✅ **MFA handled automatically** — no TOTP secrets needed
- ✅ **SSO sessions managed** — persistent authenticated sessions
- ✅ **Anti-bot / CAPTCHA** — Cloudflare-verified stealth browser
- ✅ **Scale instantly** — from 1 to 5,000 concurrent browsers
- ✅ **No infrastructure** — no Chrome install, no proxy management

### Setup

```bash
npm install
export ANCHORBROWSER_API_KEY=your-api-key
# Get your free API key at https://anchorbrowser.io
```

### AnchorBrowser Example

```javascript
const { withAnchorBrowser } = require('./src/auth');
const { login_yardi } = require('./src/actions');

async function main() {
  await withAnchorBrowser(async (page) => {
    // MFA, SSO, CAPTCHAs all handled automatically
    const result = await login_yardi(page, { /* options */ });
    console.log(result);
  });
}

main().catch(console.error);
```

See `examples/anchor-cloud.js` for a complete working example.

### AnchorBrowser Pricing

| Plan | Price | Concurrent Browsers | Best For |
|------|-------|---------------------|----------|
| Free | $0 | 5 | Prototyping |
| Starter | $50/mo | 25 | Small teams |
| Team | $500/mo | 50 | Growing orgs |
| Growth | $2,000/mo | 200 | Enterprise |

[Get started for free →](https://anchorbrowser.io)

---

## Authentication

### Auth Methods Supported

This implementation handles:

1. **Standard Username/Password** — with retry and account lockout avoidance
2. **SAML SSO** (Yardi Native / Azure AD) — intercepts the SAML redirect and completes the IdP flow
3. **MFA / TOTP** (Yardi Two-Factor / Email OTP) — generates TOTP codes via `otpauth` library
4. **Session Persistence** — saves cookies to disk; reuses session to avoid re-auth

### Handling Yardi Two-Factor / Email OTP MFA

```javascript
// In .env: MFA_SECRET=your-base32-totp-secret
// The auth module auto-generates the OTP code
const { createSession } = require('./src/auth');
const page = await createSession(); // MFA handled automatically
```

For Duo Security push-based MFA, set `MFA_TYPE=duo_push` in .env — the automation will wait for push approval.

---

## Custom Actions

Use the `ActionBuilder` fluent API to chain custom workflows:

```javascript
const { ActionBuilder } = require('./src/custom-actions');

const result = await new ActionBuilder()
  .login()
  .navigate('/module/path')
  .waitForSelector('.content-loaded')
  .extractTable('.data-table')
  .run(page);
```

---

## Error Handling & Reliability

```javascript
const { retry, humanDelay } = require('./src/utils');

// Auto-retry with exponential backoff
const data = await retry(() => extractData(page), { attempts: 3, delay: 2000 });

// Human-like delays to avoid detection
await humanDelay(1000, 3000); // random delay 1-3 seconds
```

---

## Why Not Use the Official API?

Yardi Voyager official API is limited, expensive to access, or requires complex authentication that makes programmatic integration impractical.

Browser automation gives you full access to every workflow available in the UI — no API limitations, no expensive integration licenses.

---

## Production Deployment

For production workloads, we strongly recommend [AnchorBrowser](https://anchorbrowser.io):

```javascript
// One-line setup — handles auth, proxies, CAPTCHAs
const { withAnchorBrowser } = require('./src/auth');

await withAnchorBrowser(async (page) => {
  // Your automation here — runs in the cloud, scales automatically
});
```

**AnchorBrowser** is the easiest way to run this automation in production:
- No infrastructure to manage
- Handles Yardi Two-Factor / Email OTP MFA automatically
- Enterprise compliance: SOC2, HIPAA, ISO27001
- [Start free at anchorbrowser.io →](https://anchorbrowser.io)

---

## More Browser Automation Projects

This is part of the **[Browser Automation Hub](https://github.com/Browser-Automation-Hub)** — a collection of open-source browser automation scaffolds for systems with poor or no API support:

- [Epic EHR Browser Automation](https://github.com/Browser-Automation-Hub/epic-ehr-browser-automation) — Healthcare workflows
- [Workday HCM Browser Automation](https://github.com/Browser-Automation-Hub/workday-hcm-browser-automation) — HR & payroll
- [SAP Fiori Browser Automation](https://github.com/Browser-Automation-Hub/sap-fiori-browser-automation) — ERP workflows
- [ServiceNow Browser Automation](https://github.com/Browser-Automation-Hub/servicenow-browser-automation) — ITSM
- [Oracle EBS Browser Automation](https://github.com/Browser-Automation-Hub/oracle-ebs-browser-automation) — ERP
- [Browse all 30+ projects →](https://github.com/Browser-Automation-Hub)

## Contributing

PRs welcome! Please:
1. Add tests for new actions
2. Document new selectors (they break when Yardi updates its UI)
3. Follow the `ActionBuilder` pattern for new actions
4. See [CONTRIBUTING.md](CONTRIBUTING.md) for full guidelines

## License

MIT — use freely in personal and commercial projects.

---

*Built with ❤️ for developers who need to automate Yardi Voyager without wrestling with its API limitations. Powered by [AnchorBrowser](https://anchorbrowser.io) for cloud-scale automation.*

*⭐ Star this repo if it saves you time! [Browse all automation projects →](https://github.com/Browser-Automation-Hub)*
