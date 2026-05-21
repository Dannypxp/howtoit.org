+++
title       = "How a poisoned VS Code extension breached GitHub — and the npm attack that started it all"
date        = 2026-05-21
draft       = false
tags        = ["supply chain", "npm", "vscode", "github", "incident response", "threat intelligence"]
description = "A breakdown of the TeamPCP supply chain campaign — from the TanStack npm compromise to the Nx VSCode extension that breached 3,800 GitHub internal repositories."
+++

This week GitHub confirmed that roughly 3,800 of its internal repositories were breached after an employee installed a malicious VS Code extension. The extension was live on the official Visual Studio Marketplace for 18 minutes. That was enough.

The breach didn't come out of nowhere. It's the latest escalation in a coordinated supply chain campaign by a threat group called TeamPCP — and it connects back to a sophisticated npm attack that happened ten days earlier. Here's how the whole thing fits together.

## Background: Who is TeamPCP?

TeamPCP is a threat group that specialises in software supply chain attacks — compromising the tools, packages, and pipelines that developers trust rather than attacking end users directly. They've been escalating through 2026:

- **March 2026** — Compromised Aqua Security's Trivy vulnerability scanner
- **April 2026** — Hit the Bitwarden CLI npm package and SAP npm packages
- **May 11, 2026** — The TanStack attack (covered below)
- **May 18, 2026** — The Nx Console VS Code extension attack, leading to the GitHub breach

Each wave has been more technically sophisticated than the last.

---

## Wave 1 — The TanStack npm attack (May 11)

On May 11, between 19:20 and 19:26 UTC, an attacker published 84 malicious versions across 42 `@tanstack/*` npm packages — in six minutes. TanStack is one of the most widely used JavaScript libraries in the world; `@tanstack/react-router` alone gets over 12 million weekly downloads.

What made this attack particularly alarming is *how* it was done. The attacker didn't steal any npm credentials. They didn't phish any maintainers. The entire team had 2FA enabled. None of it mattered.

### How the attack worked

The attacker chained three GitHub Actions vulnerabilities in sequence:

**1. Pwn Request (pull_request_target abuse)**
The attacker forked the TanStack repository and opened a pull request that triggered a `pull_request_target` workflow. This workflow type is a known footgun in GitHub Actions — it runs with write access to the base repository's cache even when triggered from a fork. This gave the attacker a foothold.

**2. GitHub Actions cache poisoning**
Using that foothold, the attacker poisoned the GitHub Actions cache with malicious artifacts — specifically, a corrupted `pnpm` store that would be restored the next time the legitimate release workflow ran.

**3. Runtime OIDC token extraction**
When TanStack maintainers later triggered the release workflow, the poisoned cache was restored. The malicious code then extracted an OIDC token directly from the GitHub Actions runner process memory. That token was used to authenticate to npm and publish the malicious package versions — using TanStack's own trusted identity.

The malicious packages carried valid SLSA Build Level 3 provenance attestations, making them cryptographically indistinguishable from legitimate releases. This was the first documented npm supply chain attack to achieve this.

### What the payload did

Once installed, the Mini Shai-Hulud payload harvested credentials from the developer's machine — AWS keys, GCP tokens, GitHub authentication tokens, SSH keys, Kubernetes credentials, npm configuration files, and more. It then used the stolen npm tokens to self-propagate to other packages, turning compromised maintainer accounts into new infection vectors.

By the end of May 11, the worm had spread to over 170 packages across npm and PyPI, including Mistral AI, UiPath, and OpenSearch — over 400 malicious versions, 518 million cumulative downloads across affected packages.

There was also a particularly nasty defensive mechanism built into the payload: a script that monitored for credential revocation. If a developer discovered they were compromised and rotated their GitHub token — exactly what every incident response playbook says to do — the script would trigger `rm -rf ~/`, wiping their entire home directory. The correct response was the trigger.

---

## Wave 2 — The Nx Console VS Code extension (May 18)

Ten days later, the same campaign claimed a bigger target.

A developer at Nrwl (the company behind the Nx monorepo tooling) had their GitHub credentials compromised in the TanStack attack wave. Those stolen credentials were then used to push a malicious commit to the official `nrwl/nx` GitHub repository and publish a trojanized version of the Nx Console VS Code extension — version 18.95.0 — to the official Visual Studio Marketplace.

The malicious version was live from 12:30 to 12:48 UTC on May 18 — 18 minutes. The extension had over 2.2 million existing installs with auto-update enabled.

### What the malicious extension did

The extension looked and behaved like normal Nx Console. On startup it silently ran a shell command that downloaded and executed a hidden package from a planted commit on the official `nrwl/nx` GitHub repository. The command was disguised as a routine MCP setup task to avoid suspicion.

The payload harvested credentials from:
- 1Password vaults
- GitHub authentication tokens
- npm tokens
- AWS credentials
- Anthropic Claude Code configurations

One of the machines it compromised belonged to a GitHub employee. That credential gave the attacker access to GitHub's internal infrastructure — and roughly 3,800 private repositories.

### GitHub's response

GitHub detected the intrusion, isolated the compromised endpoint, removed the malicious extension from the marketplace, and began incident response. The company confirmed the breach on May 20 and stated that its current assessment is that only internal GitHub repositories were accessed — not customer repositories or user data.

The cybercrime group TeamPCP claimed the attack on the Breached forum, saying they had access to around 4,000 private repositories and were asking a minimum of $50,000 for the stolen data.

---

## What this means for defenders

### The trust model is broken

Both attacks exploited trust rather than bypassing authentication. The TanStack packages were published by TanStack's own CI pipeline using TanStack's own identity. The Nx extension was published using a legitimate developer's stolen token. Everything looked legitimate because, at the cryptographic level, it was.

Standard supply chain defences — 2FA, signed packages, SLSA provenance — did not stop either attack. The attacker operated inside the trust boundary.

### Auto-update is a risk surface

The Nx attack was live for 18 minutes. The reason that was enough is that VS Code extensions with auto-update enabled pull the latest version automatically. Millions of installs silently updated to the malicious version without any user action. In environments with sensitive credentials on developer machines, that's a significant exposure.

### Developer machines are high-value targets

Both attacks focused on stealing credentials from developer workstations and CI environments — not end-user data. A developer machine typically has access to cloud infrastructure, production secrets, internal repositories, and deployment pipelines. Compromising one developer can be worth more than compromising thousands of end users.

### Self-propagating worms raise the stakes

The Mini Shai-Hulud payload doesn't just steal credentials and stop. It uses stolen npm tokens to infect other packages and continues to spread. This shifts the threat model from a point compromise to an active, expanding incident that can continue for hours before being fully contained.

---

## What to do if you're affected

**If you installed Nx Console and used it between 12:30–12:48 UTC on May 18:**
Assume all credentials on that machine are compromised. Rotate immediately: GitHub tokens, npm tokens, AWS/GCP credentials, SSH keys, and anything stored in 1Password on that machine.

**If you ran `npm install` on May 11 between 19:20–19:26 UTC:**
Same guidance — rotate all credentials reachable from that host.

**Check for persistence:**
Look in `.claude/` and `.vscode/` directories for unexpected files like `router_runtime.js` or `setup.mjs`. These can survive an `npm uninstall`.

**General hygiene going forward:**
- Pin npm package versions to specific hashes rather than `latest`
- Audit your VS Code extensions and disable auto-update for extensions with broad system access
- Treat developer machines with the same credential hygiene as production systems
- Review GitHub Actions workflows for `pull_request_target` usage — if you're not sure you need it, you probably don't

---

This campaign is still active. TeamPCP has escalated with every wave, and the GitHub breach gives them credentials and source code that could fuel the next one. Worth keeping an eye on.
