+++
title       = "Responding to a Tycoon2FA phishing campaign at a NERC CIP utility"
date        = 2026-05-07
draft       = false
tags        = ["incident response", "phishing", "siem", "crowdstrike", "tycoon2fa"]
description = "A walkthrough of how I detected, analyzed, and helped contain a multi-wave Tycoon2FA phishing campaign during my internship at a NERC CIP-regulated electric utility."
+++

Last summer during my internship at North Carolina's Electric Cooperatives, we got hit by a phishing campaign that turned out to be more sophisticated than it first appeared. This is a chronological walkthrough of how it was detected, how the attack evolved across multiple waves, and what containment looked like from the inside.

## Background

Tycoon2FA is a phishing-as-a-service platform that specializes in adversary-in-the-middle (AiTM) attacks. Rather than stealing a password directly, it proxies the victim through a fake Microsoft login page and captures the session cookie after MFA is completed — bypassing multi-factor authentication entirely. It's been widely used in Business Email Compromise campaigns and is sold as a kit on cybercrime forums.

## Wave 1 — Initial detection

The campaign started with what looked like a routine internal notice: an email telling employees to set up their voicemail for a new phone system we were in the process of installing. The timing was either a coincidence or, more unsettlingly, a sign the attacker had some awareness of our internal operations — a possible supply chain angle we couldn't rule out.

The email came with two lures: an HTML file named something along the lines of `voicemail_setup.html`, and a link prompting users to sign into their Microsoft account.

Detection came from two directions simultaneously. CrowdStrike Falcon fired an alert flagging that an HTML file had been executed on an endpoint. Around the same time, KnowBe4 started receiving an influx of phishing reports from employees who had flagged the email as suspicious — exactly what the training is designed to produce.

The first wave's HTML file was unobfuscated. When I opened and analyzed it, the mechanism was straightforward: the file redirected the victim to a malicious domain hosting a fake Microsoft login page. Credentials and session cookies submitted there went straight to the attacker.

## Wave 2 — The attack evolves

The second wave was a step up. The HTML payload was now obfuscated, and it included sandbox detection logic — code designed to identify whether it was being run in a virtual machine or analysis environment. If it detected a VM, it would behave differently or not execute at all, making automated analysis harder.

This is a common evasion technique used in more mature phishing kits. The fact that it appeared in wave 2 suggested the attacker was iterating based on what was and wasn't working, or that the kit they were using had this capability built in and they'd upgraded to a more advanced version.

The second wave hit more users than the first.

## Wave 3 — Broader reach

By the third wave the attacker had refined the approach further. More employees were hit. The campaign was clearly still active and the attacker wasn't deterred by the initial noise — they kept pushing.

## Identifying affected users

Some users self-reported — they called in to say they'd clicked the link or opened the HTML file, which immediately put them on the list. But self-reporting only catches the people who knew something was wrong.

To find the rest, I worked through CrowdStrike's SIEM. I queried for:

- Endpoints that had executed the HTML file
- Users who had visited the malicious domain
- Accounts that had opened the email in Outlook

Cross-referencing those three data sources gave me a more complete picture of the blast radius. By the time I'd finished pulling and correlating that data, the affected user count was in the teens.

## Containment recommendations

I put together a set of containment recommendations and presented them to the head of IT, who made the final calls:

**Laptop disposal** — For users whose machines had executed the obfuscated HTML payload, I recommended disposing of the laptops entirely. Given that the payload included VM detection and was designed to evade analysis, we couldn't be confident about what it had done once it ran. In a NERC CIP environment where regulatory compliance is on the line, the risk calculus favored replacement over remediation.

**Account deletion and rebuilding** — Affected accounts were deleted and rebuilt rather than simply having passwords reset. Because Tycoon2FA captures session cookies post-MFA, a password reset alone doesn't invalidate an active stolen session. Full account rebuilding was the cleaner solution.

**Email format rotation** — To prevent the attacker from continuing to target the same employees in subsequent waves, I recommended changing the company's email naming format. If the attacker had harvested a list of valid email addresses in the format `firstname.lastname@domain.com`, switching the format made that list stale.

**KnowBe4 campaign ramp-up** — We increased the frequency and intensity of simulated phishing campaigns to reinforce awareness across the organization while the incident was still fresh.

**Barracuda gateway rule fix** — The root cause of why the emails were getting through in the first place came down to a misconfigured rule in the Barracuda email gateway — a setting related to how a specific email account was handled that allowed the malicious emails to bypass filtering. Fixing that rule closed the gap that had let all three waves in.

## Outcome

The response resulted in some employees receiving new laptops, others waiting for replacements, and all affected accounts being rebuilt. The email naming format was changed. The Barracuda gateway rule was corrected, which stopped further waves from reaching inboxes.

No regulatory incidents were reported. The campaign was contained.

## Takeaways

A few things stood out from this response that I think are worth noting:

**Self-reporting is not a detection strategy.** The users who called in to report clicking were helpful, but they were a fraction of those affected. SIEM correlation — cross-referencing file execution, domain visits, and email opens — found the rest. Detection can't rely on users knowing they've been compromised.

**Obfuscation and VM detection are table stakes now.** Wave 1 was unobfuscated. Wave 2 had sandbox evasion. That's not a custom capability — it's a feature of the Tycoon2FA kit. Phishing-as-a-service has lowered the bar to the point where even commodity campaigns can evade basic analysis environments.

**AiTM bypasses MFA.** The standard advice of "enable MFA and you're protected from phishing" is incomplete. AiTM attacks like Tycoon2FA specifically defeat TOTP and push-based MFA by capturing the session cookie after authentication completes. Phishing-resistant MFA — hardware keys like YubiKeys, or passkeys — is the actual answer.

**In a regulated environment, the cost of uncertainty is high.** The decision to dispose of laptops rather than remediate them was driven partly by the NERC CIP context. When the consequences of a missed infection include regulatory penalties and potential grid impact, the bar for "clean enough" is higher than in most environments.
