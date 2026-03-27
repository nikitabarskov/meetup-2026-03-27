---
theme: default
background: https://cover.sli.dev
title: API Therapy
comark: true
duration: 35min
---

# API Therapy
## A small-talk about things we might do with APIs

Nikita Barskov, 27th March 2026

---

# whoami

Nikita, 29 y.o., living permamently in Norway since 2019.

## Things I do.
- Solve problems by writing software
- Software engineer with primary focus on data engineering and machine learning.
- A little less than 10 years of experience.
- In Norway since 2019, worked at places like Coop Norge and Volur. 
- Currently I am a full-time employee at Deepinisght, Norwegian Health Tech Start up.

## Things I know

Go, Python, OpenTofu / Terraform, Cloud, design and engineering of APIs and Data Models.

## Things I wish I knew

Design.

## Free time

I am doing triathlon for the last 9 years of my life without knowning that.

---

# What are we going to do today?

- Figure out why do we need a therapy session about APIs?
- What are the APIs, where do they come from, and why do we have them?
- **Decoupling: why it is the whole point of an interface.**
- Look at the internals of the APIs and different design choices.
- Think about how API design can improve collaboration and vice versa.
- Making sure we enjoy popular culture references.

---

# What is an Interface?

Dieter Rams joined Braun in 1955 and spent over four decades shaping what good industrial design means. His tenth principle:

> **"Good design is as little design as possible — less, but better."**

When Apple designed the original iPod in 2001, the team drew directly from Rams' 1958 Braun T3 transistor radio — the rounded rectangle, the minimal controls, the deliberate restraint. Jony Ive acknowledged Rams as a significant influence and wrote the foreword for a Rams monograph. The lineage is real and documented.

Now think about two microwaves:

**The good one:** a timer knob, a power knob, a door handle. Done.

**The average one:** 47 buttons. A "sensor reheat" button. A "popcorn" button that doesn't actually work for your brand of popcorn. You lost the manual 3 years ago. You press "start" and hope.

The second microwave is not *more powerful* — it is just *harder to use correctly*.

Rams called unnecessary complexity a failure of design. Jobs called it a failure of taste. Both were right.

This is not just about appliances.

---

# Rams' Ten Principles — Applied to APIs

Rams wrote these for physical products. They translate without modification.

| Principle | What it means for your API |
|---|---|
| **1. Innovative** | Use new capabilities where they genuinely help — not because they are new |
| **2. Useful** | Every endpoint, field, and parameter must serve a real consumer need |
| **3. Aesthetic** | Consistent naming, predictable patterns, clean error messages |
| **4. Understandable** | A consumer should grasp what a method does without reading the source |
| **5. Unobtrusive** | Your API should not impose your internal model on the consumer |
| **6. Honest** | Don't return `200 OK` when something went wrong |
| **7. Long-lasting** | Design for backward compatibility from day one |
| **8. Thorough** | Handle edge cases explicitly — nulls, empty lists, partial failures |
| **9. Environmentally friendly** | Don't transfer data the consumer didn't ask for |
| **10. As little as possible** | The smallest API surface that solves the problem is the right one |

---

# The Map Is Not the Territory

Alfred Korzybski wrote it in _Science and Sanity_ in 1933. It still applies to your API in 2026.

When someone calls your endpoint, they bring **their mental model** of what it should do.

- A functional programmer expects pure, predictable transforms.
- An OOP developer expects objects with clear responsibilities.
- A front-end engineer expects to get exactly what they need to render a screen.

None of these are wrong. They are just **different maps of the same territory.**

The question is: does your API speak the same language as the people consuming it?

---

# What IS an API?

**Application Programming Interface** — or, more usefully: a **contract**.

A contract needs to be written down. That is where API specifications come in.

| Type | Spec format |
|---|---|
| REST | OpenAPI (OAS) — YAML/JSON, generates docs, SDKs, stubs |
| RPC / binary | Protocol Buffers (`.proto`) — the contract *is* the source of truth |
| Event-driven | AsyncAPI — Kafka topics, AMQP queues, WebSocket channels |
| Language-level | Function signatures + type hints + doc comments |

*A spec that does not exist is a contract no one can read.*

---

# A Brief History of APIs

The concept is older than the term.

**1951** — Maurice Wilkes and David Wheeler publish the first documented subroutine library for the EDSAC computer at Cambridge. A catalog of reusable routines others can call without knowing the internals. Retrospectively: the first API.

**1968** — Cotton and Greatorex coin "application program interface" at an AFIPS computer graphics conference. The context: Fortran subroutine calls that abstract away the specific graphics hardware. Hardware independence through a defined boundary.

**1972** — David Parnas formalises *information hiding* — the principle that a module should expose only what callers need to know. The theoretical foundation for every API ever built.

**1988** — POSIX standardised. One API surface for Unix-compatible operating systems. Write once, run on any conforming system.

**2000** — Two things happen:
- Salesforce launches the first public commercial web API (XML/SOAP over HTTP) at the IDG Demo conference in February.
- Roy Fielding publishes his dissertation at UC Irvine — *"Architectural Styles and the Design of Network-based Software Architectures"* — and defines REST.

**2002** — Amazon opens its e-commerce platform to third-party developers via web services. AWS (cloud APIs) follows four years later.

**2015–2016** — GraphQL (Meta) and gRPC (Google) challenge REST's dominance for specific use cases.

*The medium keeps changing. The principle — hide the implementation, expose the contract — has not changed since 1951.*

---

# What an API Actually Promises

```python
def is_odd(n: int) -> bool: ...
```

That single line says:
- Give me a number.
- I will tell you if it is odd.
- I will not crash your server. I will not send an email. I will not touch the database.

This is a promise. Your API is a system of promises.

> REST endpoints, gRPC services, Kafka topics, SQL views — all of them are contracts.
> The medium changes. The idea doesn't.

---

# The Core Idea: Decoupling

An interface only has value if it **hides what is behind it.**

The moment a consumer needs to know about your database schema, your internal identifiers, your ORM model, or your deployment topology — the interface has failed. You have coupling dressed up as an API.

Decoupling is not a pattern. It is the *purpose* of an interface.

```
[ Consumer ]  ──── contract ────  [ Producer ]
                                       │
                              (database, services,
                               logic, migrations)
                               ← none of this leaks
```

**Jeff Bezos understood this in 2002.** His internal mandate to Amazon engineering teams:

> All service interfaces, without exception, must be designed from the ground up to be externalizable — designed as if the outside world could use them.

If your teams design their internal APIs as if a stranger with no knowledge of your internals will call them tomorrow, coupling cannot survive. That discipline is the mandate.

This idea runs through everything we are going to talk about:
- Which technology you choose
- How you handle breaking changes
- Who designs the API and when
- What your data model exposes vs. what it keeps private

---

# Loose Coupling in Practice

**Tight coupling** means a change on one side forces a change on the other.

**Loose coupling** means each side can evolve independently, as long as the contract is respected.

Three concrete examples:

**1. Stripe's payment API**
Stripe has been running since 2011. They have changed their internal systems many times. Their API for charging a card looks almost the same as it did a decade ago. Merchants who integrated in 2012 still work today. That is loose coupling working.

**2. The USB-C port**
Your laptop does not know or care whether you plug in a charger, a monitor, or a hard drive. The port is the contract. What is on the other end is not its problem. Adding a new device type does not require changing your laptop.

**3. Kafka topics as API boundaries**
A payments service publishes `payment.completed` events. A notifications service, a fraud detection service, and a reporting service all consume it. None of them are coupled to each other — only to the event schema. You can add a new consumer without touching the producer or any existing consumer.

*Loose coupling is not a property of the technology. It is a property of the discipline with which you treat the contract.*

---

# APIs Are Everywhere (Not Just HTTP)

The word "API" is not a synonym for "REST over HTTP." It never was.

| Type | Examples | Best for |
|---|---|---|
| REST | GitHub API, Stripe | Human-readable, broad compatibility |
| gRPC | Internal microservices | Machine-to-machine, high throughput |
| GraphQL | Meta, Shopify | Consumer-driven, flexible queries |
| Events | Kafka, Pub/Sub | Async, decoupled data flows |
| SQL Views | Data warehouses | Analysts, reporting, data products |

*Every boundary between systems is an API. If you wrote a function someone else calls — that's an API too.*

---

# Choosing Your Technology

**REST** — widely understood, works almost everywhere, browsers love it. The safe default for public-facing APIs.

**gRPC + Protobuf** — binary format, static typing, blazing fast for machine-to-machine. Browsers don't fully support bidirectional streaming, and the toolchain takes effort to set up. Worth it for internal high-throughput services.

**JSON Schema** — great for REST, but remember: JSON is *JavaScript Object Notation*, not a schema language. It doesn't know what a `datetime` is. Protobuf does.

**AIP / AEP** — Google's API Improvement Proposals (aip.dev) are Google's internal design conventions, made public. AEP (aep.dev) is a distinct, independently governed project built by practitioners from Google, Microsoft, IBM, Roblox and others — a community standard, not simply an open-source fork of AIP. Both are worth reading.

*The right choice comes from your actual needs. Not from what's trending on Hacker News.*

---

# The Protobuf vs JSON Reality Check

```protobuf
message Timestamp {
  int64 seconds = 1;
  int32 nanos = 2;
}
```

```json
{
  "created_at": "2026-03-27T10:00:00Z"  // or is it a string? or a number?
}
```

With JSON: a `datetime` field is a string. Or maybe a Unix timestamp. Or maybe both, depending on who wrote it.

With Protobuf: there is **one canonical way** to represent a timestamp. The compiler enforces it. The generated code handles it.

Small difference. Massive impact at scale.

---

# Producers and Consumers

Data Mesh got something right: **producers should own their data.**

But here is the pattern I keep seeing:

> Back-end engineer spends a week building an API. Front-end engineer gets it. Front-end engineer spends two weeks figuring out which of the 50 fields they actually need and what the other 46 are for.

This is not a front-end problem. This is a **communication failure** that happened before a single line of code was written.

Think of it like this: a chef cooking a dinner party menu without knowing who the guests are, whether anyone is vegetarian, or even how many people are coming.

The food might be excellent. It will still be wrong.

---

# Clients Are Always Right (About Their Needs)

Not a customer service slogan. A design principle.

The API should be shaped by what the consumer needs to accomplish — not by what is convenient for the producer to expose.

**Design from the outside in:**
1. What does the consumer need to do?
2. What is the simplest interface that enables that?
3. Only then: what does the producer need to implement behind it?

Most teams do this in reverse. They start from the data model, generate endpoints, and hand it over. The consumer then adapts to the producer's internal structure — which is exactly the coupling we said we were trying to avoid.

**A concrete test:** if your API consumer needs to make multiple calls and join the results themselves to accomplish one user-facing action, the API was designed inside-out.

*The client is not always right about the solution. They are always right about the problem.*

---

# What "Client-Driven" Actually Means in Practice

Consumer-first design is not the same as "the client dictates the schema."

| What it is | What it is not |
|---|---|
| Understanding the use case before designing the shape | Letting consumers define your data model |
| Returning what is needed for the task, nothing more | Exposing every field and letting the client filter |
| Designing error messages for the caller's context | Propagating internal stack traces |
| Versioning deliberately, with notice | Changing fields whenever the DB schema changes |
| Asking "what problem are you solving?" | Asking "what endpoint do you want?" |

The front-end engineer who says *"I need shoe size, brand, and model on this page"* is giving you a requirement. The back-end engineer's job is to translate that into a contract — not to hand back a 50-field object and wish them luck.


---

# The Vacuum Problem

What happens when a back-end engineer designs an API with **zero consumer input**?

They make assumptions. Hundreds of them. And every assumption is baked into the API surface:

- Field names that make sense in the database schema but not in the UI
- Internal identifiers from CRM/ERP systems leaking into the response
- 50-field objects when the client needs 4 fields
- Pagination styles that don't match the front-end's mental model

This is not incompetence. This is **missing context.**

*The back-end engineer's job is not just to build. It is to extract the problem from the consumer and solve it.*

---

# Open for Extension, Closed for Modification

Bertrand Meyer wrote it in 1988. Robert Martin made it the O in SOLID.

> **"Software entities should be open for extension, but closed for modification."**

It was written about classes. It applies perfectly to APIs.

**Closed for modification** means: once you publish a contract, you do not change it in a way that breaks existing consumers. You do not rename fields. You do not remove endpoints. You do not change the meaning of a status code.

**Open for extension** means: you can add new fields, new endpoints, new enum values, new optional parameters — without touching what already exists.

In practice:

```
✓  Add a new optional field  →  existing clients ignore it, new ones use it
✓  Add a new endpoint        →  existing integrations unaffected
✓  Add a new event type      →  consumers that don't care just skip it

✗  Rename a field            →  every consumer breaks silently or loudly
✗  Remove an endpoint        →  someone's production system stops working at 3am
✗  Change a field's type     →  runtime errors in clients you don't control
```

This is not academic. It is the difference between an API that ages well and one that generates support tickets.

---

# Breaking Changes Break Trust

> "Please download the latest version of the app. The previous version has been deprecated."

Every time you see that message, something went wrong upstream.

Here is a concrete example: you have a field called `name` on a car object. You realize it should be `model_name`. You rename it and ship.

Every client that read `name` is now broken. Not because they did anything wrong — because **you changed the contract.**

The fix was not to rename. The fix was to **add** `model_name` alongside `name`, deprecate `name` in your docs, and remove it in a future major version with advance notice. Open for extension. Closed for modification.

At scale, a broken contract means:
- 1000s of clients you don't control
- Mobile apps you can't force-update
- Partners who integrated months ago and don't check your changelog

*Your API is a promise. Breaking it has a cost, even if you can't see the bill immediately.*

---

# Database Migrations and the Blue-Green Trap

You have two replicas. You do a blue-green deployment. You drop a column.

Replica 1: migration applied. Column gone. Happy.

Load balancer: *"Replica 1 looks busy, let me send this request to Replica 2."*

Replica 2: still on the old version. Old ORM code reads for the column. Column missing. **Request fails.**

This is a violation of the same principle we just talked about — you *modified* the storage contract instead of *extending* it. The safe path is a multi-step migration:

```
Step 1: add the new column, keep the old one   ← deploy app that writes both
Step 2: backfill old data into new column
Step 3: deploy app that reads from new column only
Step 4: drop the old column                    ← now it is safe
```

This is not a theoretical edge case. It happens because the database schema, the ORM model, and the API surface are all treated as **the same thing** when they are not.

Remember the decoupling slide? This is exactly what breaks when you ignore it.

*Don't use your ORM as the source of truth for your API contract.*

---

# The Therapy: What Should You Actually Do?

**If you are a producer:**
- Before writing a single line of code, ask: *who consumes this, and what do they actually need?*
- If you don't know, go find out. That is part of the job.
- Lean toward less surface area, not more. You can always add. You cannot easily remove.

**If you are a consumer:**
- Articulate the problem, not the solution.
- "I need to show shoe size, description, brand, and model on the product page" is useful.
- "Give me a `/shoes/{id}/full-details` endpoint" is an assumption dressed as a requirement.

**Both sides:**
- APIs are a collaboration artifact. They are not thrown over a wall.
- Treat them as first-class citizens with clear ownership, versioning strategy, and a changelog.

---

# The Driving Analogy

When you drive a car, you do not think about:
- Tire pressure differential between front and rear axles
- The combustion cycle of the engine
- How the ABS sensor communicates with the braking system

You think about where you are going.

**That is what a good API should feel like for its consumers.**

The complexity is real. It just lives on the right side of the interface — **decoupled from the person using it**.

---

# Summary

- An API is a **contract**. Treat it like one.
- The purpose of an interface is **decoupling** — hide what is behind it.
- Loose coupling means each side evolves independently. The contract is what holds.
- **Open for extension, closed for modification** — add, don't rename; add, don't remove.
- Simpler interfaces require **more thought**, not less.
- Different mental models are real. Design for your **actual consumers**, not for your data model.
- Technology choices matter — pick based on **your constraints**, not trends.
- **Producers own the quality.** But consumers must articulate their needs.
- Breaking changes are not free. Plan for **backward compatibility** from day one.
- APIs are not an afterthought. They are the product.

---

# Thank You

Questions? Disagreements? Strong opinions about microwave UX?

*Find me after the talk.*

Nikita Barskov — [github.com/nikitabarskov](https://github.com/nikitabarskov)
