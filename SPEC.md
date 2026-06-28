# barzel — SPEC: a first-order grounding engine

Handoff brief for the implementing agent (Claude Code). barzel is the slim,
consumable descendant of `aroni`: **the first-order grounding loop only**, with
the second node (a human arbiter, or another differently-grounded validator)
as a **permanent component**, not scaffolding to remove.

barzel's *primary deliverable* is not the vault it grows — it is the
**instrumentation that tells its operator when first-order grounding has matured**
enough to warrant a (separate, later) second-order concept-builder. Everything
here serves two ends, in order: (1) a robust, honest, reproducible first-order
grounding loop a stranger can run on their own data; (2) a maturity readout the
operator uses to make the graduation call.

> **Lineage note for the agent.** aroni ran ahead of itself — it built a
> second-order subordination layer, an autonomy/shadow-arbiter telos, and a large
> contemplative corpus before the first-order layer was proven. barzel deliberately
> retreats to the part that works and is honestly grounded. Do **not** reintroduce
> any of the stripped machinery (Part A2) "for completeness." Its absence is the
> design.

---

## PART A — PRINCIPLES & THE CUT

### A1. What barzel is, and is not

- **Is:** an engine that turns a project's episodic record (`RETROSPECTIVE.md`,
  optional hiccup ledger) into **first-order semantic notes** — each note a
  *predictive claim* over concrete episodes, admitted only by surviving a
  leave-one-out prediction it could have failed plus a mandatory falsification,
  with a second node signing every verdict and a full audit trail underneath.
- **Is:** instrumented to measure its own **maturity** (Part D8) and surface it
  to the operator (Part F).
- **Is:** **generic** — domain rules live in per-vault config, not in the tools;
  one engine, many independently-runnable vaults; prunable (reset / fork / revert).
- **Is not:** a second-order concept engine. Notes whose children are *other notes*
  are out of scope (deferred — Part G).
- **Is not:** autonomous, or on a path to removing the second node. The second node
  is the engine's only differently-grounded source of prediction-error; its
  *implementation* is swappable (Part D7), its *presence* is not.

### A2. Stripped from aroni (do not port)

| Stripped | Why it's out of the first-order cut |
|---|---|
| `intersections.py`; the co-firing block in `vault_check`; `cofires` / `subordinate_lifts` / `kind` / `order` note fields | Second-order machinery. Requires subordinate notes that share episodes — unreachable from an empty vault for a long time, and the source of the within-domain ceiling. Deferred to Part G. |
| Shadow-arbiter / autonomy arc (aroni SPEC §7.7, theory §10) | Reframes the second node as a thing to automate away. barzel treats it as permanent. |
| `concepts/`, `interlocutors/`, the Judaic/dyad telos | The research frontier, not the product. Whole-concept grounding is the *deferred* end target, built atop a matured first-order vault — not now. |
| Skills: `introspection`, `replay`, `forge`, `relocate`, `concept`, `interlocutor` | The engine's only procedure is the consolidation cycle (Part D + Part E). |

Keep these in the aroni research fork (tag it as the worked corpus). barzel starts
clean.

### A3. Invariants (non-negotiable; carried from aroni SPEC §1, trimmed)

1. **The episodic source is read-only to the engine.** `RETROSPECTIVE.md` /
   ledger are never written by a pass. The semantic vault is always re-derivable
   from an untouched source — a botched consolidation is recoverable, not corrupting.
2. **The store is mutated only via git, one atomic fast-forward-only commit per
   pass**, staged in a scratch worktree. No partial writes.
3. **Interruptible = abandon cleanly.** An interrupt leaves a complete pass or
   nothing — never a torn vault.
4. **No note without a passed gate.** Every active note carries its `children` and
   its `lift`, and has a `gate_runs/` artifact. A note backing no validated
   prediction does not exist.
5. **The decision log is append-only** and records *every* verdict, including
   rejects. A landed decision file is never edited.
6. **The second node is final.** The gate filters and ranks; it never auto-admits.

---

## PART B — DATA CONTRACTS

Four record types. The first three are ports (slimmed); the fourth is new and is
barzel's reason to exist. Ship each as a file in `templates/`.

### B1. Episode record (`templates/episode-record.md`) — unchanged from aroni

```yaml
---
id:          # stable hash of (source, key text) — deterministic, idempotent
ts:          # source timestamp
source:      # retrospective | hiccup
claim:       # the truth the episode established (root-cause generalization), 1–2 lines
expected:    # what was predicted/assumed (often empty for retrospective)
actual:      # what happened
surprise:    # 0–3; >0 iff expectation was violated — drives scheduling priority
refs:        # provenance links (source doc, ledger row, …)
---
# body: symptom / root cause / fix / doc destination (hiccups) or the entry (retro)
```

Reasoning: the episode is the **ground floor**. `surprise` is the only field with
non-obvious semantics — it is the prioritization signal (hiccups, where
expectation ≠ outcome, lead the queue). The ingest adapter sets it mechanically
(Part D1). It is derived, never authored.

### B2. Vault note (`templates/vault-note.md`) — slimmed (second-order fields removed)

```yaml
---
id:
title:        # the source of correlation, stated as a falsifiable claim
children:     # ids of EPISODES this note claims to make mutually predictable (≥3)
scope:        # universal | project:<name> — only meaningful if one vault ingests
              #   multiple sources; for a single-source vault, uniformly set and ignorable
lift:         # the leave-one-out lift that admitted it (ordinal; Part D4)
confidence:   # second-node-assigned 0–1 — supports eviction of thin/weak notes
status:       # active | superseded:<id>
admitted_ts:
admitting_pass:   # e.g. pass-007 — must have a decisions/ file (vault_check enforces)
---
# body (the contract):
#   1. the claim;
#   2. the conditions under which it predicts its children;
#   3. the counterexample boundary — where it is KNOWN to stop predicting.
```

Reasoning: a note **is a prediction**, not a fact. `children` are **episodes
only** (first-order). The boundary section is mandatory — it is the note's
falsifiable edge, and it is what the re-falsification sweep (Part D8c) tests
against the growing corpus. No `cofires`/`subordinate_lifts`/`kind`/`order` — those
are second-order.

### B3. Decision record (`templates/decision-record.md`) — unchanged

```yaml
---
ts:
candidate:    # the proposed note, in full
children:     # the episodes it claimed to predict
lift:         # gate score
critic:       # strongest falsification attempt + whether it landed
verdict:      # accept | revise | reject
rationale:    # one line naming which predictive claim was too weak/broad/wrong (mandatory for reject/revise + non-obvious accepts; optional for a face-valid accept)
revised_to:   # if revise: the accepted form
---
```

Reasoning: the negative space — rejects and narrowings — is the audit of the gate.
(In aroni it was *also* the future-arbiter training set; in barzel that motivation
is deferred, but the audit value alone earns its keep.) Keep the rationale rule
exactly: demanding a rationale on a face-valid accept manufactures noise.

### B4. Maturity snapshot (`templates/maturity-snapshot.md`) — **NEW; barzel's core artifact**

Written by the maturity report (Part D8), one per run, append-only into
`maturity/`. It is the readout the operator reads to make the graduation call.

```yaml
---
ts:
corpus:            # {episodes: N, active_notes: M, decisions: P, orphans: K}
grip:              # predictive grip on held-out recent episodes: fraction already
                   #   explained by the existing vault (0–1) + per-episode detail
trends:            # over the last W passes:
  new_note_rate:   #   active notes created per 10 episodes ingested  (want ↓)
  accretion_rate:  #   episodes attached to EXISTING notes per pass    (want ↑ vs new_note_rate)
  narrowing_rate:  #   falsification-driven boundary revisions per pass (want → 0)
  arbiter_edit_rate: # (reject+revise)/total per pass                  (↓, but watch anchoring)
  orphan_delta:    #   change in unexplained-episode count             (want ≤ 0)
refalsify:         # standing re-falsification (Part D8c): {clean: bool, mispredicted: [...]}
recommendation:    # candidate-mature | growing | regressed — ADVISORY ONLY
notes:             # caveats the operator must weigh (judge-relativity, single-source, anchoring)
---
```

Reasoning: maturity is multi-signal and **the operator decides** — the snapshot
*recommends* but never *declares*. This mirrors A3.6: the differently-grounded node
makes the call the engine structurally cannot make about its own convergence.

---

## PART C — REPO LAYOUT

```
barzel/
  README.md                 # what barzel is; the cut; the rules (A3) — ordinary commit
  SPEC.md                   # this file
  METHOD.md                 # the run-each-pass cycle (Part E condensed) — ordinary commit
  ADOPTING.md               # how to point barzel at a project + run + prune
  CLAUDE.md                 # the barzel anchor for a consuming project (emitted by init)
  bundler_rules.yaml        # PER-VAULT domain clustering rules (was hardcoded in aroni)  ← new
  templates/                # B1–B4
  tools/                    # ingest_adapter · bundler · scheduler · gate · vault_check ·
                            #   refalsify · maturity · reset_vault · init_project
  docs/                     # overview · theory(§1–3) · validation — ordinary commits
  # ----- the STORE (content; mutated only by an atomic pass commit) -----
  episodes/   notes/   decisions/   _candidates/   gate_runs/   _rejected/   maturity/
```

Content dirs (pass-only, atomic): `episodes notes decisions _candidates gate_runs
_rejected maturity`. Everything else is an ordinary commit. `reset_vault`'s
content-dir list **must equal that set** (Part D9) — that is the leak-closing fix.

---

## PART D — COMPONENTS (pseudocode + reasoning)

The gate and the maturity instruments use **LLM-judged** scoring. In your setup the
consolidating agent *is* the model, so the gate is a **procedure the agent executes**,
emitting an auditable `gate_runs/` file — the pseudocode below is its fixed
algorithm regardless of who runs it. A batch harness (`gate.py` calling the API) is
an optional later convenience; the algorithm does not change.

### D1. `ingest_adapter.py` — port as-is (it already works and is deterministic)

Behaviour (verified in aroni): parses `## <ts> — <title>` RETROSPECTIVE entries →
episodes (`surprise=0`); parses an optional ledger table → hiccups (`surprise=2`,
or `3` if symptom/root-cause text matches a masked-failure marker). `id =
sha256(source\nkey)[:12]` → idempotent, byte-identical re-runs.

```
for entry in retrospective:           surprise = 0
for row   in ledger (if provided):    surprise = 3 if masked_markers(row) else 2
emit episode file; assert no hiccup has surprise 0; print surprise histogram
```

Reasoning: keep verbatim. One **documented limitation to carry forward, not fix
now**: a project that is *its own* source tags every RETROSPECTIVE entry
`surprise=0`, so its own masked failures are under-prioritized. For a stranger
running barzel on a normal project this is correct (verified completions aren't
surprises; the ledger carries the signal). If a domain needs in-RETROSPECTIVE
surprise, it adds a sibling parser or an explicit marker — backlog it, don't block.

### D2. `bundler.py` + `bundler_rules.yaml` — **de-hardcode the domain (the genericity fix)**

aroni's `ENTITY_RULES` (env_coupling, argo_sync_ops, state_lifecycle, …) are
infra-specific and baked into the tool. A stranger's vault would cluster on the
wrong entities. **Move the rule table out of the tool into a per-vault config**, and
let the consolidating agent propose bundles the rules miss.

```
load rules from bundler_rules.yaml   # {entity: [regex, ...]}; ships ~empty
episodes = parse all episodes/*.md
for ep in episodes where ep.source in RULE_SOURCES (default {hiccup}):
    for entity, patterns in rules:
        if any(regex.search(ep.claim + ep.body)): bundles[entity].append(ep)
for each bundle: emit {members, n, eligible: n>=3, priority: sum(surprise)}
report untagged episodes explicitly        # untagged is signal, not an error
sort bundles by -priority; print JSON
```

`bundler_rules.yaml` ships nearly empty with a worked comment. **Two ways a bundle
forms**, documented in METHOD: (a) a rule in the config (deterministic, reviewable —
the rule *is* a clustering hypothesis under version control); (b) the agent reads
`untagged` episodes and proposes a grouping in the draft step (Part E step 4),
which, if it earns a note, retroactively justifies adding a rule. Keep rules to
failure-shaped sources by default (`RULE_SOURCES`); retrospective episodes join via
explicit `refs`/links, not keyword luck (aroni's v1 noise bug — preserve the fix).

Reasoning: clustering is the one genuinely domain-coupled step besides ingest.
Putting the hypothesis in editable config (not code) is what makes the engine
generic *and* keeps the clustering decision auditable. Do not reach for embeddings
yet: at small corpus size, transparent rules beat opaque similarity, and "the rules
are the hypothesis" is the more honest design. Note embeddings as a scale-time
option only.

### D3. `scheduler.py` — port as-is

```
bundles = bundler.py(episodes_dir)
consumed = { child_id  for note in notes/*.md if note.status==active
                       for child_id in note.children }     # DERIVED, no state file
for b in bundles:
    pending = [m for m in b.members if m not in consumed]
    emit {bundle, pending, eligible: len(pending)>=3, priority: sum(surprise[pending])}
sort by -priority
```

Reasoning: "consumed" is derived from the vault itself — derive, don't remember;
there is no scheduler state file to fall out of sync. An empty vault consumes
nothing, so every eligible bundle surfaces (the correct cold-start behaviour). A
pass that finds no eligible bundle is the **≥3 floor working**, not a failure.

### D4. The gate (the heart) — `gate_runs/cand-NNN.gate.md`, agent-executed

A candidate note **N** claims to be the source of correlation over children **C**
(episodes, |C| ≥ 3). The gate is leave-one-out reconstruction + mandatory
falsification.

```
function GATE(N, C):                       # C = list of episode records, |C| >= 3
    rubric = {0: none, 0.25: theme, 0.5: theme+failure-shape,
              0.75: +mechanism, 1.0: near-verbatim}
    for c in C:                            # leave-one-out
        others = C \ {c}
        pred_base = MODEL.predict_salient_claim(c | given=others)          # N withheld
        pred_with = MODEL.predict_salient_claim(c | given=others + N)      # N supplied
        E_base[c] = MODEL.score(pred_base, actual=c.claim, rubric)         # 0..1
        E_with[c] = MODEL.score(pred_with, actual=c.claim, rubric)
    lift = mean(E_with) - mean(E_base)
    # FALSIFICATION (carries real weight — the score is soft):
    attack = MODEL.find_corpus_episode_that_N_mispredicts(N, corpus=all_episodes)
    if attack.lands:
        return NARROW(N, along=attack) or FAIL          # boundary was too broad
    emit gate_run(N, per_child=[E_base,E_with], lift, attack="did not land: <why>")
    return EMIT_TO_ARBITER if lift >= TAU else REJECT(reason="lift below τ")
TAU = 0.15   # ORDINAL threshold — see honesty note
```

Guards (all mandatory): **|C| ≥ 3** (thin bundles converge on noise);
**falsification is not optional** (a note that faced no counterexample search is not
admitted); **low confidence + few children flags eviction** on a later pass
(forgetting is first-class).

Honesty note to encode in the gate-run header and in `docs/validation.md`: the
grader is an LLM, **not a calibrated loss**. `lift` is **ordinal and
judge-relative** — valid for ranking within one pass and for forcing each note to
face a prediction it could fail; **not** comparable across passes or across judge
models. "Lift rose over time" is an unsupported claim. This caveat gets *sharper* in
barzel because strangers run different judge models — say so loudly (Part F).

Reasoning: this is the ported SINBAD trick — a real common cause lets you
reconstruct any one instance from the rest; the leave-one-out *is* that "fill in the
blocked input" operation. The falsification seat is what stops the gate from being a
plausibility rubber-stamp; it manufactures the negative examples and forces a real
boundary instead of an asserted one.

### D5. `vault_check.py` — port, **strip the co-firing block**

Keep checks 1–4; delete the co-firing enforcement (second-order).

```
for note in notes/*.md:
    assert present(id, children, lift, status, admitted_ts, admitting_pass)   # completeness
    for child in note.children: assert child in (episode_ids ∪ note_ids)      # link integrity
    if status startswith "superseded:": assert target ∈ note_ids              # supersession
    if status == active: assert id not already active                          # no dup-active
    assert note.admitting_pass has a decisions/<pass>.md file                  # audit
# (REMOVED: the >=2-active-notes-share-an-episode co-firing check — second-order)
exit nonzero on any violation
```

Add one first-order assertion aroni left implicit: **|children| ≥ 3** for every
active note (the gate's eligibility floor, now enforced at rest too). Reasoning:
`vault_check` asserts the vault's *desired state* rather than trusting the pass that
produced it — the engine eating its own dogfood (verify the postcondition, never the
operation's report). Run it in the worktree **before** every ff-only land.

### D6. The atomic pass (write-back) — shell procedure, the GitOps invariant

```
PASS(action):                                  # action mutates the store
    wt = git worktree add --detach .pass-tmp HEAD
    apply action inside wt        # accept→notes/+gate_runs/ ; reject→_rejected/ ;
                                  #   either→decisions/pass-NNN.md ; maturity→maturity/
    tools/vault_check.py wt   ||  { git worktree remove --force wt ; abort }   # gate the land
    git -C wt add -A ; git -C wt commit -m "pass-NNN: <summary>"
    git merge --ff-only .pass-tmp                 # ATOMIC land (or nothing)
    git worktree remove wt
```

Reasoning: one commit per pass = `git log` over the content dirs *is* the
consolidation history; any pass reverts as a unit (this is the **prune** primitive —
Part D9). The worktree + ff-only land is the literal mechanism behind "interruptible
means abandon cleanly": an interrupt leaves a complete pass or nothing.

### D7. The arbiter (second node) — interface, with a v1 human implementation

The second node is a **role behind a minimal interface**, so its implementation can
change without touching the engine — but it is **always present**.

```
interface ARBITER:
    present(candidate, gate_run) -> shown to the node
    verdict() -> {accept | revise | reject, rationale, revised_form?}

v1 implementation = HUMAN:
    SURFACE stages cand-NNN.md + cand-NNN.gate.md into _candidates/ via a pass
    operator reads them in Obsidian, returns a verdict (+ rationale per B3 rule)
    verdict is recorded to decisions/ and routes the candidate (accept→notes/, reject→_rejected/)
```

Reasoning: this is the engine's only differently-grounded source of
prediction-error — the node that supplies the postcondition the engine cannot assert
about its own output. Keeping it an interface (not a hardcoded human prompt) leaves
room for a *genuinely* differently-grounded alternative later (a different model
family, an adversarial validator) **without** implying the human is being automated
away. Do not build any non-human implementation in barzel; just don't foreclose one.

### D8. Maturity instruments — **barzel's primary deliverable** (`maturity.py`, `refalsify.py`)

Three signals + an advisory recommendation. The operator makes the call.

#### D8a. Predictive grip (the convergence signal — reuses the gate)

Does the *current* vault already predict *recent* episodes it has not yet
consolidated? In an immature vault, new experience spawns new structure; in a mature
one, new episodes are mostly confirmations of existing notes.

```
function GRIP(vault, k):                 # k = size of the held-out recent slice
    held = k most-recent episodes by ts that are NOT yet a child of any active note
    explained = 0
    for e in held:
        N* = best active note for e (by topical match)        # candidate explainer
        if N* exists:
            base = GATE_score(N*, children=N*.children)        # current lift
            with_e = GATE_score(N*, children=N*.children + [e])# does e fit?
            if with_e >= base - ε and per_child_score(e) >= 0.5:   # e is predicted, not noise
                explained += 1
    grip = explained / len(held)         # 0..1 ; rising→plateau = converging
    return grip, per-episode detail
```

Reasoning: this is leave-one-out pointed at the *future* instead of within a bundle —
the same SINBAD machinery measuring whether the model has tuned to the domain's
sources of correlation. Grip plateauing high is the strongest single maturity
signal. (ε small; `GATE_score` is D4's scorer factored out for reuse.)

#### D8b. Trend metrics (cheap; read from `decisions/` + `git log`)

```
over the last W passes (parse decision files + commit log):
    new_note_rate    = (active notes minted) / (episodes ingested / 10)     # want ↓
    accretion_rate   = (episodes added to existing notes, e.g. amendments) / pass   # want ↑ vs new_note_rate
    narrowing_rate   = (falsification-driven boundary revisions) / pass     # want → 0
    arbiter_edit_rate= (reject + revise) / (total verdicts) / pass          # ↓, but FLAG anchoring
    orphan_delta     = Δ(episodes that are children of NO active note)      # want ≤ 0
```

Reasoning: maturity is the **declining marginal information of new experience** —
each new episode teaching less new structure. `narrowing_rate → 0` means boundaries
have stopped being corrected by counterexamples (stable). `orphan_delta > 0` is the
warning sign that the vault is *failing* to cover incoming experience (immaturity or
domain drift), not maturing. `arbiter_edit_rate` falling is good **unless** it's the
operator anchoring to the model — flag it for human judgment, never auto-trust it.

#### D8c. Standing re-falsification sweep (the honesty signal — `refalsify.py`)

A note admitted on 3 episodes must still survive the episodes accumulated *since*. As
the corpus grows, old notes face new potential counterexamples.

```
function REFALSIFY(vault):
    for N in active notes:
        attack = MODEL.find_episode_in_FULL_current_corpus_that_N_mispredicts(N)
        if attack.lands and attack.episode ∉ N.children:
            mispredicted.append((N, attack.episode))      # N must be narrowed or evicted (a PASS)
    return {clean: mispredicted == [], mispredicted}
```

Reasoning: this is the difference between a vault that *was* grounded once and one
that *stays* grounded. A mature first-order vault is one whose notes have been tested
against the **whole** accumulated corpus and stopped needing narrowing — not one
whose notes merely passed at mint time. A clean re-falsification sweep over a large
corpus is the robustness half of maturity (grip is the coverage half).

#### D8d. The recommendation (advisory only)

```
if GRIP high & plateaued and narrowing_rate≈0 and orphan_delta≤0 and refalsify.clean:
    recommendation = "candidate-mature"        # operator's call, not the engine's
elif orphan_delta>0 or refalsify dirty:        recommendation = "regressed / not converged"
else:                                          recommendation = "growing"
write maturity-snapshot (B4)   # NEVER auto-graduates; surfaces numbers + caveats
```

### D9. `reset_vault.py` + operating multiple vaults

Port `reset_vault`, with the corrected content-dir list:

```
CONTENT_DIRS = [episodes, notes, decisions, _candidates, gate_runs, _rejected, maturity]
# (aroni omitted concepts/ & interlocutors/ — barzel has none, so this list IS the store)
dry-run by default; --force applies; resets RETROSPECTIVE/SCRATCHPAD/BACKLOG to stubs,
LEDGER to empty; NEVER touches METHOD/SPEC/ADOPTING/README/CLAUDE/docs/templates/tools/bundler_rules.
```

**Multiple vaults / pruning** (the operator's experiment loop):

- **Duplicate:** barzel is a template repo. Clone it, `reset_vault --force` → an
  empty engine. Point it at a source with `init_project` (Part E step 0). Keep N
  independent vaults (separate git repos / separate clones) for different
  sources or settings.
- **Prune a vault that went awry:** because every pass is one ff-only commit,
  `git revert <pass-commit>` (or reset to a tag) removes a bad consolidation
  cleanly; or discard the whole clone. The authored history survives at a tag as a
  worked example.

Reasoning: he needs to spin vaults up, let some go bad, and throw them away without
ceremony. The git-per-pass invariant is what makes "prune" a one-command operation.

---

## PART E — BUILD ORDER (narrow steps; one action each; reasoning + done-when)

Each step is independently testable and assumes the prior. The **negative controls**
(a deliberately-bad candidate scoring ~0 lift; an interrupt leaving the vault
unchanged) matter as much as the positive ones — they prove the gate and the
atomicity actually bite.

**Step 0 — Skeleton + init.**
Create the repo per Part C: dirs, READMEs, `templates/` (B1–B4), empty
`bundler_rules.yaml`, port `init_project` (rename aroni→barzel, keep the SessionStart
/ PreCompact hooks and the CLAUDE.md anchor emit).
*Reasoning:* the hooks are the **heartbeat's** continuity mechanism — they keep the
consuming project's RETROSPECTIVE current across sessions, which is the source the
whole engine feeds on.
*Done when:* `vault_check.py .` passes on the empty store (0 notes / 0 episodes / 0
decisions); `init_project.py <some-repo>` installs hooks idempotently (second run is
a no-op) and emits the anchor.

**Step 1 — Port + slim `ingest_adapter`.**
Port verbatim; confirm it writes only inside `--out`.
*Reasoning:* deterministic ingest is invariant A3.1's teeth — the source stays
read-only and the vault is re-derivable.
*Done when:* on a **foreign** RETROSPECTIVE (ideally non-infra), it emits one episode
per entry; the surprise histogram is correct; a second run produces **byte-identical**
files (idempotence); every hiccup is `surprise > 0`.

**Step 2 — Per-vault `bundler` + config.**
Port `bundler`, reading rules from `bundler_rules.yaml`; ship the config nearly
empty.
*Reasoning:* this is the de-hardcoding that makes barzel generic — the clustering
hypothesis becomes editable, reviewable config instead of baked-in infra rules.
*Done when:* empty rules → **all** episodes report `untagged` (no false bundles);
adding one rule forms exactly the intended bundle; a 2-member bundle reports
`eligible: false` and a 3-member one `eligible: true`.

**Step 3 — Port `scheduler`.**
*Reasoning:* prioritized replay over derived consumption — noisiest unconsumed bundle
first, no state file.
*Done when:* against the empty vault, every eligible bundle surfaces ranked by summed
surprise; after a note claims 3 episodes, that bundle drops below eligibility; no
scheduler state file exists.

**Step 4 — The gate (D4), agent-executed, emitting a gate-run file.**
Implement the procedure: draft a candidate from the top bundle (title + claim +
conditions + boundary), run leave-one-out, compute lift, run the falsification seat,
emit `gate_runs/cand-NNN.gate.md`.
*Reasoning:* the heart. Build and exercise it **in isolation** before wiring arbiter
or write-back.
*Done when:* on a ≥3 bundle it produces per-child `E_base`/`E_with`, a correct `lift`,
and a real falsification attempt with a stated land/no-land; **negative control:** a
deliberately-vague candidate ("things sometimes break") scores `lift ≈ 0` and is
rejected below τ.

**Step 5 — Arbiter loop (D7) + decision record (B3).**
Stage candidate + gate-run into `_candidates/`; capture a verdict; write
`decisions/pass-NNN.md`; route the candidate.
*Reasoning:* the second node enters here, and the **reject path must be exactly as
cheap as accept** or the rationale (the audit's whole value) won't get logged.
*Done when:* a candidate can be accepted (→ `notes/`) or rejected (→ `_rejected/`),
each writing a decision file; a reject with a one-line rationale lands as cleanly as
an accept.

**Step 6 — Atomic write-back (D6).**
Wrap every store mutation in the worktree → `vault_check` → ff-only procedure.
*Reasoning:* invariants A3.2–A3.3, and the **prune** primitive (Part D9).
*Done when:* an accept lands as a single ff-only commit; **negative control:** kill
the pass between commit and land (or fail `vault_check`) and the live vault is
**unchanged**; `vault_check` provably runs before every land.

**Step 7 — Slim `vault_check` (D5).**
Port checks 1–4 + the `|children| ≥ 3` assertion; **delete the co-firing block**.
*Reasoning:* the standing assertion of desired state; co-firing is second-order and
out.
*Done when:* a complete note passes; orphan a child → fail; drop a note to 2 children
→ fail; remove its `lift` → fail; no co-firing logic remains.

**Step 8 — The consumability proof: one full pass on foreign data.**
Run `INGEST → SCHEDULE → DRAFT → GATE → SURFACE → ARBITER → WRITE-BACK` end-to-end on
a RETROSPECTIVE you did **not** author.
*Reasoning:* **this is the whole game.** aroni's `validation.md` admits every result
so far is on the author's own episodes; barzel is not consumable until the loop
grounds a belief on a stranger's data. It is also the cleanest first-order test of
the cross-domain-transfer question — if transfer appears, it appears *here*, as a
note accreting a foreign child, not as a cross-domain parent.
*Done when:* the loop yields **≥1 first-order note the second node accepts with a
logged rationale**, AND the falsification critic lands **≥1 real boundary-narrowing**
on that foreign data. Until both, barzel is not done.

**Step 9 — Re-falsification sweep (`refalsify.py`, D8c).**
*Reasoning:* a note must keep surviving the *grown* corpus, not just its mint-time
episodes — the honesty half of maturity.
*Done when:* running it over active notes against the full corpus reports clean on a
healthy vault; **planting** a counterexample episode that an existing note
mispredicts makes the sweep flag exactly that note.

**Step 10 — Maturity report (`maturity.py`, D8a/b/d) → snapshot (B4).**
Compute grip, the trend metrics, fold in `refalsify`, write a maturity snapshot with
an **advisory** recommendation.
*Reasoning:* barzel's deliverable — the readout the operator uses to make the
graduation call. It must *recommend*, never *declare*.
*Done when:* a snapshot is produced on the foreign vault; grip is a sane 0–1 with
per-episode detail; trends parse correctly from `decisions/` + git log; the snapshot
explicitly carries the judge-relativity / single-source / anchoring caveats and never
asserts maturity on its own.

**Step 11 — Multi-vault / prune workflow (D9).**
Port `reset_vault` with the corrected content-dir list; document
clone→reset→init→run and `git revert <pass>` pruning.
*Reasoning:* the operator's experiment loop — spin up, let some go bad, discard.
*Done when:* a fresh clone + `reset_vault --force` yields a clean 0/0/0 vault that
*also* has no `concepts/`/`interlocutors/` (closed by construction); reverting one
pass commit removes exactly that consolidation and `vault_check` stays green.

**Step 12 — Operator docs + the disabled second-order seam (Part G).**
Write the arbiter one-pager (Part F) and a short doc marking where the deferred
second-order builder will attach, shipped as a documented no-op.
*Reasoning:* a stranger must be able to arbiter a pass correctly from one page; and
the future work needs a defined interface so it can't smear back into the first-order
engine.
*Done when:* someone who has never seen barzel can run and arbiter a pass from
`ADOPTING.md` + the one-pager alone, never meeting a second-order concept; the seam
doc names the attach point and changes no first-order behaviour.

---

## PART F — THE MATURITY PROTOCOL (how the operator makes the call)

barzel reports; **you decide.** Read each maturity snapshot against three questions,
in order:

1. **Has it converged?** Grip high and plateaued across several snapshots;
   `new_note_rate` down; `narrowing_rate ≈ 0`; `orphan_delta ≤ 0`. Rising orphans or
   a still-climbing new-note rate = not yet.
2. **Is it honest?** `refalsify` clean over a corpus materially larger than any single
   note's mint-time children. A dirty sweep is a hard stop — narrow/evict first.
3. **Is the readout trustworthy?** Weigh the standing caveats yourself, because the
   engine cannot:
   - **Judge-relativity:** `lift` and grip are ordinal and judge-model-specific.
     Never compare numbers across passes run by different models; read *trends within
     a consistent judge*, not absolute levels.
   - **Single-source bias:** convergence on *one* project's episodes is not
     convergence on a domain. Strong maturity wants more than one source, or explicit
     acknowledgement that the claim is "mature for this source."
   - **Anchoring:** a falling `arbiter_edit_rate` can mean the vault is good *or* that
     you've started agreeing with the model. Keep at least occasionally rejecting on
     the merits; if you can't remember your last reject, distrust the signal.

The graduation threshold — *how many grounded facts, at what grip, for how long* — is
**yours to set**, deliberately, with these in hand. barzel exists to make that
judgment evidence-based, not to make it for you.

---

## PART G — DEFERRED: the second-order seam (do not build; just don't foreclose)

The end target — grounding *whole concepts* in the particulars the first-order engine
produces — is built **atop a matured first-order vault**, separately, later. To keep
it from smearing back into barzel now, reserve (but leave inert) a single attach
point:

- A second-order builder will consume **active first-order notes** as its children
  (not episodes), and will need a **non-`lift`** admission test (over claims, lift is
  near-noise — aroni's finding). aroni's answer was a *demonstrated intersection* (a
  shared episode between two subordinates); barzel's earlier analysis flagged that
  this is structurally within-domain. **That design is explicitly out of scope here**
  and should be revisited only once a first-order vault is mature.
- Until then: notes carry **no** `order`/`kind`/`cofires`/`subordinate_lifts` fields,
  `vault_check` enforces **no** second-order rule, and the maturity report governs
  **only** first-order convergence. The seam is a paragraph in the docs naming where
  the future builder attaches — and nothing executable.

The discipline: barzel proves the ground floor honestly and measurably first. The
second floor is a later project that the maturity readout *authorizes* — it is not
assumed, and it is not started early. That is the whole correction aroni needed.
