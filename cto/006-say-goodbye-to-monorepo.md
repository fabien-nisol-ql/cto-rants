# Why We Don't Use a Monorepo for Microservices

> _"Dev convenience is not our goal. A clean, isolated microservice architecture is."_

In today's development world, monorepos are all the rage. Everyone from big tech to trendy tooling platforms advocates for the simplicity of putting all your services, libraries, and tooling into one giant repository.

But at our company, we’ve made a deliberate architectural choice:

> **We don’t use a monorepo for our microservices — and here’s why.**

---

## 🧱 Microservices Require Isolation, Not Just Modularity

Microservices aren’t just "small services" in separate folders. They are autonomous components that:

- Are independently versioned and deployed  
- Own their own data and business model  
- Evolve without coordinating with other teams  
- Fail and recover in isolation  

A monorepo, by contrast, often undermines this in practice. While it starts as a productivity boost, it slowly introduces **coupling** in the worst places.

---

## 🔍 Where Monorepos Go Wrong for Microservices

In a monorepo setup with 10+ services, we’ve observed these common pitfalls:

### 1. **Shared Models Across Services**
Services often begin sharing internal classes or database entities across boundaries. For example:

```java
import com.shared.models.User;
```

This means that a change in `User` can break unrelated services like `order-service`, `billing-service`, or `email-service`. This **violates microservice isolation** and turns versioning into a nightmare.

### 2. **Centralized Versioning**
Monorepos tend to push for single-version tagging (e.g., `v2025.07.23`) for the whole repo. This removes the ability to say:

- “User-service is at v3.1.2”  
- “Billing-service hasn’t changed since v1.0.9”

Microservices should evolve at their own pace. Global versioning destroys this autonomy.

### 3. **Leaky Architecture via Shared Libraries**
It becomes too tempting to "just use that utility class" or "reuse this business rule" across services. Before long, services are no longer true microservices — they’re modularized pieces of a distributed monolith.

---

## ✅ What We *Do* Share — And Why That’s OK

We’re not dogmatic. Shared behavior is fine — as long as it doesn’t introduce coupling:

- ✅ Logging and monitoring wrappers  
- ✅ HTTP and REST conventions  
- ✅ Dev tooling (Makefiles, CI workflows, generators)  
- ✅ JWT and auth utilities  
- ✅ File readers, serializers, etc.

These are **infrastructure concerns**, not domain logic. We share these in a dedicated tools repo and fetch them as needed.

> 💡 But we never share business models, persistence classes, or domain logic.

---

## 🚫 Developer Convenience Is Not the Goal

Yes, monorepos are “easier” — but that’s not our benchmark.

We prioritize:

- **Service autonomy**  
- **Domain ownership**  
- **Version independence**  
- **Clear API contracts**  
- **Robust fault isolation**  

These things are **why** we chose microservices in the first place. Trading them away for dev convenience defeats the purpose entirely.

---

## 🚀 On Tech Stack Choices: Micronaut First, But Open to Justified Alternatives

While we enforce architectural boundaries tightly, we are more flexible when it comes to technology stack — within reason.

We default to **Java with Micronaut** for new services because:

- ✅ It's fast and lightweight  
- ✅ It provides solid support for dependency injection, config, testing, and native image compilation  
- ✅ Our team is proficient with it  
- ✅ It's consistent across services, making ops and debugging easier

That said, we’re not religious about it. There are valid reasons to use another language or framework, **but they must be justified**, not just personal preference:

- 🛠️ **Missing critical library support**  
- 🚀 **Performance reasons** (e.g., ultra-low latency in Rust, data pipelines in Python)  
- 📉 **Micronaut feature gaps** that would force heavy lifting or duplication

These exceptions are **rare** and should be reviewed with the architecture team. But the baseline is simple:

> **Use Java + Micronaut unless you can clearly prove there's a better technical reason not to.**

---

## 🛠️ How We Share Dev Tooling Without a Monorepo

To avoid duplication while staying polyrepo:

- We maintain a **central `dev-tools` Git repo**  
- Each service includes a thin Makefile that auto-fetches shared targets  
- All shared behavior lives in `Makefile.include` or scripts like `generate_jwts.py`  
- Services remain fully decoupled, but consistently equipped  

No submodules. No monoliths. No leaking abstractions.

---

## 💬 Final Thoughts

A monorepo may seem like an easier path — but it's **not the right one for real microservices**.

> Microservices aren’t just about folder structure. They’re about **ownership**, **boundaries**, and **independence**.

We want services that can move, break, recover, and evolve without being chained together. And that’s why we chose a clean, distributed, **polyrepo** architecture — not a monorepo.

---

🛤️ _Disagree? Agree? Want to hear more about our tooling setup or service template? Let’s chat._

