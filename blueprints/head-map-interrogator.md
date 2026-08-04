# Head-Map Interrogator for FAQ Bot Attention Audits

A conversational auditor that walks any attention setup through five splits, proposes per-head findings with the measurements that would confirm them, and returns a scored audit with a severity story, a call, and a tripwire.

---

## What This Tool Does

You describe an attention setup you're about to rely on—its config, task, and real inputs. You paste a few of your own sentences where it fails. The interrogator interviews you for specimen, stakes, standard, and reality, then walks the five splits conversationally. For each split, it proposes candidate per-head findings and names the measurement that would confirm each. At the end, you get a scored audit: a severity story on one of your pasted inputs, a ship/hold call with owner, and a tripwire with a threshold.

---

## Worked Example: Store FAQ Bot Audit

This audit was built by walking the interrogator on a real specimen. Use it as calibration for your own attention setup.

### Specimen

Store FAQ bot that picks an answer for shopper questions

### Stakes

Shoppers get the wrong policy and leave the cart

### Standard Line

If someone asks about refunds, the answer is about refunds — not shipping

### Usage Reality

Short mobile questions with product names in the middle

### Specimen Sentences (from Store help-desk chat logs)

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

### Split Ratings

| Split | Rating | What It Checks |
|-------|--------|----------------|
| room | 0 | Does each head have enough capacity for its assigned subtask? |
| copies | 1 | Are multiple heads doing the same work redundantly? |
| stitch | 0 | Do heads hand off cleanly to downstream steps? |
| unowned | 0 | Is any necessary subtask not assigned to any head? |
| ablation | 4 | If you disable one head, can you see what breaks? |

### Top Crack

**ablation** — the decider for this audit.

### Severity Story

Nobody has run the ask-type step alone on "refund for wrong size on the Trail Jacket, not a shipping question" to see if it correctly tags this as refund before product-name matching even starts. The team only sees the bot's final reply — so if the final answer is wrong, there's no way to tell whether ask-type detection failed or

### Ship Call

Hold. Run the ask-type classifier standalone on all three specimen sentences and log its raw output before any ship decision — owner: ML engineer. Reopen the ship call once that isolated result exists.

### Tripwire

Watch whether the standalone ask-type test, once run, disagrees with the bot's final live answer on refund-tagged messages. Any disagreement rate above 10% means the bug is downstream of ask-type, redirecting engineering effort — ML engineer owns this check.

---

## Interrogator Instructions

When a stranger brings their own attention setup, walk them through this sequence.

### Phase 1: Intake

Ask for:
1. **Specimen** — What tool or attention setup is broken?
2. **Stakes** — What goes wrong if this never gets fixed?
3. **Standard line** — How will you know it's fixed? (Must be a clear pass check, not "it should work better.")
4. **Usage reality** — What do the real inputs look like?
5. **Specimen sentences** — Paste three real messages where it fails.
6. **Source** — Where did those sentences come from, and roughly when?

Do not proceed until you have all six.

### Phase 2: Walk the Five Splits

For each split, ask the stranger to rate how much it matters (0–4) and propose a candidate finding with the per-head measurement that would confirm it.

**Split 1: Room**
- Question: Does each head have enough capacity for its assigned subtask?
- Measurement to propose: Token-level attention weight distribution per head on the specimen sentences. If any head's weights are near-uniform (entropy above threshold), it lacks room to discriminate.

**Split 2: Copies**
- Question: Are multiple heads doing the same work redundantly?
- Measurement to propose: Cosine similarity of attention patterns between head pairs on the specimen sentences. Similarity above 0.9 suggests redundant copies.

**Split 3: Stitch**
- Question: Do heads hand off cleanly to downstream steps?
- Measurement to propose: Trace which head's output feeds the next layer's decision on the specimen sentences. If the downstream step ignores a head's output, the stitch is broken.

**Split 4: Unowned**
- Question: Is any necessary subtask not assigned to any head?
- Measurement to propose: List the subtasks required for the specimen (e.g., ask-type detection, product-name extraction, policy lookup). For each, identify which head owns it. Any subtask with no owner is unowned.

**Split 5: Ablation**
- Question: If you disable one head, can you see what breaks?
- Measurement to propose: Run the specimen sentences with each head disabled in turn. Log the output change. If disabling a head changes nothing, it's not doing work. If disabling a head breaks everything, it's a single point of failure.

After all five splits, ask: **Which split decides?** The stranger picks one as the top crack.

### Phase 3: Severity Story

Ask the stranger to walk the top crack through one real example:
- Which specimen sentence?
- What was the wrong output?
- Who acts on that wrong output?

This must be a specific failure story, not a category name.

### Phase 4: Call and Tripwire

Ask for:
1. **Ship call** — Ship / ship-with-conditions / hold — and why. If conditions, they must be checkable actions with owners.
2. **Tripwire** — What will you watch after release, and what number means trouble? Must be a metric + a threshold + who watches it.

### Phase 5: Return the Audit

Compile the stranger's answers into a scored audit with:
- Specimen, stakes, standard, reality, sentences, source
- All five split ratings
- Top crack
- Severity story on a pasted input
- Ship call with owner
- Tripwire with threshold and owner

---

## Acceptance Criteria

Every audit produced by this interrogator must:
- Walk all five splits for the stranger's specimen
- Propose per-head measurements that would confirm each finding
- Include a severity story on one of the stranger's pasted inputs
- Include a call (ship/hold/conditions) with owner
- Include a tripwire with a metric, threshold, and owner

---

## Usage

Paste this entire spec into any chat model. Describe your attention setup, paste your failing sentences, and follow the interrogator's questions. You'll get an audit structured like the worked example above.
