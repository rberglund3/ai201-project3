# TakeMeter — Project Planning

**Task:** Fine-tuning DistilBERT to classify the quality of discourse in the r/VALORANT subreddit.

---

## 1. Community Context

[r/VALORANT](https://www.reddit.com/r/VALORANT/) is the primary community hub for players of Riot Games' tactical 5v5 shooter *VALORANT*, where members share strategy discussion, request coaching, post highlights, and react to the competitive scene. The community is fast-moving and high-emotion, so the same thread often mixes substantive strategic conversation with raw frustration, which makes the discourse hard to navigate. This project classifies posts into **Tactical_Analysis**, **Salt_Vent**, and **Improvement_Seeking** because distinguishing constructive strategy from emotional venting from genuine help requests lets the community surface useful knowledge, route coaching requests to people who can answer them, and keep low-effort rage from drowning out high-effort contributions.

---

## 2. Label Taxonomy

The taxonomy is built around the **author's primary communicative intent**: are they *contributing* knowledge, *expressing* emotion, or *requesting* help? Every post falls into exactly one of these intents, which is what makes the labels mutually exclusive and collectively exhaustive (see [§4](#4-mutual-exclusivity--exhaustiveness)).

### Tactical_Analysis
> **Definition:** A post whose primary intent is to *contribute* strategic, mechanical, or meta knowledge by explaining, evaluating, or theorizing about how the game should be played.

**Examples:**
1. *"Why a default-to-Pearl B execute beats a fast A rush against double-sentinel comps — breakdown with utility timings."*
2. *"Agent tier list after the Chamber nerf: I think Cypher is now the strongest controller-adjacent sentinel on attacker-sided maps. Here's my reasoning."*

### Salt_Vent
> **Definition:** A post whose primary intent is to *express* frustration, anger, or emotional release about the game, teammates, enemies, or developer decisions, without seeking a solution or contributing analysis.

**Examples:**
1. *"Hardstuck Silver for the THIRD act because my teammates instalock duelists and go 2-15 every single game. I'm done."*
2. *"Riot really released another $25 skin bundle but can't fix the servers I've DC'd from twice tonight. Absolute joke."*

### Improvement_Seeking
> **Definition:** A post whose primary intent is to *request* help, feedback, or guidance in order to get better at the game or solve a personal performance problem.

**Examples:**
1. *"I peak with my crosshair too high every round and lose first duels — what drills or settings should I use to fix this?"*
2. *"Plat 2 Jett main here, VOD attached. Can someone tell me what's holding me back from Diamond?"*

---

## 3. Edge Case Handling

**Ambiguous post (between Tactical_Analysis and Salt_Vent):**

> *"Sage walls do NOTHING anymore since the nerf. You literally cannot hold a site solo as a sentinel now, the wall melts in one Raze nade and the whole class is unplayable garbage. Riot has no idea what they're doing balancing this game."*

This post is hard to classify because it contains a **real tactical claim** (the Sage wall's reduced durability changes how a sentinel can hold a site solo) wrapped in **clear emotional venting** ("unplayable garbage," "no idea what they're doing").

### Decision Rule

> **Apply the label that matches the post's *primary intent*, judged by what the author is trying to accomplish — not by the topic they mention.**
>
> **Step 1 — Emotion test (separates `Tactical_Analysis` from `Salt_Vent`).** Ask: *"If you removed all emotional language, is there still a substantive, defensible point that a reader could learn from or debate?"*
>
> - **Yes → keep it as a contribution** (provisionally `Tactical_Analysis`). The emotion is decoration around a genuine analytical claim (e.g., a post that explains *why* the wall change breaks solo site holds with reasoning or numbers).
> - **No → `Salt_Vent`.** The "tactical" reference is just a hook for the complaint, and stripping the emotion leaves nothing to discuss.
>
> For the Sage example above, removing the venting leaves only "the Sage wall is weaker after the nerf" with no supporting reasoning or actionable insight — the post's real purpose is to vent. **Label: `Salt_Vent`.**
>
> **Step 2 — Dependency test (separates `Improvement_Seeking` from `Tactical_Analysis`).** Many posts both contribute analysis *and* ask the reader a question, so the presence of a question is **not** by itself decisive. When a post does both, ask: *"Could the author already act on their own conclusion if nobody replied?"* — equivalently, *"Does this post still deliver standalone value with zero replies?"*
>
> - **No — the author lacks the answer and needs it to proceed → `Improvement_Seeking`.** The question is *load-bearing*: the post exists to get help. (e.g., "I don't know if it's my crosshair or my sens — which is it?"; "Is there an actual lineup for this, because I don't have one?")
> - **Yes — the author has already produced a working, self-sufficient answer and only invites validation or discussion → `Tactical_Analysis`.** The question is a *rhetorical sanity-check* appended to a finished contribution. (e.g., a complete strategy breakdown the author is "already climbing with," capped by "does this hold up?")
>
> **Order matters:** apply the emotion test first to rule out `Salt_Vent`, then the dependency test to settle `Tactical_Analysis` vs. `Improvement_Seeking`. A post can fail the emotion test (pure venting) even if it ends in a question — an angry "why is this agent garbage??" with no real ask is still `Salt_Vent`.
>
> **On reader-directed questions:** `Improvement_Seeking` is the label whose *primary purpose* is a load-bearing request for help — a question the author depends on to act. A reader-directed question alone does **not** force this label; an author who has already reached a usable conclusion and merely seeks validation is contributing knowledge (`Tactical_Analysis`), not seeking improvement. The dependency test, not the mere presence of a "?", decides which applies.

### Real-World Annotation Edge Cases (Milestone 3 Additions)

During the annotation of our 255 real-world examples, these three specific posts from our dataset proved genuinely difficult to resolve and required strict adherence to our decision tests:

1. **The UI Quality-of-Life Annoyance**
   * *Post:* *"Anyone else annoyed that the game mode defaults back to Unrated every time? ... If I was playing Competitive, Swiftplay, or Deathmatch, why not just keep that selected the next time I queue up?"*
   * *The Debate:* `Tactical_Analysis` (discussing game menu systems/patch behavior) vs. `Salt_Vent`.
   * *Resolution:* **`Salt_Vent`**. Applying Step 1 (The Emotion Test): if you strip away the emotional frustration language (*"annoyed," "little annoyances," "bothered"*), there is no substantive, defensible tactical gameplay strategy or mechanical meta-knowledge left for a reader to learn. The post's primary communicative intent is venting about a minor client inconvenience, not analyzing how the game is played.
2. **The Out-of-Game Technical Query**
   * *Post:* *"Just wondering, does vanguard ever flag background apps that aren’t related to the game? ... I ran into an account issue and it made me wonder, does vanguard ever detect other running apps..."*
   * *The Debate:* `Tactical_Analysis` (analyzing the game's anti-cheat infrastructure) vs. `Improvement_Seeking`.
   * *Resolution:* **`Improvement_Seeking`**. Applying Step 2 (The Dependency Test): the author has run into a breaking account issue and completely lacks the technical answer required to troubleshoot it. The question is fully load-bearing; the post provides zero standalone strategic value to the community on its own and exists purely to acquire external assistance.
3. **The Multi-Variable Diagnostic Request**
   * *Post:* *"How do I get out of bronze? ... After my longest break ever (thanks to college apps) I started playing again around May and the same thing happened... I'd like to do some better analysis."*
   * *The Debate:* `Tactical_Analysis` (given the explicit request for "better analysis") vs. `Improvement_Seeking`.
   * *Resolution:* **`Improvement_Seeking`**. Although the author uses analytical terminology, the underlying communicative intent is a macro-level request for personal performance guidance. The author cannot move forward without external tracking advice or gameplay tips, meaning they fail the dependency test for standalone data contribution.

---

## 4. Mutual Exclusivity & Exhaustiveness

The three labels partition the space of posts by **primary communicative intent**, the single dimension that defines the taxonomy:

| Label | Primary Intent | Direction | Litmus Test |
|---|---|---|---|
| **Tactical_Analysis** | *Contribute* knowledge | Author → community (gives) | "Here is what I think is true about the game." |
| **Salt_Vent** | *Express* emotion | Author → outward (releases) | "Here is how I feel, and I want to be heard." |
| **Improvement_Seeking** | *Request* help | Community → author (gets) | "Here is my problem, please help me solve it." |

- **Mutually exclusive:** A post has exactly one *primary* intent. Although a single post may combine analysis, emotion, and a question, the Decision Rule in [§3](#3-edge-case-handling) forces a single winner by asking which intent the post exists to serve. *Giving* knowledge, *expressing* feeling, and *getting* help are distinct goals that cannot all be primary at once.
- **Collectively exhaustive (for this community):** Within r/VALORANT discourse, every text post is fundamentally doing one of these three things — adding to the shared knowledge base, releasing emotion, or asking for assistance. The taxonomy therefore covers the full space of in-scope discourse posts.

> **Scope note:** This taxonomy targets *discourse* posts. Out-of-scope content for the model (e.g., pure highlight clips, fan art, LFG/team-finding, or moderator announcements) is handled by upstream filtering and is not part of the three-way classification task.

---

## 5. Data Collection Plan

**Source & method.** We will scrape text and link posts from the r/VALORANT [`/new`](https://www.reddit.com/r/VALORANT/new/) feed using the Reddit API (PRAW). The `/new` feed (rather than `/hot` or `/top`) is chosen deliberately: it gives an unbiased chronological sample that has not yet been filtered by community upvotes, so our distribution reflects what people actually post rather than what the community elevates. For each post we capture `id`, `title`, `selftext`, `link_flair_text`, `created_utc`, `score`, and `num_comments`, then concatenate `title + selftext` as the model input.

**Target dataset.** 200 labeled posts total, with a hard floor of **≥ 20% (≥ 40 examples) per label** so no class is starved during fine-tuning.

**Two-phase collection to guarantee balance.** Natural `/new` traffic is dominated by **Salt_Vent** and one-line **Improvement_Seeking** posts, while **Tactical_Analysis** is comparatively rare, so a naive scrape will under-represent it. We therefore collect in two phases:

1. **Phase 1 — Baseline sweep.** Pull the most recent ~500 posts from `/new`, deduplicate by `id`, drop out-of-scope content ([§4 scope note](#4-mutual-exclusivity--exhaustiveness)), and begin labeling. After the first pass we compute the per-label counts.
2. **Phase 2 — Targeted top-up.** For any label below its 40-example floor, run **focused retrieval** to backfill it, drawing only from r/VALORANT and still preferring recent posts:

   | Under-represented label | Reddit search keywords | Likely flairs |
   |---|---|---|
   | **Tactical_Analysis** | `guide`, `lineup`, `meta`, `tier list`, `breakdown`, `theory` | `Discussion`, `Guide`, `Strategy` |
   | **Improvement_Seeking** | `how do I`, `tips`, `VOD review`, `hardstuck`, `improve`, `settings` | `Question`, `Educational/Guide` |
   | **Salt_Vent** | `rant`, `done with this game`, `unplayable`, `Riot please`, `tilted` | `Discussion`, `Esports` (rage threads) |

   Keyword/flair hits are used **only to surface candidate posts**, never to assign labels — every candidate still passes through the manual labeling process in [§8](#8-ai-tool-plan) so that retrieval bias does not leak into the ground truth.

**Stopping rule.** Collection stops once we have 200 labeled posts **and** every label is at ≥ 40 examples. If targeted top-up over-fills a class, we cap it by random down-sampling rather than letting one label exceed ~45% of the set, keeping the distribution within a roughly 40/30/30 band.

---

## 6. Evaluation Metrics

We report **two headline metrics together** because each hides a different failure mode:

- **Overall Accuracy** answers "what fraction of all posts did we get right?" It is intuitive and matches the deployment experience of a user scrolling a feed, but on an imbalanced set it can be inflated by simply predicting the majority class. We keep it because our target distribution is only mildly imbalanced (~40/30/30) and accuracy is the most legible single number for a project stakeholder.
- **Macro F1-score** averages the per-class F1 with **equal weight per label**, ignoring class frequency. This is the metric that protects the minority **Tactical_Analysis** class: a model that quietly fails on tactical posts can still post high accuracy, but its Macro F1 will collapse. Macro F1 is therefore our primary model-selection metric, and Accuracy is the secondary sanity check.

**Why precision is critical for `Tactical_Analysis`.** The deployment goal is to *surface high-effort strategic posts*. A false positive — labeling a rant or a help request as Tactical_Analysis — pollutes the very feed we are trying to make valuable, eroding user trust in the "analysis" surface. We would rather miss a borderline tactical post (recall cost) than promote junk into the curated view, so we optimize **precision** for this label.

**Why recall is critical for `Salt_Vent`.** The deployment goal for Salt_Vent is *moderation triage and de-amplification* — catching toxic, low-value venting so it can be down-ranked or routed to mods. A false negative here (venting that slips through as analysis or a question) is the expensive error, because it lets the rage we are trying to filter reach the community. Misclassifying a bit of genuine analysis as salt is a milder cost, so for this label we prioritize **recall** (catch as much venting as possible).

This precision/recall split is exactly the trade-off Macro F1 balances per class, which is why it is our primary metric.

---

## 7. Definition of Success

> **Success threshold:** **Accuracy ≥ 75%** *and* **Macro F1 ≥ 0.70** on a held-out test split.

**Why this is realistic.** With a 200-post dataset and a 3-way intent task that includes a genuinely ambiguous boundary ([§3](#3-edge-case-handling)), even strong human annotators will not agree 100% of the time — the analysis-vs-salt edge alone caps the achievable ceiling. A fine-tuned DistilBERT on a few hundred in-domain examples typically lands in the 75–85% accuracy range for a clean 3-class scheme, so 75% / 0.70 sits at the low-but-meaningful end of what is achievable rather than at an aspirational ceiling we are unlikely to hit.

**Why this is "good enough" for deployment.** TakeMeter is a **discourse-quality utility for an online community, not a safety-critical classifier.** Its outputs feed soft actions — sorting, tagging, surfacing, and triage suggestions — where a human (a moderator or the reader's own judgment) remains in the loop and the cost of any single error is low. At ≥ 75% accuracy and ≥ 0.70 Macro F1, the tool is right far more often than a naive keyword filter and adds real signal to the feed, while the Macro F1 floor guarantees the minority Tactical_Analysis class is handled competently rather than sacrificed. Chasing a higher bar would demand a substantially larger labeled set for marginal deployment benefit, so this threshold is the honest "ship it" line for a v1 community utility.

---

## 8. AI Tool Plan

We use LLMs as **auxiliary tooling around a human-owned ground truth**, never as the source of the labels themselves. Each use below has a defined boundary so model assistance cannot contaminate the dataset.

**Label Stress-Testing.**
Before bulk labeling, we will prompt an LLM to generate **5–10 synthetic "boundary" posts** that deliberately sit on our decision lines — e.g., a vent that contains a buried tactical claim ([§3](#3-edge-case-handling)), a help request phrased as analysis, or an analysis post written in an angry tone. We hand-label these ourselves first, then check whether our [§3 Decision Rule](#3-edge-case-handling) produces a clear, consistent verdict for each. Cases that split our own judgment trigger a refinement of the label definitions *before* real annotation begins, so the taxonomy is hardened against ambiguity up front. These synthetic posts are used only for definition-testing and are **excluded from the 200-post training/eval set.**

**Annotation Assistance.**
We will perform **100% manual labeling of all 200 posts** to preserve ground-truth purity — no LLM-assigned labels enter the dataset. As a *quality check only*, after the first **20 human-labeled examples** we will have an LLM independently classify those same 20 posts (given our §2 definitions and §3 rule) and **cross-audit** its labels against ours. Disagreements are reviewed by hand: each one either exposes an annotator inconsistency we correct, or a gap in the written definitions we clarify. The LLM's labels are never copied into the dataset — they are a mirror held up to our own labeling discipline.

**Failure Analysis.**
After fine-tuning, we will export every **misclassified test instance** (true label, predicted label, full text, length, and any agent/flair metadata) and feed the error set to an LLM to hunt for **systematic patterns** rather than one-off mistakes. Specifically we ask it to test whether errors correlate with:

- **Post length** — are very short posts (one-liners) or very long posts disproportionately misread?
- **Sarcasm / tone inversion** — is positive-sounding sarcasm being mislabeled as Tactical_Analysis instead of Salt_Vent?
- **Agent / map / patch keywords** — do dense game-specific terms (e.g., agent names, ability lineups) bias the model toward Tactical_Analysis regardless of true intent?

The output is a ranked list of hypothesized failure drivers, which directs concrete fixes — targeted data collection for an under-served pattern, definition clarifications, or input preprocessing — feeding the next iteration of the model.
