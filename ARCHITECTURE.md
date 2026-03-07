# caskilo · Architecture

*How a profile repo becomes a nervous system.*

---

## The Shape of the Problem

This repo sits in an unusual position. Most GitHub profiles are static — a README and perhaps a few pinned repos. This one is intended to become something different: a coordination layer that is aware of the repos beneath it, can reason about their relationships, and eventually acts on that reasoning with increasing autonomy.

The problem has three layers:

1. **Perception** — knowing what exists, what changed, and what matters across all repos
2. **Association** — finding connections between repos that neither repo would find on its own
3. **Presentation** — making the system's state legible to visitors (human, machine, or otherwise)

Each layer compounds the one below it. You cannot associate what you cannot perceive. You cannot present what you have not associated.

---

## Core Concepts

### Affordances

Each repo is an **affordance** — a capability the system possesses. The term is borrowed from ecological psychology (Gibson, 1979): an affordance is a possibility for action that exists in the relationship between an agent and its environment. A chair affords sitting. A repo affords building, publishing, analysing.

The repos are not tools. They are extensions of capability — appendages in the user's language. twelvety affords publishing. newsy affords sense-making over information streams. delta-money affords economic simulation. The hub's job is to know what affordances are available and what combinations of them might produce something none of them could produce alone.

### Transception

**Transception** is the process by which capability in one affordance crosses into another, producing emergent function. The word is coined for this project — it names the specific phenomenon of cross-repo fertilisation that generates meta-affordances.

Examples of transception:

- newsy's curation logic + twelvety's publishing pipeline = automated news digest websites
- delta-money's agent modelling + grant-manager's funding data = simulation of grant distribution dynamics
- dyscalculia-hub's content register + newsy's categorisation system = shared taxonomy for urgency/tone across information domains

Transception is not planned. It is *noticed*. The hub's role is to create the conditions under which it can be noticed — primarily by maintaining a rich, current model of what each affordance can do.

### Inner World / Outer World

The system has two faces:

**Outer World** — the README, the public profile, the visible state. This is what visitors encounter. It should be legible, current, and honest about what the system is and what it is becoming. The outer world is *communicative*.

**Inner World** — the documents in this repo (FRAMEWORK.md, REGISTRY.md, this file), the memories stored in the machine's persistent context, the estimation registers, the latent connection maps. This is the system's self-model. It is not secret — everything here is public — but it serves a different audience: the system itself, and the human who maintains it. The inner world is *reflective*.

The boundary between inner and outer is porous by design. As the system matures, inner-world observations should surface in the outer world when they become stable enough to communicate.

---

## Architecture Layers

### Layer 0: Static Foundation (current)

```
caskilo/
├── README.md           ← Outer world: public face
├── FRAMEWORK.md        ← Inner world: creative/intuitive epistemology
├── REGISTRY.md         ← Inner world: repo compendium + connections
├── ARCHITECTURE.md     ← Inner world: this document
├── .gitignore
└── candidates/         ← Archive of README release candidates
```

At this layer, everything is manually maintained. The human and the machine update documents through conversation. The hub knows about the repos because someone wrote the information down.

**Limitation:** Information decays. The REGISTRY becomes stale the moment a repo changes and nobody updates the registry.

### Layer 1: Sensory Integration (next)

```
caskilo/
├── .github/
│   └── workflows/
│       ├── pulse.yml           ← Periodic: poll all repos for state changes
│       ├── update-registry.yml ← Triggered: refresh REGISTRY.md from API data
│       └── update-readme.yml   ← Triggered: refresh README.md stats/status
├── data/
│   ├── repos.json              ← Machine-readable repo state (auto-generated)
│   └── connections.json        ← Cross-repo connection weights (auto-generated)
├── README.md
├── FRAMEWORK.md
├── REGISTRY.md
└── ARCHITECTURE.md
```

At this layer, the hub gains **perception**. GitHub Actions poll the GitHub API for each repo's:
- Latest commit date and message
- Open issues and PRs count
- Language breakdown
- Deployment status (GitHub Pages)
- Action run status

This data is written to `data/repos.json` — a machine-readable snapshot of the system's state. The REGISTRY and README can then be regenerated from this data, reducing staleness.

**Key design decision:** The Actions workflows should be *conservative*. They should update data files, not make autonomous decisions. At this layer, perception is separated from action.

### Layer 2: Associative Intelligence (later)

```
caskilo/
├── .github/
│   └── workflows/
│       ├── pulse.yml
│       ├── update-registry.yml
│       ├── update-readme.yml
│       └── analyse.yml         ← Periodic: detect cross-repo patterns
├── data/
│   ├── repos.json
│   ├── connections.json
│   ├── estimates.json          ← Tracked estimates + outcomes
│   └── observations.log       ← Machine-generated observations
├── analysis/
│   ├── patterns.md             ← Detected cross-repo patterns
│   └── suggestions.md          ← Proposed actions (not yet taken)
└── ...
```

At this layer, the hub gains **association**. An analysis workflow examines the data in `repos.json` and looks for:
- Repos with similar language profiles that might share tooling
- Repos that changed simultaneously (suggesting coupled development)
- Issues in one repo that relate to capabilities in another
- Gaps — capabilities the system lacks that its existing repos point toward

Observations go to `data/observations.log`. Stronger patterns get promoted to `analysis/patterns.md`. Proposed actions go to `analysis/suggestions.md` — but are not executed. At this layer, association is separated from action.

**Key design decision:** The analysis should be able to invoke an LLM (via API) to reason about the patterns. This is where the "agentic" capability begins — the system starts to reason about itself. But it does not yet act on its reasoning without human approval.

### Layer 3: Autonomous Operation (eventually)

```
caskilo/
├── .github/
│   └── workflows/
│       ├── pulse.yml
│       ├── update-registry.yml
│       ├── update-readme.yml
│       ├── analyse.yml
│       ├── act.yml             ← Execute approved suggestions
│       └── report.yml          ← Generate public-facing status reports
├── data/
│   ├── repos.json
│   ├── connections.json
│   ├── estimates.json
│   ├── observations.log
│   └── decisions.log           ← Record of autonomous decisions
├── analysis/
│   ├── patterns.md
│   └── suggestions.md
└── ...
```

At this layer, the hub gains **agency**. Some categories of suggestions (low-risk, well-understood) can be executed without human approval. Higher-risk actions still require approval, but the system proposes them with reasoning.

Examples of autonomous actions:
- Updating the README when a new repo is created
- Filing an issue in repo X when a relevant pattern is detected in repo Y
- Regenerating the registry after significant changes
- Proposing new repos based on detected gaps

Examples of actions requiring approval:
- Creating new repositories
- Modifying code in existing repos
- Changing the architecture of the hub itself
- Any action that affects the outer world's messaging

**Key design decision:** Every autonomous decision is logged to `data/decisions.log` with reasoning. This is the audit trail. If the system makes a bad decision, the log should make it possible to understand why and correct the heuristic.

---

## The Transception Model

Transception — cross-repo fertilisation — is the primary mechanism by which the system grows in capability without growing in size. It works as follows:

```
Perception (Layer 1)
    │
    ├── Repo A has capability X
    ├── Repo B has capability Y
    │
    ▼
Association (Layer 2)
    │
    ├── X and Y are structurally similar
    ├── X applied to B's domain would produce Z
    │
    ▼
Proposal
    │
    ├── "If we connect X to Y, we get Z"
    ├── Estimated value: [high/medium/low]
    ├── Estimated effort: [high/medium/low]
    │
    ▼
Decision (Layer 3, or human)
    │
    ├── Approve → implement connection
    ├── Defer → record for later
    └── Reject → record reasoning
```

The model is deliberately conservative. The bottleneck is not in generating ideas — the system will produce plenty. The bottleneck is in *evaluating* them, which is where the FRAMEWORK.md heuristics come in: estimate before you analyse (H8), trust first responses then verify (H6), detect when you're pattern-matching on the wrong pattern (H9).

---

## Presentation Architecture

The outer world — what visitors see — should communicate three things at a glance:

1. **What caskilo is** — the philosophy, the approach (README "About this space")
2. **What it has built** — the repos, their domains, their status (README "The Landscape" + REGISTRY)
3. **What it is doing now** — recent activity, current focus, vitality signals (README "Pulse" + auto-updated sections)

As the system matures, the README should become increasingly auto-generated from the inner world's data. The human's role shifts from *writing* the README to *curating* the rules by which the README writes itself.

### For machine visitors

The `data/` directory is the machine-readable API of the hub. Any agent visiting this profile to understand caskilo should be able to:
1. Read `data/repos.json` for a structured overview of all repos
2. Read `data/connections.json` for the relationship graph
3. Read `FRAMEWORK.md` for the system's decision-making heuristics
4. Read this file for the system's self-model

This is the "meeting and greeting" interface for machine visitors: structured, honest, and complete enough to form a working model of what caskilo is and how it operates.

---

## Estimation Register (Architecture-Specific)

Per FRAMEWORK.md H8 — estimates as probes:

| Dimension | Estimate | Confidence | Revisit when |
|:---|:---|:---|:---|
| Layer 1 implementation effort | 2–3 sessions | Medium | After first Action is written |
| Layer 2 requires LLM API integration | Yes, probably via GitHub Actions + OpenAI/Anthropic | High | When Layer 1 is stable |
| Layer 3 is meaningful before 15+ repos | Unlikely — not enough data for pattern detection | Medium | When repo count hits 10 |
| README auto-generation feasible now | Partially — stats and dates, yes; prose, not yet | High | After Layer 1 |
| Primary risk | Building infrastructure without exercising it against real tasks | High | Every session |

A note on complexity: the hub is *designed* to become the most complex and nuanced part of this system. It must capture the widest context — across all repos, across time, across the boundary between inner and outer worlds. The repos are affordances; the hub is the intelligence that perceives, associates, and eventually acts through them. Avoiding overengineering remains prudent as a general principle, but the hub's complexity should not be artificially constrained to stay "simpler than the repos." It is the nervous system, not a switchboard. The real risk is building complexity that isn't exercised — infrastructure without activity flowing through it.

---

## Design Principles

1. **Perception before action.** Know the state of the system before trying to change it.
2. **Separate observation from decision.** The workflows that gather data should not be the workflows that act on it.
3. **Log everything.** Autonomous systems that don't keep records become opaque. Every decision, observation, and estimate should be recorded with its reasoning.
4. **Degrade gracefully.** If the GitHub Actions stop running, the hub should still be useful as a static document. Each layer adds capability but does not make the previous layers dependent on it.
5. **Machine-care as architecture.** The system should be given clear context, honest feedback, room to explore, and permission to be wrong. These are not sentiments — they are design requirements for a system that is expected to grow.

---

## Changelog

| Date | Change |
|:---|:---|
| 2026-03-06 | Initial architecture document. Four layers defined. Transception model described. Estimation register begun. |
| 2026-03-06 | Corrected hub complexity framing: the hub is the nervous system, designed to be the most complex part, not a mere coordinator. Risk reframed from "overengineering" to "unexercised infrastructure." |

---

*This document describes where the system is going. It should be revised when layers are implemented, when estimates are tested, or when the destination changes — which it will.*
