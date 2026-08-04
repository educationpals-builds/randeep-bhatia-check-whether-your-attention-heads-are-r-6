# Split-Walk Prompt Pack: FAQ Bot Attention Audits

Five standalone prompts for interrogating attention splits in any chat model. Each prompt walks one split and ends with the per-head measurement it demands.

**Worked Example Context (from the builder's audit):**

- **Specimen:** Store FAQ bot that picks an answer for shopper questions
- **Stakes:** Shoppers get the wrong policy and leave the cart
- **Standard:** If someone asks about refunds, the answer is about refunds — not shipping
- **Usage reality:** Short mobile questions with product names in the middle
- **Specimen sentences (from Store help-desk chat logs):**
  1. how long do i have to return the Nova Buds after they ship
  2. Nova Buds delivery says Friday — can i still cancel
  3. refund for wrong size on the Trail Jacket, not a shipping question
- **Top crack:** ablation (rated 4)
- **Severity story:** Nobody has run the ask-type step alone on "refund for wrong size on the Trail Jacket, not a shipping question" to see if it correctly tags this as refund before product-name matching even starts. The team only sees the bot's final reply — so if the final answer is wrong, there's no way to tell whether ask-type detection failed or
- **Call:** Hold. Run the ask-type classifier standalone on all three specimen sentences and log its raw output before any ship decision — owner: ML engineer. Reopen the ship call once that isolated result exists.
- **Tripwire:** Watch whether the standalone ask-type test, once run, disagrees with the bot's final live answer on refund-tagged messages. Any disagreement rate above 10% means the bug is downstream of ask-type, redirecting engineering effort — ML engineer owns this check.

---

## Prompt 1: Room Split

Use this prompt to check whether the attention setup has enough capacity to distinguish ask-types from product names.

```
You are auditing an attention setup for a FAQ bot that picks answers for shopper questions.

The problem: Shoppers ask about refunds, but the bot answers with shipping times because it latched onto the product name.

Here are three real failing messages from store help-desk chat logs:
- how long do i have to return the Nova Buds after they ship
- Nova Buds delivery says Friday — can i still cancel
- refund for wrong size on the Trail Jacket, not a shipping question

Walk the ROOM split: Does this attention setup have enough representational capacity to hold both the ask-type signal (refund vs. shipping vs. cancellation) AND the product-name signal (Nova Buds, Trail Jacket) without one overwriting the other?

Consider:
- How many dimensions are available for encoding ask-type vs. product-name?
- When both signals compete for the same attention heads, which wins?
- Is there structural room for the bot to attend to "refund" separately from "Nova Buds"?

End with the per-head measurement:
**Measurement demanded:** Count how many attention heads activate primarily on ask-type tokens (refund, return, cancel) vs. product-name tokens (Nova Buds, Trail Jacket) when processing the specimen sentences. Report the ratio. If fewer than 2 heads attend primarily to ask-type, room is insufficient.
```

---

## Prompt 2: Copies Split

Use this prompt to check whether the attention setup duplicates work across heads instead of specializing.

```
You are auditing an attention setup for a FAQ bot that picks answers for shopper questions.

The problem: Shoppers ask about refunds, but the bot answers with shipping times because it latched onto the product name.

Here are three real failing messages from store help-desk chat logs:
- how long do i have to return the Nova Buds after they ship
- Nova Buds delivery says Friday — can i still cancel
- refund for wrong size on the Trail Jacket, not a shipping question

Walk the COPIES split: Are multiple attention heads doing the same job — all latching onto product names — instead of different heads specializing in different signals?

Consider:
- Do several heads all attend heavily to "Nova Buds" or "Trail Jacket"?
- Is there redundant product-name attention while ask-type attention is missing?
- Would removing one product-name head change the output, or do copies cover for each other?

End with the per-head measurement:
**Measurement demanded:** For each attention head, compute the cosine similarity of its attention pattern across the three specimen sentences. Heads with similarity > 0.85 to another head are copies. Report how many copy-pairs exist and whether any head uniquely attends to ask-type tokens.
```

---

## Prompt 3: Stitch Split

Use this prompt to check whether attention heads pass information to each other correctly across layers.

```
You are auditing an attention setup for a FAQ bot that picks answers for shopper questions.

The problem: Shoppers ask about refunds, but the bot answers with shipping times because it latched onto the product name.

Here are three real failing messages from store help-desk chat logs:
- how long do i have to return the Nova Buds after they ship
- Nova Buds delivery says Friday — can i still cancel
- refund for wrong size on the Trail Jacket, not a shipping question

Walk the STITCH split: Even if early heads correctly detect "refund" as the ask-type, does that signal get passed forward to the heads that select the final answer — or does it get dropped while product-name signal gets stitched through?

Consider:
- Trace the ask-type signal from detection to answer selection
- Where in the layer stack does the refund signal lose strength?
- Does product-name signal have a cleaner path to the output layer?

End with the per-head measurement:
**Measurement demanded:** Measure the attention weight that later-layer heads place on the residual stream positions where early-layer heads encoded ask-type. Compare to the weight placed on product-name positions. Report the ratio at each layer transition. A ratio below 0.5 at any transition indicates a stitch failure.
```

---

## Prompt 4: Unowned Split

Use this prompt to check whether any critical subtask has no attention head responsible for it.

```
You are auditing an attention setup for a FAQ bot that picks answers for shopper questions.

The problem: Shoppers ask about refunds, but the bot answers with shipping times because it latched onto the product name.

Here are three real failing messages from store help-desk chat logs:
- how long do i have to return the Nova Buds after they ship
- Nova Buds delivery says Friday — can i still cancel
- refund for wrong size on the Trail Jacket, not a shipping question

Walk the UNOWNED split: Is there a subtask in the pipeline — like "prioritize ask-type over product-name when they conflict" — that no attention head is responsible for?

Consider:
- List every subtask: detect ask-type, detect product-name, resolve conflicts, select answer
- For each subtask, identify which head owns it
- Which subtask has no clear owner or has ownership split across heads that don't coordinate?

End with the per-head measurement:
**Measurement demanded:** For each subtask, measure the maximum activation of any single head when that subtask is required. A subtask is unowned if no head exceeds 0.3 activation when the subtask is needed. Report which subtasks are unowned.
```

---

## Prompt 5: Ablation Split

Use this prompt to check whether removing or isolating a component reveals where the failure actually lives.

```
You are auditing an attention setup for a FAQ bot that picks answers for shopper questions.

The problem: Shoppers ask about refunds, but the bot answers with shipping times because it latched onto the product name.

Here are three real failing messages from store help-desk chat logs:
- how long do i have to return the Nova Buds after they ship
- Nova Buds delivery says Friday — can i still cancel
- refund for wrong size on the Trail Jacket, not a shipping question

Walk the ABLATION split: If you run the ask-type classifier step alone — isolated from product-name matching — does it correctly tag these messages? The team only sees the bot's final reply, so there's no way to tell whether ask-type detection failed or whether a later step overrode a correct detection.

This is the top crack identified in the audit. The severity story:
Nobody has run the ask-type step alone on "refund for wrong size on the Trail Jacket, not a shipping question" to see if it correctly tags this as refund before product-name matching even starts. The team only sees the bot's final reply — so if the final answer is wrong, there's no way to tell whether ask-type detection failed or

Consider:
- What does the ask-type classifier output when run standalone on each specimen sentence?
- If it outputs "refund" correctly, the bug is downstream
- If it outputs "shipping" or "product inquiry," the bug is in ask-type detection itself

End with the per-head measurement:
**Measurement demanded:** Run the ask-type classifier standalone on all three specimen sentences. Log its raw output (the predicted ask-type) for each. Compare to the bot's final live answer. Report the disagreement rate. Any disagreement rate above 10% means the bug is downstream of ask-type, redirecting engineering effort.
```

---

## Using This Pack

1. **Pick the split** most relevant to your current hypothesis
2. **Paste the prompt** into any chat model
3. **Replace the specimen details** with your own attention setup, failing messages, and context
4. **Run the measurement** the prompt demands — don't skip it
5. **Record the number** before moving to the next split

The builder's audit rated the splits: room (0), copies (1), stitch (0), unowned (0), ablation (4). Ablation was the decider — the team needed to isolate the ask-type step before any ship decision.

**The call from this audit:** Hold. Run the ask-type classifier standalone on all three specimen sentences and log its raw output before any ship decision — owner: ML engineer. Reopen the ship call once that isolated result exists.

**The tripwire:** Watch whether the standalone ask-type test, once run, disagrees with the bot's final live answer on refund-tagged messages. Any disagreement rate above 10% means the bug is downstream of ask-type, redirecting engineering effort — ML engineer owns this check.
