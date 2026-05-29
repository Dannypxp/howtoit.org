+++
title       = "TeamPCP published their own malware on GitHub — with an instruction manual"
date        = 2026-05-21
draft       = false
tags        = ["threat intelligence", "supply chain", "malware", "github", "open source"]
description = "On May 12, TeamPCP posted the full source code for the Shai-Hulud worm to GitHub under the MIT license. Copycat actors were forking it within hours. Here's what happened and what it means."
+++

On May 12, 2026 — one day after the TanStack npm attack — the threat group TeamPCP posted the complete source code for their Shai-Hulud credential-stealing worm to GitHub. Not accidentally. On purpose. With a README that included deployment instructions.

The message in the repository read:

> "Shai-Hulud: Open Sourcing The Carnage. Is it vibe coded? Yes. Does it work? Let results speak. Change keys and C2 as needed. Love – TeamPCP."

They chose the MIT license — one of the most permissive open source licenses available, allowing anyone to use, modify, and redistribute the code freely.

Within hours, independent threat actors were forking the repo and submitting pull requests.

---

## What is Shai-Hulud?

Shai-Hulud — named after the colossal sandworms in Frank Herbert's Dune — is a credential harvesting framework built specifically for GitHub Actions supply chain attacks. It's written in TypeScript and runs on the Bun runtime, an unusual combination for malware but an effective one.

The tool is designed to run silently inside GitHub Actions environments and steal everything of value: GitHub tokens, cloud credentials (AWS, GCP, Kubernetes), SSH keys, npm configuration files, and environment variables. It exfiltrates stolen data to attacker-controlled infrastructure, with the victim's own GitHub token used as a fallback exfiltration channel if the primary command-and-control server is unreachable.

A few details from the source code stood out to researchers who analyzed it:

**Russian locale detection.** The malware includes logic to skip execution if the system locale is set to Russian. This is classic crimeware behavior — a geographic carve-out to avoid operating in jurisdictions where prosecution is more likely. It's a strong indicator of the group's likely origin.

**Obfuscated build option.** The package includes both a standard build and an obfuscated build (`bun run build:obf`) specifically designed to evade static analysis tools.

**Disguised package name.** In `package.json`, the package is named `voicefromtheouterworld` — masking its purpose if package metadata is used for quick triage.

**Spoofed commit timestamps.** Every commit in the repository is dated January 1, 2099. A deliberate obfuscation technique to make forensic timeline analysis harder and to avoid triggering any tooling that flags recently modified files.

**Sigstore provenance support.** The code shows capability to add Sigstore provenance attestations to malicious releases — the same technique used in the TanStack attack to make malicious packages cryptographically indistinguishable from legitimate ones.

---

## Why publish it at all?

This is the obvious question. Publishing your own malware is not standard practice, even for brazen threat actors. A few theories from the security community:

**Reputation and recruitment.** Publishing working, production-grade offensive tooling is a credibility signal in the cybercrime ecosystem. It establishes TeamPCP as technically capable and attracts collaborators or buyers.

**Chaos as a strategy.** By open-sourcing the worm, TeamPCP dramatically amplifies the threat landscape at no cost to themselves. Every copycat attack that uses their code creates more noise, makes attribution harder, and stretches defender resources.

**The code had already served its purpose.** The Mini Shai-Hulud campaign on May 11 was already done by the time the source was published. If the tooling was going to be burned anyway, releasing it publicly costs nothing and generates notoriety.

**Provocation.** The message "Let the Carnage Continue" and the challenge framing suggest this is partly theatrical. TeamPCP has been posting about their attacks on public forums and Telegram throughout the campaign.

---

## The copycat problem

This is the part that has defenders most concerned — not the original Shai-Hulud, but what comes next.

Within hours of the repository appearing, independent actors had already forked the code. One account, identified by OX Security, submitted a pull request adding FreeBSD support — expanding the potential target surface before GitHub had even taken the repo down. By the time GitHub removed the repositories, there were dozens of forks, and the code was already circulating independently.

The barrier to running a sophisticated GitHub Actions supply chain attack just dropped significantly. Before this release, mounting a Shai-Hulud-style attack required understanding the TanStack-style Pwn Request chain, GitHub Actions cache poisoning, and OIDC token extraction — a non-trivial combination. Now anyone can fork the code, change the C2 infrastructure as the README instructs, and run their own campaign.

Security researchers have described this as "open season" for supply chain attacks.

---

## What the code revealed about the campaign

Researchers who analyzed the source before GitHub removed it found that the code mapped directly to compiled artifacts already observed in the wild — files like `router_init.js`, `setup.mjs`, and `opensearch_init.js` that had been found on compromised machines during the TanStack and Nx attacks. This confirmed the attribution and gave defenders a much clearer picture of how the attacks worked at the code level.

The framework follows a modular pipeline design with distinct stages for initial access, credential harvesting, exfiltration, and propagation. The exfiltration uses encrypted channels and a fallback mechanism that uses the victim's own GitHub token to create hidden repositories — making the exfiltration blend in with normal GitHub activity.

---

## The timeline in context

It's worth stepping back and looking at the full sequence:

- **May 11** — TanStack npm attack. 84 malicious packages published in 6 minutes. Worm spreads to 170+ packages.
- **May 12** — TeamPCP publishes Shai-Hulud source code to GitHub with a note saying "let the carnage continue."
- **May 13** — Security researchers spot and document the repository. Forks already proliferating. GitHub removes the repo but forks survive.
- **May 18** — Nx Console VS Code extension compromised using credentials stolen in the TanStack wave. Malicious version live on the official marketplace for 18 minutes.
- **May 20** — GitHub confirms breach of ~3,800 internal repositories traced back to the Nx extension.

The open-source drop sits exactly in the middle of this timeline — between the npm attack that provided the stolen credentials and the VS Code extension attack that used those credentials to reach GitHub itself.

---

## What this means for defenders

**Signature-based detection is increasingly insufficient.** The Shai-Hulud code showed capability to generate packages with valid SLSA provenance attestations, meaning cryptographic signing no longer guarantees legitimacy. Detection needs to focus on behavior — what a package or extension *does* at runtime — not just whether it came from a trusted publisher.

**The threat landscape just widened.** Before the code drop, Shai-Hulud-style attacks were constrained to actors with enough sophistication to build the toolchain from scratch. That constraint no longer exists. Expect to see lower-sophistication actors running variants of this campaign.

**Developer tooling is the new perimeter.** Both the VS Code extension attack and the GitHub Actions chain exploit the implicit trust developers place in their tooling. VS Code extensions run with full access to the developer's machine. GitHub Actions workflows have access to secrets and publishing credentials. These surfaces need the same scrutiny as production infrastructure.

**Check your extensions now.** If you use VS Code, audit your installed extensions. Disable auto-update for any extension that has broad system access. An 18-minute window is not enough time for human review to catch a malicious update before it hits your machine.

The code may be gone from GitHub, but it's not gone. This campaign is still active, and the tools are now in more hands than before.
