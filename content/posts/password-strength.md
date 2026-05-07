+++
title       = "How to actually make a strong password (and why most advice is wrong)"
date        = 2026-05-07
draft       = false
tags        = ["passwords", "authentication", "fundamentals"]
description = "Most password advice focuses on complexity rules that don't work. Here's what actually makes a password hard to crack."
+++

Most people have been taught the same password advice: use uppercase letters, numbers, and symbols. Make it at least 8 characters. Change it every 90 days.

Almost all of that is wrong, or at least badly incomplete. This post breaks down what actually determines password strength, how attackers approach cracking, and what you should actually do.

## What "password strength" really means

A password is strong if it takes an attacker an impractical amount of time to guess it. That's it. Strength isn't about how complicated it looks — it's about how long an exhaustive search takes.

That time depends on two things:

- **Entropy** — how many possible passwords exist in the space you're drawing from
- **Cracking speed** — how many guesses per second an attacker can make

If an attacker can make 10 billion guesses per second (realistic with a modern GPU and a leaked hash), a truly random 8-character password using letters, numbers, and symbols has about 6.6 quadrillion combinations — which sounds like a lot until you realise it falls in under 8 days of continuous cracking.

## How attackers actually crack passwords

Understanding the attack shapes the defence. Attackers rarely brute-force every possible combination from scratch. Instead they work in layers:

### 1. Leaked database lookups

The first thing any serious attacker does is check if your password hash matches one in a breach database. Sites like Have I Been Pwned index billions of leaked credentials. If you've reused a password that appeared in any previous breach, it's already known — complexity doesn't matter.

### 2. Dictionary attacks

Attackers use wordlists — the 10 million most common passwords, every word in the English dictionary, common names, sports teams, keyboard walks (`qwerty`, `123456`). Tools like Hashcat can run through these at enormous speed.

### 3. Rule-based attacks

This is where "add a number and symbol" advice falls apart. Attackers know that humans follow predictable patterns when told to add complexity:

- `password` → `Password1!`
- `liverpool` → `L1verpool!`
- `sarah` → `Sarah1990!`

Hashcat has thousands of built-in rules that apply exactly these transformations automatically. If your password started as a word and you "complexified" it, it's vulnerable.

### 4. Mask attacks

Attackers model password structure. They know that most 8-character passwords follow patterns like `Xxxxxxx1` (capital first letter, lowercase body, number at end). Mask attacks enumerate only passwords matching likely structures, dramatically reducing the search space.

## What actually makes a password strong

### Length beats complexity

Entropy scales dramatically with length. Every additional character multiplies the search space by the size of your character set. A 20-character lowercase-only passphrase is exponentially harder to crack than an 8-character password full of symbols.

Compare:
- `P@ssw0rd!` — 9 characters, looks complex, cracks in seconds because it's a known pattern
- `correct horse battery staple` — 4 random common words, 28 characters, would take longer than the age of the universe to brute-force

This is why NIST (the US National Institute of Standards and Technology) updated its guidelines in 2017 to recommend length and randomness over forced complexity rules.

### Randomness matters more than you think

The problem with human-chosen passphrases is that humans are bad at random. We pick words that mean something to us, we use cultural references, we reach for the same handful of "random" words. Attackers know this and build wordlists accordingly.

True randomness means using a source of entropy that has no connection to you. A password manager generating a random string, or the Diceware method (literally rolling dice to pick from a wordlist), gives you genuine randomness.

### Uniqueness across sites

A strong password used on multiple sites is a weak password. When one site is breached and your credentials are dumped, every other site where you use that password becomes compromised. Credential stuffing — automatically trying leaked username/password pairs across other sites — is one of the most common attack vectors today.

## Practical recommendations

**Use a password manager.** This is the single highest-impact change most people can make. Bitwarden (open source, free), 1Password, and KeePassXC are all solid options. Let it generate and store a unique random password for every site. You only need to remember one strong master password.

**For your master password, use Diceware.** Roll five or six dice, look up each result in the [EFF Diceware wordlist](https://www.eff.org/files/2016/07/18/eff_large_wordlist.txt), and concatenate the words. This gives you a genuinely random passphrase that's both memorable and has high entropy.

**Enable MFA everywhere it's offered.** A strong password and MFA together mean an attacker needs both your password *and* physical access to your second factor. TOTP apps (Aegis on Android, Raivo on iOS) are better than SMS codes, which are vulnerable to SIM swapping.

**Check Have I Been Pwned.** Visit [haveibeenpwned.com](https://haveibeenpwned.com) and check your email address. If any of your accounts have appeared in a breach, rotate those passwords immediately.

**Stop changing passwords on a schedule.** NIST explicitly recommends against forced periodic rotation unless there's evidence of compromise. Frequent rotation encourages weaker passwords because people struggle to remember new ones. Change passwords when there's a reason to, not on a calendar.

## The underlying principle

Password strength is about the cost of exhaustive search. Make that cost high by using long, random, unique passwords — and remove the human predictability that rule-based attacks exploit. A password manager does most of this work for you automatically.

The goal isn't a password that *looks* strong. It's one that genuinely is.
