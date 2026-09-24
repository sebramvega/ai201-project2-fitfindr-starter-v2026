# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

FitFindr helps a user search for a thrifted clothing item using a plain-language request that can include a description, size, and maximum price. It searches the available listings, selects the strongest match, and suggests ways to style that item using pieces from the user's existing wardrobe. It then creates a short fit-card caption for the selected find. If no listing matches the request, the agent stops before the styling tools and tells the user what they can change in their search.

---

## Tool Inventory

### `search_listings`

- **What it does:** Searches the available thrift listings for items matching the user's description, with optional size and maximum-price filters. Size matching is case-insensitive and matches complete size tokens, so a size like `M` can match `M` or `S/M` without incorrectly matching unrelated sizes.
- **Inputs:** `description` (str), `size` (str or None), `max_price` (float or None)
- **Returns:** A list of matching listing dictionaries, ordered with the best keyword match first and limited to the configured search result limit. Each dictionary contains the listing's item details such as title, size, price, style tags, brand, and platform.
- **When it has nothing:** Returns an empty list `[]`.

### `suggest_outfit`

- **What it does:** Suggests one or two ways to style the new thrifted item using pieces from the user's existing wardrobe.
- **Inputs:** `new_item` (dict), `wardrobe` (dict)
- **Returns:** A non-empty string containing outfit suggestions that use the new item and, when available, specific pieces from the user's wardrobe.
- **When it has nothing:** If the wardrobe contains no items, returns general styling advice for the new item instead of failing or returning an empty string.

### `create_fit_card`

- **What it does:** Creates a short, post-style caption for the thrift find using the selected item and outfit suggestion.
- **Inputs:** `outfit` (str), `new_item` (dict)
- **Returns:** A two-to-four sentence caption that mentions the item, price, and platform and describes the outfit's vibe.
- **When it has nothing:** If `outfit` is empty or whitespace, returns a descriptive message instead of raising an error.

---

## Planning Loop

**Branch rule:** If `search_listings` returns an empty list, the agent puts a useful message in `session["error"]` telling the user to try increasing the budget, changing the size, or using a broader description, then returns the session without calling `suggest_outfit` or `create_fit_card`. Otherwise, the agent selects the first search result, stores it in the session, passes it to `suggest_outfit`, stores that result, and then passes the stored outfit and selected item to `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** I used regular expressions to pull a size after the word `size` and a maximum price after the word `under`. Those parts are removed from the original query, and the remaining text becomes the description passed to `search_listings`.

**What moves through the session:** The original query is stored in `session["query"]`. The parsed description, size, and maximum price go into `session["parsed"]`, followed by the search results in `session["search_results"]`. The first result is saved as `session["selected_item"]`, the styling result is saved as `session["outfit_suggestion"]`, and the final caption is saved as `session["fit_card"]`. If the search is empty, `session["error"]` is set and the later fields remain `None`.

---

## Sample Run

**One full query**

```text
$ python agent.py
=== A query the data can match ===
  found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop
  outfit:   **Outfit 1: Y2K Streetwear Edge**
*   **Thrifted item:** Y2K Baby Tee — Butterfly Print
*   **Existing pieces:** Baggy straight-leg jeans (dark wash), vintage black denim jacket, chunky white sneakers, and black crossbody bag.
*   **Vibe:** A classic contrast of a fitted, feminine graphic tee with ultra-baggy denim, tied together with a vintage denim layer and chunky sneakers for an authentic early-2000s street style look.

**Outfit 2: Casual Earth-Tone Contrast**
*   **Thrifted item:** Y2K Baby Tee — Butterfly Print
*   **Existing pieces:** Wide-leg khaki trousers, brown leather belt, and chunky white sneakers.
*   **Vibe:** An effortless blend of Y2K graphic playfulness and minimalist earth tones, letting the pink and purple butterfly print pop against the neutral khaki trousers.
  fit card: Scored this adorable Y2K Baby Tee — Butterfly Print on Depop for just $18.00 and I am completely obsessed! Styled it with baggy dark wash denim and a black vintage jacket for the ultimate contrast between fitted and oversized. It gives off the absolute best nostalgic early-2000s streetwear energy.

=== A query it can't ===
  stopped: I couldn't find a matching item. Try increasing your budget, changing the size, or using a broader description.
  fit_card is None — it should still be None here

The second one should stop before the fit card. If both paths look the same,
the branch isn't doing anything yet.
```

**The three tools, tested one at a time**

```text
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_012', 'title': 'Oversized Crewneck Sweatshirt — Vintage Navy', 'description': 'Perfectly faded navy crewneck. Genuinely vintage — not manufactured distressed. Ribbed cuffs and hem. No graphics, clean.', 'category': 'tops', 'style_tags': ['vintage', 'basics', 'oversized', 'classic'], 'size': 'XL (fits oversized)', 'condition': 'good', 'price': 20.0, 'colors': ['navy'], 'brand': None, 'platform': 'thredUp'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```text
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe; item={'title':'Vintage Graphic Tee','category':'tops','colors':['black'],'style_tags':['vintage','grunge']}; print(suggest_outfit(item, get_example_wardrobe()))"

**Outfit 1: 90s Streetwear Edge**
*   **Thrifted Item:** Vintage Graphic Tee
*   **Existing Pieces:** Baggy straight-leg jeans (dark wash), Black combat boots, Black crossbody bag
*   **Why it works:** Tucking the graphic tee into the dark wash baggy jeans creates an effortless, balanced silhouette. Pairing it with the black combat boots and crossbody bag anchors the grunge aesthetic for a cohesive, streetwear-ready look.

**Outfit 2: Casual Contrast**
*   **Thrifted Item:** Vintage Graphic Tee
*   **Existing Pieces:** Wide-leg khaki trousers, Chunky white sneakers, Black cropped zip hoodie (worn open or carried)
*   **Why it works:** The relaxed khaki trousers soften the dark, edgy vibe of the vintage tee, while the chunky white sneakers add a fresh, casual contrast that keeps the outfit grounded and modern.
```

```text
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Scored these vintage Levi's 501 jeans for just $38.00 on Depop and I'm honestly obsessed. Paired them with crisp white sneakers for that effortlessly cool, casual streetwear vibe that goes with literally everything. Such a timeless thrift find!
```

---

## How I Used AI

**Moment 1**

- **What I asked for:** I used AI while implementing `search_listings` to help translate the tool specification into filtering and keyword-matching logic, including the size-matching requirement.
- **What came back:** The implementation loaded the listings, filtered by maximum price and size, scored description keywords against listing information, sorted the matches, and returned the configured number of results.
- **What I changed:** I used complete size tokens rather than a simple substring check. This allows a request for `M` to match `M` or `S/M` without incorrectly treating unrelated sizes such as `XL` or `US 9` as matches. I also tested the no-match case separately to confirm the function returns `[]`.

**Moment 2**

- **What I asked for:** I used AI to help build and debug the planning loop that parses the user's query, stores tool results in session state, and stops when the search returns no matches.
- **What came back:** The loop used regular expressions to extract size and maximum price, stored the parsed values and tool results in the session, and branched on whether `search_listings` returned any results.
- **What I changed:** During testing, my first price-parsing test returned `None` because PowerShell expanded `$30` before Python received the query. I reran the test with the dollar sign escaped and confirmed the parser returned `30.0`. I also tested the impossible-query path separately and confirmed that `selected_item`, `outfit_suggestion`, and `fit_card` remain `None` when the search returns an empty list.
<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
