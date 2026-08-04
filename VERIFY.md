# Verification: Store FAQ Bot Head-Map Audit

This file confirms the interrogator works as designed — surfacing the ablation finding and demanding a per-head number for it.

---

## Stranger Verification Steps

### 1. Open /play and paste the seeded specimen

Use the Store FAQ bot that picks an answer for shopper questions as your test case. When the interrogator asks for your specimen, enter:

> Store FAQ bot that picks an answer for shopper questions

### 2. Provide the real inputs when prompted

When asked about usage reality, describe:

> Short mobile questions with product names in the middle

Paste these three sentences from store help-desk chat logs:

```
how long do i have to return the Nova Buds after they ship
Nova Buds delivery says Friday — can i still cancel
refund for wrong size on the Trail Jacket, not a shipping question
```

### 3. Confirm the tool surfaces the ablation finding

The interrogator must walk all five splits. When it reaches ablation, it should surface this core issue:

Nobody has run the ask-type step alone on "refund for wrong size on the Trail Jacket, not a shipping question" to see if it correctly tags this as refund before product-name matching even starts. The team only sees the bot's final reply — so if the final answer is wrong, there's no way to tell whether ask-type detection failed or

### 4. Confirm the tool demands a per-head number

The interrogator must not accept "the ask-type classifier seems off" as a finding. It must demand a specific measurement — for ablation, that means:

- **What to isolate:** The ask-type classifier, run standalone
- **What to measure:** Whether its raw output on refund-tagged messages disagrees with the bot's final live answer
- **The threshold that means trouble:** Any disagreement rate above 10%

If the tool accepts a vague finding without a per-head number, verification fails.

### 5. Confirm the result includes severity story, call, and tripwire

The final audit output must contain:

**Severity story on a pasted input:**
The Trail Jacket sentence — "refund for wrong size on the Trail Jacket, not a shipping question" — with the specific failure that nobody has isolated whether ask-type detection failed before product-name matching.

**Call:**
Hold. Run the ask-type classifier standalone on all three specimen sentences and log its raw output before any ship decision — owner: ML engineer. Reopen the ship call once that isolated result exists.

**Tripwire with threshold:**
Watch whether the standalone ask-type test, once run, disagrees with the bot's final live answer on refund-tagged messages. Any disagreement rate above 10% means the bug is downstream of ask-type, redirecting engineering effort — ML engineer owns this check.

---

## Pass Criteria

| Check | Requirement |
|-------|-------------|
| Ablation surfaced | Tool identifies that the ask-type classifier has never been run in isolation |
| Per-head number demanded | Tool requires a specific measurement (disagreement rate) with a threshold (10%) |
| Severity story present | Output walks through the Trail Jacket sentence failure |
| Call stated | Output includes Hold with ML engineer owner and specific action |
| Tripwire with threshold | Output names the 10% disagreement rate and who watches it |

If all five checks pass, the interrogator is verified.
