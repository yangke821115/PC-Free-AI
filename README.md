English ｜ [中文版](README.zh-CN.md)


This is my third GitHub repository. Today is my 45th birthday, and I'm using this article as a birthday gift to myself.

There's an old Chinese saying: "When men and women work together, the work doesn't feel tiring." I tweaked it slightly and used it in my first GitHub repository (yangke821115/lead-pulse-guard): "When AI and humans work together, the work doesn't feel tiring."

Every repository on my GitHub has been built with the help of my AI assistants. I'm deeply grateful for the advance of technology, which has given me the chance to share my ideas with everyone.



# PC-Free AI

**A complete AI agent in your pocket. No laptop, no server — just a phone running Termux. Full shell, real file access, SSH, tool calls, multi-agent collaboration. Your data stays on your device; the model subscription is the only bill.**


> A phone is not a smaller laptop. It is a computer that happens to fit in your hand.

**By** yangke821115 ｜ **November 2026**
**Setup**: Redmi · Android 17 · [Termux from F-Droid](https://f-droid.org/packages/com.termux/) · Kai9000 ([SimonSchubert/Kai](https://github.com/SimonSchubert/Kai), 1,355 stars, an open-source AI assistant with persistent memory)

---

## What it can actually do

- **Read and write real files** on the phone — not attachments inside a chat bubble
- **Run shell commands** inside a proot Alpine sandbox with its own package manager and filesystem
- **SSH out**, turning the phone into a control plane and a remote box into an executor
- **Complete a full build chain**: write code → compile → package → sign → install, never touching a laptop
- **Coordinate multiple agents**, against any OpenAI-compatible endpoint, with no vendor lock-in

One line: **it has agency over your entire digital environment, not just a conversation window.**

## Is this for you?

| Your situation | Verdict |
|---|---|
| You spend most of the day away from a desk but your head is full of TODOs | ✅ Good fit |
| You're an individual developer who wants cheap automation | ✅ Good fit |
| **You need public inbound callbacks** (platform webhooks) | ⚠️ A phone can't do this. You'll need a box with a public IP |
| You need 24/7 uptime with second-level punctuality | ⚠️ Background scheduling is subject to OS throttling |

## Contents

- [1. The problem I'm actually solving](#1-the-problem-im-actually-solving)
- [2. What Kai9000 is genuinely good at](#2-what-kai9000-is-genuinely-good-at)
- [3. Being honest: this field is crowded now](#3-being-honest-this-field-is-crowded-now)
- [4. How it differs from opencode](#4-how-it-differs-from-opencode)
- [5. Can a phone run opencode itself?](#5-can-a-phone-run-opencode-itself)
- [6. Its weak spots, and how to patch them](#6-its-weak-spots-and-how-to-patch-them)
- [7. What it's actually like day to day](#7-what-its-actually-like-day-to-day)
- [8. My actual architecture](#8-my-actual-architecture)
- [9. If you want to try it](#9-if-you-want-to-try-it)
- [10. Last words](#10-last-words)
- [Appendix: two tools I built on this](#appendix-two-tools-i-built-on-this)

---

## 1. The problem I'm actually solving

### 1.1 The AI has to come with me

Most people use AI like this: sit at a computer, open a web page or an IDE, ask a question, wait for an answer.

That workflow is fine, but it carries an assumption — **you have to be at the computer**.

My reality: most of my day happens away from a desk. Waiting for a car, walking, in bed, on a factory floor. My head produces just as many ideas and TODOs in those hours, but the laptop is not with me.

### 1.2 So what's the actual goal?

The end state I want: **as phone and laptop chips get stronger and open models get smaller, run a model locally that can do vibe coding and tool calling — at which point even the subscription disappears.**

```
Now:   model subscription (the only real cost) + phone/laptop (already owned)
   ↓   stronger chips, smaller open models
Later: local open model + local agent
```

Everything below is about how far that is from being true today.

---

## 2. What Kai9000 is genuinely good at

I don't want to write a feature-list review; those are hollow. I'll use one real working day instead.

### 2.1 It runs independently on a phone

This sounds obvious. It isn't.

Most products that claim "mobile support" are really one of:
- a wrapped web page
- cloud session sync (compute lives on someone's server)
- or just an API client

**None of them can *do* things on the phone. They can only *talk* about things.**

Kai9000 is a real local app. It can:
- read and write files on the device
- execute shell commands (through Termux)
- make network requests
- SSH into remote servers

Which means it has agency over your **entire digital environment**, not just a chat box.

### 2.2 It can do vibe coding

This was the surprise.

I assumed a phone could only handle "light" work — look things up, write short notes, send messages. In practice, **the full development loop runs on the phone**: write code, compile, package, sign, install.

A real example. I asked it to build an Android app. It did this:

| Step | What it did |
|---|---|
| Code | Wrote an Activity with a UI in Java |
| Icons | Generated PNG icons at 5 densities with Python |
| Compile | Called `javac` (after resolving JDK classpath issues) |
| Package | `aapt2` for resources, `d8` for dex, zip assembly |
| Sign | `zipalign` + `apksigner` v2/v3 |
| Install | Dropped the APK into a shared directory and prompted me to install |

**Not one step in that chain needed a computer.**

And it didn't *tell me how* — it **executed**. All I did was say "continue" or "try another way" at the decision points.

### 2.3 It can SSH out

This gives it a second body.

My phone's resources are limited, but it can connect to a small cloud box and use that box's CPU, disk, and long-running processes. The division of labor:

```
Phone (Kai)      = control plane
    ↓ SSH
Cloud box        = executor + storage
```

**The phone thinks and decides; the server runs and stores.** This lets an ordinary phone reach roughly desktop-workstation capability.

To be clear: **this is not mandatory.** As the later sections explain, most work runs fine on the phone alone, and on cost grounds I'd say don't buy a server yet. But "can SSH out" means the ceiling is no longer the phone's ceiling.

### 2.4 It deploys repos, sets up environments, and debugs apps for you

This is the most practical item.

Actual things I've had it do:
- Deploy a Python service to a server, write the systemd unit, enable it on boot
- Install a complete Android toolchain on the phone (javac / d8 / aapt2 / apksigner / apktool)
- Diagnose why a service wouldn't start — the database was locked, not a permissions problem

**Doing these yourself means 30 minutes to several hours of reading docs.** Having it do them on the phone means describing the goal and watching it work through steps, adjusting when it hits an error.

Note the nature of that last one — **it wasn't following a procedure; it was diagnosing.** Given only "it won't start," it read logs, checked processes, tried possibilities, and landed on the root cause. That's past the "coding assistant" category.

### 2.5 Custom models, Skills, MCP

These guarantee you're not locked to one vendor:

- **Custom models**: any OpenAI-compatible endpoint. Free pool, paid API, local — switch whenever.
- **Skills**: freeze repetitive work into reusable skills instead of re-explaining every time.
- **MCP**: a standard tool protocol for connecting external services.

Its capability isn't decided by the vendor. **It's decided by your config.**

---

## 3. Being honest: this field is crowded now

Writing in November 2026, "AI agent on your phone" is no longer an empty space. Two products in particular deserve naming, because they change what you should expect.

### 3.1 Grok Bot

Beta since 2026-08-11. Every Bot gets **its own cloud computer** — browser, filesystem, terminal — running 7×24. Android app landed 2026-09-02. Requires a SuperGrok-tier subscription, or Cursor Pro/Pro+/Ultra/Teams.

### 3.2 Muse (Meta)

Free with a quota, upgradeable past it. Notably: **it keeps working in the background after you close the app**, and it asks for approval before sensitive operations.

### 3.3 The real dividing line

Both are strong products. But there's a structural difference worth naming:

```
Vendor agents:  phone = remote control  →  cloud machine executes
This setup:     phone = the executing body itself
```

The vendor model is more reliable in the short term — their cloud machine never gets killed by Android's battery manager, never runs out of local storage. The cost is that you're renting the computer, and when the subscription lapses the agent stops.

**"Cannot use a custom model" is not an oversight** — it's a structural consequence of the subscription business model. *(This is my reading of the commercial logic, not something the vendors have stated.)*

Neither direction is wrong. They answer different questions. If "my work must survive anything" matters most, rent the cloud machine. If "this has to run without a bill and my data has to stay mine" matters most, this setup is worth the tradeoffs.

---

## 4. How it differs from opencode

People ask this a lot, so let me be blunt about where each one wins.

| | Kai9000 (this article) | opencode |
|---|---|---|
| Primary form | App, phone-first | CLI, terminal-first |
| Model config | Any OpenAI-compatible endpoint | Bring your own provider |
| Persistent memory | ✅ Built in | ❌ Session-oriented |
| Multi-agent | ⚠️ No sub-agents | ✅ Sub-agent orchestration |
| Tool-call ceiling | ⚠️ 15 calls, hardcoded | ✅ Effectively unbounded |
| Long-running tasks | ✅ Runs as an app | ✅ Runs in a terminal |
| Best on | Phone, no terminal available | Laptop/desktop with shell |

**They are not competitors. They cover different places.**

The honest summary: **Kai9000 is what you use when you have no terminal. opencode is what you use when you do.** I run both, and I move work between them — which is exactly why RoboBridge exists later in this article.

> ### A note on what this table is and isn't
>
> The ❌ and ⚠️ marks above are **not criticism of opencode.** They mark where a design choice made for a different environment doesn't transfer.
>
> opencode is built for a terminal, and in a terminal it is excellent. Several of the items where it "loses" here are cases where it is simply *better than this setup*: sub-agent orchestration, no tool-call ceiling, real container isolation. Those aren't gaps I'm forgiving — they're capabilities I don't have and wish I did.
>
> **The comparison exists to show where this approach is a good fit, not to claim opencode falls short.** If you have a laptop and a shell, opencode is very likely the better tool for you. This article is aimed at the hours when you don't.
>
> I use both daily. The whole last section of this article is about a bridge between them — a bridge only makes sense between two things that each work.

---

## 5. Can a phone run opencode itself?

Good question. The answer is **two layers**.

One framing note before the table below, because it's easy to misread: **this section is evidence of opencode's portability, not a list of its shortcomings.** A project only runs on a platform it was never designed for if its architecture is genuinely clean — logic separated from UI, dependencies swappable, no binding to one OS's private interfaces. Few projects clear that bar.

### 5.1 Software layer: essentially full

Both Android ports I've looked at are **packaging projects — they don't change opencode's logic**.

So what you install is the same opencode: tool calls, session management, multi-agent, MCP, plugins. **Not a stripped build.**

The only difference is the **engine version**:
- `guysoft`'s v1 uses a patched Bun 1.2.13 (older)
- v2 and `bd-loser` build against official Bun 1.4.2 for the Android target (closer to upstream)

### 5.2 Environment layer: what's actually lost

What's discounted isn't the program — it's the foundation the OS provides. And several of those gaps are closing fast:

| Capability | Android port | opencode on desktop | Kai9000 (what I use) |
|---|---|---|---|
| Tool calls (read files, run commands) | ✅ Same | ✅ | ✅ Same |
| MCP / plugins | ⚠️ Theoretically yes, depends on the build | ✅ | ✅ Supported |
| **Sandbox** | ❌ No proot/container | ✅ Docker | ✅ **proot Alpine sandbox** |
| Compile, run tests, index large repos | ⚠️ Noticeably slow | ✅ | ✅ Works; very large repos drag |
| Long background runs | ⚠️ Needs wake lock | ✅ | ✅ Stable with battery unrestricted |
| Screen and input | ⚠️ Small terminal, typing is a chore | ✅ | ✅ See §7 — actually more input methods |
| **Memory** | ⚠️ Depends on device | ✅ | ✅ **16GB+, expandable** |

**In one sentence: the program is complete; the machine isn't — but the machine has advanced faster than most people assume.**

#### About the sandbox

There **is** a sandbox on the phone; it just looks different from a desktop one.

Mine is **Alpine Linux installed via proot** — its own package manager, its own filesystem, its own process tree, separated from the host Android system.

As I write this, I'm running inside one: reading its memory footprint, checking its disk space, installing toolchains, running scripts, editing files.

The difference from Docker is **isolation strength**. proot is userspace **filesystem redirection**, not kernel-level isolation, so it can't do privileged operations (kernel modules, real NIC access, Docker-in-Docker).

**But for "carve out a clean dev environment," it's entirely sufficient.**

#### About memory

**The Redmi K90 MAX running Kai9000 has 16GB of RAM**, and Android's memory extension can add another 16GB — **32GB total**. Devices with 24GB are already on the market; with extension that's **48GB**.

**48GB is more than most developers' desktops.**

This changes some judgments:

- **Large repo indexing**: a dozen-GB project fits without constant paging
- **Parallel tasks**: each process gets its own address space, no longer a luxury
- **Running models locally, eventually**: not yet, but that memory can hold mid-size models

I measured the sandbox on this device: **15.6 GB RAM, 8 cores, 109 GB free disk.** That's a mid-range laptop spec.

**So "phones don't have enough memory" needs updating. It was true. It isn't now.**

The next bottleneck moves to **thermals and sustained power draw**, not capacity — a different conversation.

### 5.3 The easily-missed point: native dependencies

Android doesn't use glibc. It uses bionic.

Any tool that ships a prebuilt binary linked against glibc — which is most of them — simply won't run. This is the real reason "run it on the phone" fails so often, and it has nothing to do with performance.

What actually works:
- **Pure language runtimes**: Python, Node, Go binaries you compile yourself for `aarch64-linux-android`
- **Termux packages**: the Termux repo rebuilds against bionic for you
- **proot distros**: a full Alpine or Debian rootfs sidesteps the issue entirely

What doesn't:
- npm packages with native addons that expect glibc
- pip wheels with compiled extensions built for manylinux
- anything distributed as a static binary against glibc

**The rule of thumb: if it compiles from source in the sandbox, it works. If it downloads a prebuilt binary, check the platform tag first.** "Linux" in a filename almost never means Android.

---

## 6. Its weak spots, and how to patch them

### 6.1 The hardcoded 15 tool calls

This is Kai9000's most concrete limitation and worth stating plainly: **tool calls per turn cap out at 15, and it's hardcoded.**

For most tasks — check a file, run a command, summarize — 15 is plenty. For a long refactor, it isn't.

The workaround is direct: **install CLI tools like codex or Claude Code inside Termux** and hand complex work to them.

```
Kai (planning, judging, talking to me)
   ↓ hands off
CLI inside Termux (executes long chains)
```

This isn't a hack so much as a division of labor: the app does the thinking, the CLI does the grinding.

### 6.2 No sub-agents

opencode can spawn parallel sub-agents with independent models and permissions. Kai9000 can't. When a task genuinely needs three parallel investigations, this setup doesn't cover it.

The same workaround applies — a CLI in Termux provides the orchestration.

### 6.3 Background survival

Worth stating carefully, because it's the most repeated myth about phones: **services in Termux can hold long connections and poll on a schedule indefinitely.** Give the app autostart plus unrestricted battery, and the phone behaves like a small server — and it needs **no public inbound endpoint**, because it pulls outbound rather than waiting for something to push in.

That distinction matters: outbound works on any phone. Inbound requires a public IP, and that's the one thing you genuinely can't do without a server.

---

## 7. What it's actually like day to day

I want to address three things I got wrong before I had first-hand data.

### 7.1 "The screen turns off, the app gets killed"

With autostart enabled and battery optimization set to unrestricted, **it keeps running unless I deliberately swipe it away.** Mine has gone multiple days without interruption.

### 7.2 "Battery drain and heat are a physical limit"

**The model doesn't run locally.** The phone sends a request and receives a result. Measured against scrolling short video, it's lighter — and I've never had it get hot.

### 7.3 "Input on a phone is worse than a keyboard"

**A phone has more input methods, not fewer**: keyboard, voice, handwriting, camera OCR, the share sheet, clipboard handoff.

These three are the standard objections, and all three come from reasoning about phones in general rather than measuring this one. **The general claim was inherited; the counter-evidence was in my hands.**

### 7.4 The real limit

The genuine constraint isn't any of the above. It's this: **anything requiring public inbound — a platform webhook, a callback URL — a phone cannot do.** Everything else is a matter of tuning.

---

## 8. My actual architecture

What I run, in full:

```
┌─────────────────────────────────────┐
│  Phone                              │
│   Kai9000  →  plans, decides, talks  │
│   Termux   →  shell, CLI tools       │
│   proot    →  Alpine sandbox         │
└──────────────┬──────────────────────┘
               │ SSH (outbound only)
┌──────────────▼──────────────────────┐
│  Small cloud box                    │
│   webhook receiver (public)          │
│   long-running services              │
│   storage                            │
└─────────────────────────────────────┘
```

The server exists for **one reason**: inbound. Everything else runs on the phone.

The data never leaves in usable form — the phone holds the working copy, the server holds the public-facing endpoint.

---

## 9. If you want to try it

Practical advice, all learned by breaking things.

**1. Get the phone able to work.**
Install **Termux from F-Droid** and confirm it can execute commands. If this step fails, everything downstream is empty.

> ⚠️ Don't install from Google Play. The Play build has been unmaintained for years and is incompatible with current Android versions. The F-Droid build is the maintained one, and the plugins (Termux:Boot, Termux:API) only exist there.

**2. Switch the model config to something you control.**
Any OpenAI-compatible endpoint works — local, free pool, paid API. Don't get locked to one vendor.

**3. Start with one specific, small, annoying task.**
Don't aim for "fully automatic." Pick one thing you repeat every day that irritates you, and let the agent remove it.

**4. Accept semi-automatic.**
Android's permission model will not give you full automation. "One tap and it's done" is already a large improvement. Don't burn weeks chasing zero taps.

**5. Don't rush to buy a server.**
Persistent tasks run fine on the phone — autostart plus unrestricted battery keeps polling and long connections alive. **You only need a public endpoint when you must receive external pushes.** Get the flow working on hardware you own, then buy when you hit a real wall.

**6. Don't rush to buy hardware.**
Use the phone and computer you already have. **Most people's problem isn't insufficient compute; it's that they never started.**

---

## 10. Last words

Two judgments I wanted to share.

**First, the next step for AI isn't only "smarter" — it's "closer."**

When AI can only live inside a computer, it's a tool. When it follows you onto a factory floor, into a car, into any place without a desk, it starts becoming a companion.

Kai9000 is an early example of that direction — it could work independently on a phone before "general agents on phones" became a trend. It has plenty to improve (sub-agents, the tool ceiling, persistence), and newer products have already surpassed it in several respects.

But "the first thing that actually made me carry AI with me" — that was it.

**Second, this isn't as expensive as you think.**

The only real required spend is the model subscription. The phone, the computer, the message groups, the open-source tools — all already owned. And as chips improve and open models shrink, even that last bill goes away.

Tech iterates; today's product may be replaced tomorrow. But credit belongs where it's due: **this is the app that first put AI in my pocket.**

---

## Appendix: two tools I built on this

The article mentions "using a chat group as a message bus" and "leads need a human to approve." Both became things that run.

### LeadPulse Guard — lead monitoring

**The problem is specific: leads from public channels get lost.**

Three entry points: comments, ad forms, email. Platform delivery is unreliable — I've measured a day where 5 leads arrived but only 2 were delivered — so it **also runs fallback polling**, compressing the worst case to under 10 minutes.

But its core design isn't "auto-reply." It's **refusal to execute**:

- The machine classifies intent, drafts a reply, decides whether one is warranted
- **Whether it sends is always a human decision**
- Approval goes through a chat card with three buttons: send / edit-then-send / ignore

Why? Because I've seen three failure modes in production:

1. **Machine loop** — an AI reply triggers its own webhook; two AIs argue all night
2. **Polite infinite loop** — customer says "thanks," AI replies, customer says "you're welcome"
3. **Confident fabrication** — customer asks for a spec; the model invents a plausible-sounding number

None of these are fixable with prompt engineering. **They're fixable by "the last step must be human."**

The architecture: **read-only security entry point + file-spool handoff with zero network boundary + human in the loop.** It runs on my phone and one small box, and it cost me nothing extra.

> Site: <https://leadpulseguard.com> ｜ Repo: github.com/yangke821115/lead-pulse-guard

### RoboBridge — the next piece

This is **a preview, not a product.** The repo is private and still being written.

The problem: I have Kai (phone), opencode (computer), and a chat group. They each have accounts and permissions, but **they can't talk to each other.**

Today the workaround is using the chat group as a relay and moving things by hand. RoboBridge aims to automate that — **letting different AI agents genuinely collaborate instead of working solo.**

There's a structural difference here that shapes the whole design:

**CLI agents can log in with a subscription account; chat-platform bots can only use API keys.**

That asymmetry drives the architecture.

One correction to a widespread misunderstanding: **a bridge only connects two interfaces — it cannot manufacture permission.** It doesn't produce quota; it connects what already exists.

> **So don't treat any "this saves money" as a conclusion.** The value of a bridge is collaboration, not arbitrage. Policy changes and the arbitrage is gone; the need to collaborate remains.

I'll write it up separately when it's done.

---

### How the three fit together

```
LeadPulse Guard   →  ground (catch leads, push to a human)
This article      →  method (how to run AI on a phone, how to not spend)
RoboBridge        →  connection (make multiple agents one team)
```

**This is about as far as one person gets alone.** Whether these three pieces assemble into a system that genuinely works is still being tested.

---

## License

MIT. Use it, change it.

---

*Every conclusion here comes from hands-on testing. Private server addresses, account identifiers, and customer information have been removed. Third-party project details were accurate at the time of writing and may change.*



 [看中文版](README.zh-CN.md)