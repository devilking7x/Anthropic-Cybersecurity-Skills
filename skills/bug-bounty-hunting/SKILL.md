---
name: bug-bounty-hunting
description: >-
  Hunts vulnerabilities on authorized bug bounty programs and writes reports
  that survive triage, from the pre-flight gate — program active, fee checked,
  exact asset in scope — through a five-source duplicate check, a runnable
  proof of concept on an authorized target, honest CVSS scoring, and a
  one-finding-per-report write-up with a vendor-boundary line. Use when
  starting a hunt on HackerOne, Bugcrowd, Intigriti, or a private program,
  when proving a candidate is not a duplicate, or when answering a triage
  verdict. Keywords: HackerOne, Bugcrowd, scope, duplicate check, PoC, CVSS,
  triage, signal, disclosure. Do not use for testing a single API for
  authorization flaws — use conducting-api-security-testing.
domain: cybersecurity
subdomain: vulnerability-management
tags: [bug-bounty, HackerOne, Bugcrowd, triage, CVSS, proof-of-concept, scope-check, disclosure]
version: "1.0"
author: devilking7x
license: Apache-2.0
---

# Bug Bounty Hunting

## Overview

Bug bounty hunting is paid vulnerability research under someone else's rules. The program owner sets the scope, payout table, testing limits, and definition of a valid report — and the platform (HackerOne, Bugcrowd, Intigriti, or a private program) enforces them. Your job: find a real, exploitable weakness in an authorized asset, prove it runs, and write a report a triager can reproduce in minutes.

Three principles govern everything below:

1. **Authorization first.** No test happens until the program is active, the exact asset is in scope, and the testing method is allowed. An unauthorized test is not a finding — it is a liability.
2. **A finding without a runnable proof of concept is not a report.** Static suspicion, scanner output, and "this looks vulnerable" are drafts, not submissions.
3. **Honesty is the asset.** Inflated severity, hidden duplicates, and stretched scope burn what compounds across programs: triager trust and your signal score.

## When to Use

- Starting a hunt on a new program (HackerOne, Bugcrowd, Intigriti, YesWeHack, or a private/VDP program)
- Checking whether a program is active, what it pays, and what is in scope before testing anything
- Evaluating a candidate vulnerability: is it real, is it new, is it in scope, is it worth the time?
- Building a proof of concept for a suspected vulnerability
- Scoring a finding with CVSS and writing the report
- Responding to triage: triaged, duplicate, informative, N/A, needs more info

## Prerequisites

- An account on the bounty platform and access to the target program (public, private invite, or VDP)
- The program page itself: scope list, out-of-scope list, bounty table, testing rules, disclosure policy
- Two test accounts of your own on the target, where the finding needs a victim (attack one from the other — never touch another user's account)
- A local deployment of the target code where available, so the PoC can run against your own instance
- Burp Suite (or equivalent proxy), a browser, and the FIRST CVSS calculator for scoring
- A logbook: one entry per program recording gate results, duplicate-check receipts, PoC status, report ID, and triage state

## Workflow

### Step 1 — Pre-flight gate: program, fee, scope

Do this before reading a single line of target code — skipping it wastes days on a dead program or a report that can never be paid.

1. **Confirm the program is ACTIVE.** Check the program page itself, not a third-party listing: status (public, private, invite-only), whether submissions are open, and any banner about paused intake, ended campaigns, or migrated programs.
2. **Check fees and deposits.** Some programs require a submission fee, a stake, or a deposit — read the exact amount, currency, refundability, and what happens if the report is a duplicate. Never pay a fee or deposit you cannot afford, and never pay one the program demands without disclosing it up front. If a fee surfaces mid-hunt, stop and re-evaluate.
3. **Read the program policy end to end.** Scope, out-of-scope list, testing rules, rate limits, prohibited techniques (no automated scanning, no DoS, no social engineering, no testing other users), disclosure rules, bounty table — and any signal requirement. Some programs only accept researchers above a threshold; below it you may get a limited number of trial reports. Know your allowance before you spend one.
4. **Scope check — the exact asset must be IN the scope list.** The exact domain, subdomain, repository, app version, API endpoint, or contract address you tested must appear in the program's scope list, verbatim. A wildcard covers what it literally covers, not a sibling service that "looks related". A "closest option" in a submission form is not scope, and an "Open Scope" clause in marketing copy is not a basis to submit unless the program's own scope section says so explicitly. If the exact asset is not listed, the finding is not submittable.
5. **Record the gate.** Before testing, write down: program name and URL, status, fee/deposit (amount or "none"), signal/trial allowance, in-scope assets, and the out-of-scope list.

### Step 2 — Reconnaissance and target selection

Pick **one asset, not the whole program** — depth finds bugs, breadth finds scanner noise. Map its purpose and users (what data flows through it, what is the worst thing an attacker could do), collect the public surface (endpoints, parameters, auth flows, roles, file formats, error behavior, anything unusual), and check the bounty table: some asset classes pay less, so hunt where the program pays for demonstrable impact.

### Step 3 — The five-source duplicate check

Run this **before writing any exploit code and before testing dynamically**. A duplicate kills the candidate — if it is already known, it earns nothing and costs a trial report or signal. A candidate that survives fewer than five sources is not cleared.

1. **Disclosed reports.** Search the platform's disclosed reports (HackerOne Hacktivity, Bugcrowd) by asset, endpoint, parameter name, and vulnerability type.
2. **GitHub issues — open AND closed.** Search the project's repositories (and forks) for the bug, the component, and related keywords. Closed issues matter as much as open ones: a merged fix, a "won't fix", and an already-filed duplicate all kill your candidate.
3. **CHANGELOG and release notes.** A fix described in a changelog — even vaguely ("improve validation on X") — means the bug is known and fixed. Check your tested version against the fixed version.
4. **Published audits.** Search for security audits of the project. If an audit reported this issue — even at a different severity — it is known.
5. **CVEs and advisories.** Search CVE databases, GitHub Security Advisories, and the project's advisory list for the component and version. A CVE for the same root cause is a duplicate even if your PoC looks different — triage judges root cause, not exploit style.

**Document the check.** One line per source: what you searched, what you found, the verdict. "Checked" with no record is not a check — when triage asks, you need the receipts. If any source shows the same root cause, stop.

### Step 4 — Static audit first

Read the code before you run anything. Dynamic testing without a hypothesis is fuzzing with extra steps.

1. **Get the code.** Clone the repository (if public) or study the API contract and observable behavior. Note the exact version and commit you audited.
2. **Map trust boundaries and data flow.** Trace untrusted input from entry point (request, file, webhook, message, user-supplied URL) through validation, storage, and output. Trust follows who *wrote* a value, not which channel delivered it.
3. **Form a hypothesis before testing.** Write one sentence: "I believe X is vulnerable to Y because Z, and an attacker would gain W." If you cannot write that sentence, you have a hunch, not a candidate.
4. **Prioritize by impact, not by novelty.** Auth and authorization flaws, logic errors in money or permission flows, injection, and broken access control (OWASP A01) usually outrank exotic edge cases. A boring IDOR with a working PoC beats a clever theoretical chain with none.

### Step 5 — Runnable proof of concept (mandatory)

A theoretical finding with no PoC is not a report. This is non-negotiable.

1. **PoC must run.** A script, a request sequence, a minimal reproduction, or a video plus steps — something the triager can execute and see the effect with their own eyes.
2. **PoC must run against an authorized target only.** Allowed: your own test instance, a program-provided test account or sandbox, a local deployment of the same code, or an in-scope asset with testing permitted. Never test production data, never touch another user's account, never move real money, never exfiltrate real data.
3. **PoC must demonstrate impact, not just a crash.** Show what the attacker gains: read another user's data, act as another user, bypass a payment, escalate privilege, execute code. A stack trace or error message alone is usually not impact.
4. **PoC must not depend on localhost-only assumptions.** If it only works with debug mode on, a permissive config, an old dependency, or a feature flag production does not have, it is not a finding against the program.
5. **Keep the PoC minimal, self-contained, and recorded.** One script, the exact requests (with headers), the exact steps, the expected and actual output — plus target URL or commit, test account used (your own), time, and tool versions.

### Step 6 — Honest severity scoring

Score with CVSS — and score honestly. Severity is where hunters most often damage their own credibility.

1. **Score the impact you proved, not the impact you imagined.** If your PoC reads one user's non-sensitive field, that is the impact — not the whole database.
2. **Use CVSS properly.** Base metrics: attack vector, attack complexity, privileges required, user interaction, scope change, and impact on confidentiality, integrity, availability.
3. **Never inflate.** Inflating a Medium to a High does not increase the bounty — it decreases triager trust, and triage will re-score it anyway.
4. **Explain impact in concrete terms.** Three sentences: what an attacker gains, who is affected, what it costs. "An unauthenticated attacker can read any user's reset token via the password-reset endpoint; account takeover is one request away." That is impact. "This is a serious security issue" is not.

### Step 7 — Vendor-boundary statement

Every report carries one line stating whose code or configuration the bug lives in — the report must go to the party that can fix it.

- **Vendor:** the bug is in the vendor's own product or service — report to the vendor's program.
- **Integrator / deployer:** the bug is in how a third party deployed or configured the vendor's product — report to the deployer; the vendor cannot fix someone else's deployment.
- **Upstream library / dependency:** the bug is in a library the vendor depends on — the fix belongs upstream, so name the library and version and check for an upstream CVE or advisory (Step 3).
- **Shared / unclear:** say so explicitly and name both parties. "Unclear" is acceptable; guessing wrong silently is not.

A report filed against the wrong party closes as informative or N/A.

### Step 8 — Write the report: one finding per report

1. **One finding per report.** Never bundle two bugs into one submission — bundled reports get split, re-scored, or closed as N/A, and the second bug is often lost.
2. **Structure every report the same way** (see Output Format).
3. **Never include in a report:** credentials, session tokens, API keys (yours or anyone else's), other users' private data, screenshots containing real user data, or internal URLs that are not part of the finding. Redact aggressively; rotate any token your PoC used before submitting.
4. **Read the report as the triager.** Re-read it once pretending you know nothing about the bug. If any step assumes knowledge you have but the triager does not, add it. If any sentence is marketing ("critical zero-day"), delete it.

### Step 9 — Submit, then manage the lifecycle

1. **Submit to the right program, once.** Confirm the program, the asset, and the report content match.
2. **Understand triage states** (see Common Scenarios).
3. **When to withdraw vs. defend.** Withdraw when you found a duplicate yourself, the PoC depends on an assumption that does not hold on the real target, or the asset is out of scope — withdrawing promptly protects your signal. Defend when triage misunderstood the PoC (clearer reproduction), scored the impact lower than demonstrated (show the next step of the chain), or closed it as a duplicate of a report with a different root cause (explain the difference, with code). Defend once, with evidence, politely — then accept the decision.
4. **Disclosure rules.** Disclose only when the program's policy allows it (usually after fix, with permission, or after an agreed embargo).
5. **Track everything.** Keep a log per program: candidate, duplicate-check result, PoC status, report ID, triage state, bounty, lessons.

## Key Concepts

| Concept | Meaning |
|---|---|
| Scope | The exact list of assets a program accepts reports on; verbatim match required |
| Trial report | A submission allowed when the researcher does not meet a program's signal requirement; the allowance is limited and pooled across programs |
| Signal | A researcher's reputation score on the platform, driven by report outcomes over the last 12 months |
| Duplicate | A report of the same root cause already filed; triage judges root cause, not exploit style |
| Proof of concept (PoC) | A runnable reproduction a triager can execute to see the impact |
| CVSS | The FIRST Common Vulnerability Scoring System; score the impact proved, not imagined |
| Vendor boundary | Which party owns the buggy code or configuration — vendor, integrator, upstream, or unclear |
| Informative / N/A | Triage states for a real-but-out-of-scope report, and for a non-vulnerability respectively |
| Disclosure | Publishing a finding; only when the program policy allows it |

## Tools & Systems

| Tool / system | Use |
|---|---|
| HackerOne, Bugcrowd, Intigriti, YesWeHack | Bounty platforms: program pages, scope, Hacktivity/disclosed reports |
| Burp Suite | Proxy, repeater, and intruder for building and replaying the PoC |
| GitHub search (open + closed issues) | Duplicate check source 2 |
| CVE databases, GitHub Security Advisories | Duplicate check source 5 |
| FIRST CVSS calculator | Honest severity scoring |
| Project CHANGELOG / release notes | Duplicate check source 3 |
| Published audit reports | Duplicate check source 4 |

## Common Scenarios

### Vulnerability classes worth hunting

Each class maps to an OWASP Top 10 category. **A01 broken access control:** IDOR, missing authorization, privilege escalation. **A02 cryptographic failures:** plaintext data, weak randomness, broken password-reset flows, predictable tokens. **A03 injection:** SQL, NoSQL, command, template, XSS. **A04 insecure design:** business-logic flaws (price manipulation, race conditions, workflow bypass, coupon abuse) — no scanner finds these. **A05 misconfiguration:** debug endpoints, default credentials, verbose errors, permissive CORS/storage. **A06 vulnerable components:** outdated libraries with known CVEs, only if the path is reachable here. **A07 authentication failures:** brute-force without rate limiting, session fixation, weak tokens, missing MFA. **A08 integrity failures:** unsigned updates, unsafe deserialization, CI/CD weaknesses. **A09 logging failures:** low bounty alone. **A10 SSRF:** user-supplied URLs fetched server-side without an allowlist.

### Triage states and your move

| State | Meaning | Your move |
|---|---|---|
| Triaged / Valid | Accepted as real and in scope; moves toward fix and bounty | Don't pester triage; answer clarifications promptly |
| Duplicate | Same root cause reported first | Accept it; log it (Step 3) |
| Informative | Real but out of scope, below threshold, or not actionable; no bounty | Learn from it; do not resubmit |
| N/A | Not a vulnerability — false positive, non-issue, misunderstood behavior, wrong asset | A learning signal, not an insult |
| Needs more info | Triage cannot reproduce it | Respond fast with the missing step, a cleaner PoC, or a video |
| Spam / Out of scope | Report violated program rules | Check what rule; do not repeat it |

### Common mistakes

| Mistake | Do this instead |
|---|---|
| Testing an out-of-scope asset | Gate on the exact scope list first (Step 1) |
| Skipping the duplicate check | Five sources, every candidate, documented |
| Reporting a known or fixed issue | CHANGELOG + CVE + closed-issue check before PoC |
| Inflating severity | Score the impact you proved, honestly |
| PoC that only works on localhost | Test against the real configuration; state gaps plainly |
| Pasting scanner output as a report | Scanner output is a lead, never a report |
| Bundling two bugs in one report | One finding per report, always |

### Red flags — stop and re-evaluate

- The program page says paused, ended, or "not accepting submissions" — do not hunt it
- A disclosed report, a closed GitHub issue, or a changelog entry describes the same root cause
- The "vulnerability" requires the victim to paste attacker input into their own browser console, or to install something — that is self-XSS / social engineering, not a program bug

## Output Format

### Report structure (one finding per report)

- **Title:** vulnerability class + affected asset + impact in one line. "Stored XSS in comment field allows account takeover on example.com" — not "Security issue."
- **Summary:** two or three sentences — what the bug is, where it lives, what an attacker gains.
- **Vendor-boundary line:** whose code/config (Step 7).
- **Severity:** CVSS vector and score, plus the concrete impact sentences (Step 6).
- **Reproduction steps:** numbered, exact, from a clean state — every request, parameter, and click. A triager should reproduce it without asking you anything.
- **Proof of concept:** the runnable script or request sequence from Step 5, attached or pasted in full.
- **Impact:** who is affected, what data or actions are at risk, realistic exploitation scenario.
- **Suggested fix:** concrete — parameterized query, authorization check on the resource, output encoding, upgrade to a fixed version.
- **References:** links to the affected code, the OWASP category, the CWE number, and any related advisory.

### Pre-submit checklist

Tick every box before submitting, or the report stays a draft.

- [ ] Program is ACTIVE and submissions are open (checked on the program page, today)
- [ ] Fee/deposit checked, amount recorded, affordable and disclosed (or program is free)
- [ ] Program policy read end to end, including out-of-scope list
- [ ] Exact asset is IN the scope list — verbatim, not by wildcard stretch or closest-option guess
- [ ] Five-source duplicate check done and documented (reports, GitHub issues, CHANGELOG, audits, CVEs)
- [ ] Static audit done; trust boundaries and data flow mapped; hypothesis in one sentence
- [ ] Runnable PoC built and run against an authorized target only, demonstrating concrete impact (not just an error) and free of localhost-only assumptions
- [ ] CVSS scored honestly on the impact actually proved; impact in concrete terms; vendor-boundary line written (vendor / integrator / upstream / unclear)
- [ ] One finding per report; title, repro steps, impact, and fix all present
- [ ] No credentials, tokens, or other users' private data anywhere in the report
- [ ] Report re-read as a stranger; every step reproducible; report ID, program, and triage state logged

### Verification

After a report is submitted:

- [ ] Report ID, program, asset, and date logged
- [ ] Triage state tracked to a terminal state (triaged, duplicate, informative, N/A, resolved)
- [ ] If informative or N/A: lesson written down before the next hunt
