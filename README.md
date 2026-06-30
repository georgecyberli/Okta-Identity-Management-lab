# Okta Identity Management Lab

**Topics:** Okta, Universal Directory, AD Agent, SAML SSO, Authentication Policies, MFA, Lifecycle Management, Delegated Authentication

![Cloud](https://img.shields.io/badge/Cloud-Okta_Integrator-00297A?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Identity%20%26%20SSO-red?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Zero%20Trust-green?style=flat-square)
![Practice](https://img.shields.io/badge/Practice-Hybrid%20Identity-blue?style=flat-square)
![Output](https://img.shields.io/badge/Output-Runbooks-grey?style=flat-square)

A hands-on lab series demonstrating enterprise identity management using Okta, built as part of a self-directed sysadmin training program. Each lab simulates real-world scenarios found in healthcare and enterprise IT environments, including connecting on-premises Active Directory to a third-party cloud identity provider as an alternative to native Microsoft identity tooling.

**Environment:** Okta Integrator Free Plan | Azure Student Subscription | Hybrid AD lab, fully isolated from production

---

## Overview

| # | Lab | Skills Demonstrated |
|---|-----|----------------------|
| 1 | [Tenant Setup & Universal Directory](#lab-1--tenant-setup--universal-directory) | Org provisioning, admin console, manual user/group creation |
| 2 | [AD Agent & Directory Sync](#lab-2--ad-agent--directory-sync) | Delegated Authentication, AD-to-Okta sync, profile mastering |
| 3 | [SSO App Integration](#lab-3--sso-app-integration) | SAML 2.0, group-based assignment, end-to-end SSO testing |
| 4 | [Sign-On Policies & MFA](#lab-4--sign-on-policies--mfa) | Global Session Policy, app-level Authentication Policies, MFA enforcement |
| 5 | [Lifecycle Management](#lab-5--lifecycle-management) | Deactivation/reactivation, AD-to-Okta sync flow, access deprovisioning |

---

## Lab 1 — Tenant Setup & Universal Directory

**Objective:** Provision a long-term, portfolio-ready Okta org and establish foundational identity objects — users and groups — before connecting any external identity source.

### What I Built

- Evaluated Okta's signup tiers and selected the Integrator Free Plan over the 30-day Workforce Identity trial, prioritizing indefinite availability for ongoing lab work over short-term feature parity
- Toured the admin console structure (Directory, Applications, Security, Reports) to establish a working mental model before building anything
- Created a manual test user in Universal Directory, working through Okta's email-format username requirement
- Created two groups (`Lab-Admins`, `Lab-Users`) to support later group-based app assignment
- Reviewed the Universal Directory Profile Editor schema, the attribute map that AD sync would later populate

### Key Concepts

- Universal Directory is designed as a profile master that can aggregate identity from multiple sources (on-prem AD, HR systems, other IdPs), not just a single-source directory like a standalone AD
- Okta separates the admin console (`-admin.okta.com`) from the end-user dashboard (`.okta.com`) — a common point of confusion when first navigating the platform
- Groups created before app integration exist simplifies later assignment — apps are assigned to groups, not individual users

### Skills Demonstrated

`Okta Org Provisioning` `Universal Directory` `Admin Console Navigation` `Group-Based Access Design`

---

## Lab 2 — AD Agent & Directory Sync

**Objective:** Connect on-premises Active Directory to Okta Universal Directory, enabling synced users to authenticate using their existing AD credentials without exposing password hashes to the cloud.

### What I Built

- Installed the Okta AD Agent directly on the domain controller (DC02), authorizing it against the Okta org via an activation code exchange
- Allowed the installer to auto-provision its own managed service account (`OktaService`) rather than using a manually pre-created account — eliminating an unnecessary manual step
- Configured attribute mapping in the Directory Integration wizard, reviewing which AD attributes populate which Universal Directory fields
- Ran a full directory import, reviewed staged user matches, and confirmed three AD users into Universal Directory
- Identified that the lab sync pulled in built-in AD groups and operational accounts (DnsAdmins, ADSync groups, etc.) by default, and documented the OU-scoping approach a production deployment would use instead

### Key Concepts

- Okta does not sync password hashes by default — it uses Delegated Authentication, where login credentials are validated against AD in real time via the agent, so the password never leaves the on-premises network
- Once a user is imported from AD, AD becomes the profile master for that user; their attributes become read-only in the Okta console and must be edited at the source
- Unscoped sync pulls in every AD object the agent's service account can see — production deployments require explicit OU scoping to avoid syncing irrelevant built-in groups and service accounts

### Skills Demonstrated

`Okta AD Agent` `Delegated Authentication` `Directory Sync` `Profile Mastering` `OU Scoping Strategy`

---

## Lab 3 — SSO App Integration

**Objective:** Stand up a SAML 2.0 application integration and validate a complete end-to-end SSO login using a directory-synced AD user.

### What I Built

- Added a SAML Service Provider test application from the Okta Integration Network and configured the required Assertion Consumer Service URL and Service Provider Entity ID fields
- Assigned the application to the `Lab-Users` group rather than to individual users, then added a synced AD user to that group to inherit access
- Logged in as the AD-sourced user via the end-user dashboard using their native AD password, confirming Delegated Authentication was functioning correctly
- Clicked through the assigned application tile and validated the full SAML assertion redirect completed successfully

### Key Concepts

- Okta acts as the Identity Provider (IdP); the target application is the Service Provider (SP) — Okta generates a signed assertion that the SP trusts without requiring a second password
- The ACS URL and SP Entity ID are vendor-supplied values in a real integration; understanding what each field does is necessary even when testing with placeholder values
- Group-based app assignment means onboarding and offboarding application access is a group membership change, not a per-user, per-app configuration task

### Skills Demonstrated

`SAML 2.0` `Okta Integration Network` `Group-Based App Assignment` `End-to-End SSO Validation`

---

## Lab 4 — Sign-On Policies & MFA

**Objective:** Configure and distinguish between Okta's two independent authentication enforcement layers — org-wide session policy and per-application authentication policy — and enforce MFA at both.

### What I Built

- Reviewed the default Global Session Policy and discovered MFA was not required at that layer, despite users already being prompted for MFA at login
- Traced the actual MFA enforcement source to an app-level Authentication Policy tied to the Okta Dashboard application itself, distinct from the org-wide session setting
- Modified the Global Session Policy to require MFA at the session level
- Created a custom Authentication Policy (`Lab-MFA-Policy`) and assigned it directly to the SAML test application, demonstrating per-application policy control independent of the global default
- Confirmed Okta's built-in re-authentication requirement when modifying live security policies

### Key Concepts

- Okta enforces authentication at two independent layers: the Global Session Policy (applies to the Okta session itself) and per-app Authentication Policies (apply to launching a specific application) — either can require MFA regardless of the other's configuration
- Okta-native applications (Dashboard, Admin Console, Browser Plugin) each carry their own dedicated authentication policy separate from the org default, which explains MFA prompts that appear before any custom policy has been configured
- Modifying live authentication policies triggers a step-up re-authentication challenge for the administrator making the change, a built-in safeguard against unauthorized policy tampering

### Skills Demonstrated

`Global Session Policy` `App-Level Authentication Policy` `MFA Enforcement` `Policy Layering` `Admin Step-Up Authentication`

---

## Lab 5 — Lifecycle Management

**Objective:** Validate the complete joiner/mover/leaver workflow for an AD-sourced identity, confirming that Active Directory remains the authoritative source of truth throughout deactivation and reactivation.

### What I Built

- Attempted to suspend an AD-mastered user directly in Okta and confirmed the platform restricts this — AD-sourced users can only be deactivated through Okta, not suspended, since AD has no equivalent intermediate state
- Disabled the user's account directly in Active Directory, then triggered a directory sync and confirmed Okta automatically reflected the account as deactivated, with all application assignments removed
- Verified the access boundary by confirming the deactivated user could no longer authenticate
- Re-enabled the account in AD, then discovered that AD re-enablement alone was insufficient — Okta returned an explicit error when attempting to restore app assignments until the user was also manually reactivated within Okta itself
- Documented and executed the correct three-step reactivation sequence: re-enable in AD, reactivate in Okta, re-run directory sync to restore assignments

### Key Concepts

- For AD-mastered users, Active Directory is the authoritative source — disabling the account in AD and syncing is the correct production leaver workflow, not deactivating directly in Okta
- Deactivation (unlike suspension) removes all application assignments; reactivation does not automatically restore them and requires an explicit Okta-side reactivation step before a subsequent sync can repopulate access
- Lifecycle errors surfaced by Okta (e.g. "user is in inactive status") are diagnostic signals pointing to a missing reactivation step, not sync failures

### Skills Demonstrated

`User Lifecycle Management` `AD-to-Okta Deprovisioning` `Account Reactivation Workflow` `Access Deprovisioning Validation`

---

## Tech Stack

`Okta Integrator Free Plan` `Okta Universal Directory` `Okta AD Agent` `Active Directory` `SAML 2.0` `Delegated Authentication` `PowerShell`
