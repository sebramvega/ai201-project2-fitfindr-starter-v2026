# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
The search uses keyword matching against listing data, so different phrasing may occasionally fail to match an otherwise relevant item. A 4-of-5 target allows for that limitation while still requiring the full agent flow to work reliably.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
This branch depends on the deterministic empty-list result from `search_listings`, rather than variable model output. Once no listings are returned, the loop should consistently stop instead of continuing with missing data.

---

## 3. The selected item persists through session state

Given a query that finds at least one listing, the item stored in
`session["selected_item"]` has the same listing `id` as the item received by
`suggest_outfit` — in 5 of 5 tries.

**Why this target:**
The agent should pass the selected item between tools through session state without changing or losing it. Because listing IDs are stable and directly comparable, this should succeed in every run.

---

## 4. The fit card includes the find's key details

Given a successful full run, the fit card mentions the selected item's price
and platform — in at least 4 of 5 tries.

**Why this target:**
The fit card is model-generated, so its wording can vary between runs. Requiring the price and platform in 4 of 5 tries checks that the caption consistently includes useful details without assuming identical model output every time.

---

## 5. Search respects the maximum price

Given a search with a `max_price`, every listing returned by
`search_listings` has a price less than or equal to that maximum — in 5 of 5
tries.

**Why this target:**
The price ceiling is a deterministic filter applied directly to the listing data, so no returned result should exceed the user's stated maximum price.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
