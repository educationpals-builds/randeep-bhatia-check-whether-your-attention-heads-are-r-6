# Builder's Audit Charter: Store FAQ Bot Head-Map Inspection

## Specimen Under Audit

**Tool:** Store FAQ bot that picks an answer for shopper questions

**Situation:** Shoppers ask about refunds, but the FAQ bot answers with shipping times because it latched onto the product name. Fix that before the busy sale week.

**Stakes:** Shoppers get the wrong policy and leave the cart

### Attention Architecture Context

The FAQ bot's attention mechanism divides its capacity across heads. Each head operates on a slice of the embedding space: if the model uses d_model total dimensions and h attention heads, each head works with d_model ÷ h dimensions. When a shopper types "refund for wrong size on the Trail Jacket," different heads should specialize—some attending to the ask-type (refund), others to the product entity (Trail Jacket). The failure occurs when heads that should attend to ask-type instead latch onto the product name, routing the query to shipping information.

---

## Standard Line

If someone asks about refunds, the answer is about refunds — not shipping

---

## Specimen Sentences

**Source:** Store help-desk chat logs

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

---

## Five Split Findings

| Check | Rating | Finding |
|-------|--------|---------|
| Room | 0 | No evidence that the embedding space is too cramped for the ask-type vs product-name distinction |
| Copies | 1 | Minor concern—some heads may redundantly attend to product names, but not the primary issue |
| Stitch | 0 | Cross-head integration appears functional; the problem is upstream of stitching |
| Unowned | 0 | No orphaned relationship patterns detected in the attention maps |
| Ablation | 4 | **Critical.** The ask-type classification step has never been tested in isolation. When the bot returns the wrong answer, there is no visibility into whether ask-type detection failed or whether the failure occurred downstream. |

**Decider:** ablation

---

## Severity Story

Nobody has run the ask-type step alone on "refund for wrong size on the Trail Jacket, not a shipping question" to see if it correctly tags this as refund before product-name matching even starts. The team only sees the bot's final reply — so if the final answer is wrong, there's no way to tell whether ask-type detection failed or

---

## Ship Call

**Verdict:** Hold

Hold. Run the ask-type classifier standalone on all three specimen sentences and log its raw output before any ship decision — owner: ML engineer. Reopen the ship call once that isolated result exists.

---

## Tripwire

Watch whether the standalone ask-type test, once run, disagrees with the bot's final live answer on refund-tagged messages. Any disagreement rate above 10% means the bug is downstream of ask-type, redirecting engineering effort — ML engineer owns this check.

| Metric | Threshold | Owner |
|--------|-----------|-------|
| Disagreement rate between standalone ask-type output and bot's final answer on refund-tagged messages | >10% | ML engineer |

---

## Audit Summary

This charter documents the builder's complete inspection of the Store FAQ bot's attention heads. The ablation check scored highest (4) because the ask-type classifier has never been run in isolation—making it impossible to localize the refund-vs-shipping confusion. The ship call is Hold until the ML engineer runs standalone ask-type classification on all three specimen sentences and logs raw output. Post-release monitoring watches for disagreement between isolated ask-type results and final bot answers, with 10% as the trouble threshold.
