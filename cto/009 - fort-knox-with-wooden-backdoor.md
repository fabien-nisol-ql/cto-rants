# Fort Knox with a Wooden Backdoor: Why GitHub’s “Recommended” Security Model Is Wasting Everyone’s Time

*By an engineer who’s been doing this for 40 years and is tired of losing hours to pointless ceremony.*

---

## The Problem

I just wanted to do something simple:

- Repo A (CI/CD pipeline) needs to check out Repo B (a shared library).
- Both repos live in the **same GitHub organization**.
- All I need is: *“Repo A can read Repo B.”*

That’s it. Two clicks, right?

Nope. Welcome to GitHub Apps, PEM keys, JWT signing, one-hour tokens, callback URLs you don’t need, and pages of docs about OAuth flows you will never use.

---

## The “Recommended” Solution (aka the Gaz Factory)

GitHub recommends using **GitHub Apps** for cross-repo access. Sounds great in theory:

- Mint short-lived tokens (1h).
- Scoped permissions per repo.
- Central revocation and audit.

In practice:

1. You generate a **long-lived private PEM key**.  
2. You paste it into GitHub Secrets.  
3. Your workflow uses that PEM to mint short-lived tokens.  
4. If the PEM leaks? The attacker can mint unlimited tokens until you rotate it.  

Congratulations: you’ve just reinvented the SSH private key model, but with ten more moving parts.

---

## Why This Is Worse

1. **Complexity Overload**  
   - App creation (homepage URL, client secrets, webhooks).  
   - Installation on each repo (or all repos).  
   - Extra actions to mint tokens.  
   - More YAML, more moving parts, more places to screw up.

2. **Security Paradox**  
   - The PEM is long-lived. If it leaks, game over.  
   - Short-lived tokens are a fig leaf — they’re only as secure as the long-lived PEM behind them.

3. **Developer Experience Nightmare**  
   - Engineers give up and hardcode PATs (Personal Access Tokens).  
   - PATs expire, CI/CD fails at 2 AM before release.  
   - Tech debt piles up because nobody wants to wrestle the “proper” solution.

4. **The Wooden Backdoor**  
   - You built Fort Knox around token minting.  
   - But the backdoor is: **repo secrets are available to any branch workflow** unless you bolt on environments and approvals.  
   - An attacker with push rights can just exfiltrate the PEM via logs or artifacts.  
   - So much for the fortress.

---

## The Simple Alternative: SSH Deploy Keys

SSH deploy keys are boring. That’s why they work.

- Generate a keypair.  
- Public key in the target repo (read-only).  
- Private key as a secret in the CI repo.  
- Done.  

If the key leaks? It grants **read-only access to one repo**. Blast radius contained.  
No token minting. No Apps. No PEM gymnastics. Developers actually understand how it works.

---

## The Hidden Cost

This isn’t just my frustration. Multiply it across **millions of developers worldwide**:

- Hours wasted deciphering GitHub App flows.  
- Failing pipelines because PATs expired or secrets were misconfigured.  
- Shadow practices and insecure shortcuts proliferating.  
- Morale drained. Productivity lost.  

The economic cost is in the **billions**. All because engineers designed for auditors instead of developers.

---

## Conclusion

Security that nobody can use is worse than no security at all.  

GitHub’s App-based model might make sense for Fortune 500 compliance teams, but for 90% of cases it’s **overengineered security theater**. It creates more risk, not less, by pushing developers into bad practices.

Sometimes the simplest tool — an SSH key, read-only, scoped to one repo — is actually the most secure in practice.  

Because developers will use it. Correctly. Every time.

---

*Stop building bank vault doors when the janitor’s backdoor is still made of wood.*

