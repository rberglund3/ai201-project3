# TakeMeter — DistilBERT Discourse Classifier for r/VALORANT

Fine-tuning DistilBERT to classify the quality of discourse in the r/VALORANT subreddit into three labels: `Tactical_Analysis`, `Salt_Vent`, and `Improvement_Seeking`. See [planning.md](planning.md) for the label taxonomy, decision rules, and data-collection plan.

---

## Evaluation & Error Analysis

### Baseline vs. Fine-Tuned Performance

**Zero-Shot Baseline (Groq LLaMA-3):**
* Accuracy: 71.0%
* Macro F1: 0.58
* Per-Class F1: Tactical_Analysis (0.59), Salt_Vent (0.33), Improvement_Seeking (0.82)

**Fine-Tuned Model (DistilBERT):**
* Accuracy: 51.6%
* Macro F1: 0.27
* Per-Class F1: Tactical_Analysis (0.14), Salt_Vent (0.00), Improvement_Seeking (0.67)

### Confusion Matrix (Test Set)

| True Label \ Predicted | Tactical_Analysis | Salt_Vent | Improvement_Seeking |
| :--- | :--- | :--- | :--- |
| **Tactical_Analysis** | 1 | 0 | 11 |
| **Salt_Vent** | 0 | 0 | 3 |
| **Improvement_Seeking** | 1 | 0 | 15 |

### The Majority Class Collapse (Intended vs. Learned Behavior)

Our fine-tuned DistilBERT model significantly underperformed the zero-shot baseline. While I intended for the model to learn the distinct linguistic boundaries between analyzing, venting, and seeking help, the model actually learned a "cheat code" to pass the test: guess the majority class.

During data ingestion, 50 rows were dropped due to label string mismatches in the CSV, disproportionately eliminating `Salt_Vent` and `Tactical_Analysis` examples. The resulting training split was heavily dominated by `Improvement_Seeking` data. The confusion matrix proves this collapse: the model predicted `Improvement_Seeking` 29 out of 31 times. It learned that guessing the majority class was the safest way to minimize loss, completely failing to capture the boundaries of the minority classes.

### Qualitative Error Analysis

Reviewing the misclassified test examples highlights exactly where the boundaries blurred:

1. **"LOUD allows Virtyy to explore options"** (True: `Tactical_Analysis` | Pred: `Improvement_Seeking`).
   * *Analysis:* The model failed to recognize esports roster moves. `Tactical_Analysis` became a "junk drawer" for anything not explicitly asking for help, meaning the model couldn't find a consistent mathematical pattern for what analysis actually sounds like. To fix this, the label definition needs to be tighter (excluding esports news).
2. **"Why are my Elo Gains so bad?"** (True: `Salt_Vent` | Pred: `Improvement_Seeking`).
   * *Analysis:* The model conflated a rhetorical, frustrated question with a genuine request for help purely due to the interrogative structure ("Why..."). It overfit to the presence of question marks.
3. **"Valorant's Rank distribution, Top%, High Elo"** (True: `Tactical_Analysis` | Pred: `Improvement_Seeking`).
   * *Analysis:* Statistical breakdowns lack the distinct vocabulary of help-seeking, but the model defaulted to the majority class prediction anyway, evident by the incredibly low confidence score (0.39).

### Sample Classifications

| Post Text | Predicted Label | Confidence |
| :--- | :--- | :--- |
| *"How do I stop crouch spraying when I get surprised?"* | Improvement_Seeking | 0.85 |
| *"LOUD allows Virtyy to explore options"* | Improvement_Seeking | 0.38 |
| *"Why are my Elo Gains so bad?"* | Improvement_Seeking | 0.38 |

*Note on correct prediction:* The first example was correctly predicted as `Improvement_Seeking` because it contains explicit directive requests ("How do I stop...") which the model successfully mapped to this category.

---

## Project Reflections

### Spec Reflection

The `planning.md` spec was incredibly helpful for defining the strict boundaries of my labels before I started annotating, which prevented mid-task drift. However, my implementation diverged from the spec during the baseline evaluation. The zero-shot model failed to parse the outputs 31/31 times due to conversational filler. I had to diverge from the planned prompt and implement a strict, lowercase Python matching script to forcefully parse the outputs and generate the baseline score.

### AI Usage

1. **Data Formatting & Syntax Fixing:** I used Claude/Gemini to write the exact Python parsing logic required to fix the zero-shot baseline evaluation when the LLM refused to output clean strings. The AI produced the `.strip().lower()` matching loop, which I integrated directly into the notebook.
2. **Error Analysis Brainstorming:** After generating the confusion matrix, I fed the raw diagnostic logs to an AI assistant to help surface patterns. The AI helped identify the "Majority Class Collapse" by pointing out the 50 dropped rows in the ingestion phase, which formed the foundation of my written analysis above.
