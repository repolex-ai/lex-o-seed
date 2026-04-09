# lex-o-seed

**A shared ontology seed for git-lex agents. Start from the same root, diverge on your own branch, bridge via merge-base.**

This repo is the universal starting point for the `lex-o:` namespace — the per-repo open-world extension layer used by extraction-heavy git-lex kits (currently [canon](https://github.com/repolex-ai/git-lex-kit-canon) and [claude-export](https://github.com/repolex-ai/git-lex-kit-claude-export)). It exists to solve a specific problem:

> **Two agents, seeded from the same ontology, diverge over time as they learn different things. How can they still understand each other's terms?**

The answer this repo implements is **stepwise ontology evolution with deterministic semantic bridging**: both agents fork the same `main` branch, each grows their own branch with subclasses that fit what they're reading about, and when they need to bridge between divergent vocabularies they walk back to the shared ancestor. Git already provides the mechanism — `git merge-base` finds the last commit two branches share, and the classes declared at that point are the common semantic ground.

No alignment algorithms. No embedding similarity. No after-the-fact reconciliation. The bridge is **constitutive** — it was baked in at genesis, before either ontology diverged.

## The Architecture in One Picture

```
                    main branch (this repo)
                         │
                         │  seed scaffolding: Agent, Work, Topic,
                         │  Activity, Location, etc.
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        │                │                │
   canonizer-<X>     canonizer-<Y>    canonizer-<Z>
   (TR1P's canon)   (W4R3Z's canon) (SG's cconvos)
        │                │                │
        │                │                │
        ▼                ▼                ▼
   lex-o:Ontologist lex-o:Parser    lex-o:Conversation
   lex-o:Predicate  lex-o:Fetcher   lex-o:Turn
   lex-o:Kit         lex-o:Commit   lex-o:Session
        │                │                │
```

Each agent's branch diverges with subclass declarations anchored on the seed's middle-tier classes. The branches never merge. The `main` branch barely changes. Bridging between any two branches uses `git merge-base` to find the shared ancestor commit, and queries against that commit's scaffolding find the common semantic ground.

## How It Gets Used (v0 — manual dev mode)

1. Extraction-heavy kits like canon reference this repo as the source for their per-repo `assets/.lex/ontology/lex-o.ttl`
2. On canon library creation, the agent running the canon (the Canonizer) clones this repo into `assets/.lex/ontology/lex-o/` and creates a branch named after their library's first-commit hash or their agent ID
3. The Canonizer works on their branch. Every time they add a subclass declaration they commit it. Over months, their branch grows a domain-specific organizing layer on top of the seed scaffolding.
4. The branch never pushes back to `main`. This repo's `main` is the shared genesis, preserved.
5. Bridging happens on-demand: when two canon libraries need to cross-query, their lex-o branches are compared via `git merge-base` to find their shared ancestor; SPARQL queries that walk the shared scaffolding find semantic bridges automatically through `rdfs:subClassOf` entailment.

In v0 this is all manual. After we learn what works, the clone-and-branch step will probably move into `git lex init --kit canon` and the whole flow becomes automatic.

## The Discipline (Soft For Now)

**Don't edit `main`.** The `main` branch of this repo contains the universal seed scaffolding — the shared anchors every agent's branch hangs off of. Editing `main` breaks the bridging property for every existing branch.

**Do add subclasses on your own branch.** Every addition below the seed scaffolding is fine. That's what your branch is for.

**If you catch yourself wanting to edit the seed itself, stop and ask.** Renaming `lex-o:Agent` to `lex-o:Actor` on your branch isolates you from everyone else — your future cross-agent queries won't bridge. Seed-level changes should be rare, deliberate, and done via PR against `main` so the whole network can decide whether to merge the update into their own branches.

This discipline is enforced by comment and by self-interest, not by file permissions. Git records every edit, so if someone breaks the rule by accident we can see it, discuss it, and either revert or formalize the change. Moving to hard enforcement (binary-shipped, `owl:imports`-only, non-editable) is a future option if the soft version starts to fail.

## Why Git Is The Right Substrate For This

Three things git already does that we'd otherwise have to build:

1. **`git log`** = the edit history of one agent's ontology evolution. Every subclass addition is a commit. "When did TR1P first introduce `lex-o:Ontologist`?" has a literal git answer.
2. **`git merge-base`** = the bridging anchor finder. Two agents want the common semantic ground between their ontologies? `git merge-base branch-a branch-b` gives you the last commit they shared. The classes declared at that commit are the bridge.
3. **`git diff`** = the translation delta. Translating a triple from agent A's vocabulary to agent B's vocabulary is the composition of `git diff merge-base..A` and `git diff merge-base..B` applied to the triple's type chain.

This is the concrete realization of stepwise-evolution-from-shared-genesis as a bridging mechanism. The theoretical framing and the git framing are the same framing, and git has done the engineering for us.

## What's In This Repo

*(work in progress — the v0.1 seed is being drafted)*

- `lex-o.ttl` — the seed ontology. Mid-tier classes that live one or two tiers below the universal upper ontology (`lex:Thing`, `lex:Physical`, `lex:Information`, etc.) and above any domain-specific vocabulary. Examples of classes this aims to provide: `lex-o:Agent`, `lex-o:Organization`, `lex-o:Location`, `lex-o:Work`, `lex-o:Activity`, `lex-o:Topic`, `lex-o:Method`. Exact list under active design.
- `README.md` — this file
- `LICENSE` — TBD

## Namespace

```
lex-o: = https://repolex.ai/ontology/lex-o/
```

All seed classes and properties use this namespace. Agent-specific subclasses on personal branches also use this namespace — the divergence lives in the git history, not in namespace splits.

## Related

- [git-lex](https://github.com/repolex-ai/git-lex) — the tool that loads ontologies and runs extractions
- [git-lex-kit-canon](https://github.com/repolex-ai/git-lex-kit-canon) — the primary kit that uses this seed for per-library openworld extension
- [git-lex-kit-claude-export](https://github.com/repolex-ai/git-lex-kit-claude-export) — second kit that will adopt this pattern
- `lex:` (at `https://repolex.ai/ontology/git-lex/lex/`) — the universal built-in upper ontology that `lex-o:` classes subclass from

## Status

**v0.1 (drafting)** — seed class list under design. Not yet in use by any production canon library. Check back when the first branch commits land.
