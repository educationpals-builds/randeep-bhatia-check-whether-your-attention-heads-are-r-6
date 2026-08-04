# METHOD: The Five Principles Behind the Head-Map Interrogator

This file defines the five principles that guide how the interrogator walks through any attention setup audit. These principles structure the conversation — but the letters of the framework never appear in the tool's output to strangers.

---

## The Five Principles

### P — Partition the Space

Each attention head should own a distinct slice of the representation space. When auditing the Store FAQ bot, this means asking: does the ask-type head (refund vs shipping vs cancellation) occupy a different subspace than the product-name head (Nova Buds, Trail Jacket)?

If two heads collapse into the same region, one will dominate. The FAQ bot's failure — latching onto "Nova Buds" instead of "refund" — suggests the product-name signal may be bleeding into space that should belong to ask-type detection.

**Per-head measurement:** Cosine similarity between head output vectors on refund questions that contain product names. If heads partition cleanly, similarity should be low.

---

### R — Run in Parallel

Heads should process their patterns simultaneously, not sequentially depend on each other's output within the same layer. For the FAQ bot, the ask-type classifier and the product-name matcher should each do their job independently before any combination happens.

The current failure mode — "refund for wrong size on the Trail Jacket, not a shipping question" getting a shipping answer — may indicate that product-name matching runs first and overwrites ask-type signal before it can contribute.

**Per-head measurement:** Activation timing analysis. Does the ask-type head fire with full confidence before product-name head output is available, or does it wait?

---

### I — Individuate the Pattern

Each head should specialize in one recognizable pattern. The ask-type head should fire on "refund," "return," "cancel," "wrong size" regardless of what product name appears. The product-name head should fire on "Nova Buds," "Trail Jacket" regardless of what the shopper is asking about.

When a head tries to do both jobs, it does neither well. Short mobile questions with product names in the middle stress this — the head sees "Nova Buds" and "refund" in the same window and must pick one.

**Per-head measurement:** Pattern purity score. On a held-out set of refund questions, does the ask-type head activate consistently, or does its activation depend on which product name is present?

---

### S — Stitch the Spectra

After heads run in parallel, their outputs must combine correctly. Even if the ask-type head correctly identifies "refund" and the product-name head correctly identifies "Trail Jacket," the downstream combination must weight ask-type higher for answer selection.

The FAQ bot's failure suggests the stitch may be weighting product-name signal too heavily — so "Trail Jacket" pulls toward shipping/delivery answers even when ask-type clearly says "refund."

**Per-head measurement:** Contribution ratio at the combination layer. What fraction of the final answer-selection logit comes from ask-type head vs product-name head on refund questions?

---

### M — Map What Each Head Sees

Before trusting any head's output, visualize what it actually attends to. On "refund for wrong size on the Trail Jacket, not a shipping question," does the ask-type head attend to "refund," "wrong size," and "not a shipping question"? Or does it get distracted by "Trail Jacket"?

This is the ablation principle — the one rated highest (4) in this audit. Nobody has run the ask-type step alone on the specimen sentences to see what it actually sees before product-name matching starts.

**Per-head measurement:** Attention weight distribution on each token. For the ask-type head, attention should concentrate on intent tokens ("refund," "cancel," "return") not entity tokens ("Nova Buds," "Trail Jacket").

---

## The Anti-Pattern: Collapse to Monochrome

When all heads converge on the same dominant signal — in this case, product names — the attention mechanism loses its power. Instead of a spectrum of specialized detectors, you get one loud signal drowning out everything else.

The FAQ bot shows classic monochrome collapse symptoms:
- "Nova Buds delivery says Friday — can i still cancel" → shipping answer (product name dominates)
- "refund for wrong size on the Trail Jacket, not a shipping question" → shipping answer (product name dominates)

The shopper explicitly says "not a shipping question" and still gets a shipping answer. That's monochrome collapse — the product-name signal has taken over heads that should be detecting ask-type.

---

## How the Interrogator Uses These Principles

When a stranger brings their own attention setup, the interrogator walks all five principles conversationally:

1. **Partition** — Are your heads occupying distinct subspaces?
2. **Run** — Are they processing in parallel or creating dependencies?
3. **Individuate** — Does each head specialize in one pattern?
4. **Stitch** — Are outputs combining with correct weights?
5. **Map** — Have you visualized what each head actually attends to?

For each principle, the interrogator proposes a candidate finding and names the per-head measurement that would confirm it. The stranger's audit ends with a severity story on one of their pasted inputs, a ship/hold call, and a tripwire with a threshold.

The builder's own audit — the FAQ bot with its ablation finding, Hold call, and 10% disagreement tripwire — serves as the worked example. But the framework letters (P-R-I-S-M) never appear in the interrogator's output. The stranger sees the walk, the findings, the measurements — not the mnemonic.
