---
layout: default
title: "Why LLMs for Survey Coding?"
nav_order: 4
---

# Why LLMs for Survey Coding?
{: .no_toc }

## Understanding the Approach
{: .no_toc }

<div class="code-example" markdown="1">
**Estimated time:** 10 minutes
</div>

---


## The Traditional Survey Coding Problem

You've collected open ended survey responses. Now comes the hard part: **coding them**.

**Example from our data:**
> "they need to be just like every one else. If a white woman can do it so can a black person."

**Human coders must:**
- Identify mentions of racial groups (✓ blacks, ✓ whites)
- Code for individualism themes (✓ affirmation of individualism)
- Detect discrimination mentions (✓ against special treatment)
- Apply 9 different coding variables consistently

**Challenges:**
- **Time-intensive:** ~2-5 minutes per response plus training
- **Expensive:** 1,944 responses × 2 coders × $30/hour = $1,000+
- **Inter-rater reliability:** Human coders disagree 10-20% of the time
- **Not reproducible:** Results vary by coder, mood, fatigue
- **No Fun** (This one may be subjective): as opposed to more interpretive coding, mechanistic content coding, where you are trying to act isn't super interesting


## Enter Large Language Models

Modern LLMs can code survey responses:
- **Fast:** ~2 seconds per batch of 20
- **Cheap:** ~$2.50 for all 1,944 responses
- **Reproducible-ish:** Other researchers can run the same code and obtain comparable (though not reproducible in most cases!) results 
- **Comparable accuracy:** ~90-95% agreement with human coders

## How Did We Get Here? The Origin Story

### The Old Days (aka before 2019...)

Until fairly recently, we used machine learning techniques referred to as "supervised learning" to classify text. These strategies, e.g. "Support Vector Machines" or "Naive Bayes" treated texts as a "bag of words" (i.e., position/syntax didn't matter). Classifiers were then trained on human-coded ("ground truth") data and worked by classifying answers with similar words as alike -- you can even look at a list of words with the highest scores. 

### From Images to Text

Large Language Models emerged from developments in **computer vision**, not linguistics.

**2012: AlexNet & ImageNet**
- Deep learning ("neural networks") proved effective at image classification
- Given an image → predict label ("cat", "dog", "airplane")
- Trained on millions of labeled examples

**The Key Insight:** 
These models learned **hierarchical representations**:
1. Low layers: edges, corners, colors
2. Middle layers: textures, simple shapes
3. High layers: object parts, compositions
4. Final layer: classification

**2017-2018: Attention & Transformers**
- "Attention is All You Need" (Vaswani et al., 2017)
- Same principle could work for **sequences** (like text)
- Instead of pixels → words (tokens)
- Instead of spatial relationships → sequential relationships

### The Architecture Evolution

**BERT Era (2018-2019): The "Discriminator" Approach**

BERT (Bidirectional Encoder Representations from Transformers) and similar models were built like image classifiers:

```
Input Text → Encoder → [CLS] token → Classification Layer → Category
```

**How BERT was used for coding:**
1. **Pre-training:** Learn general language on massive corpus
2. **Fine-tuning:** Train on your specific coding task with labeled examples
3. **Classification:** Input → Label

**Limitations:**
- Needs 100s-1000s of labeled examples per task
- Separate fine-tuning for each codebook
- Not flexible: codebook changes require retraining

**Modern LLMs (GPT-3+, Claude): The "Generator" Approach**

These models work completely differently:

```
Input (Text + Instructions) → Generate Next Tokens → Structured Output
```

**Key Difference:** They're trained to **predict the next token** in text:
- Not trained for classification directly
- Instead, trained on trillions of tokens to understand language patterns
- "Classification" emerges from instruction-following ability

**Why This Matters for Survey Coding:**

**Zero-shot Learning:** No training examples needed!
```r
# Just tell it what to do
"Here's a codebook. Code these responses according to it."
# It works immediately
```

**Flexible:** Change codebook anytime
```r
# No retraining needed
# Just update the prompt
```

**Consistent:** With temperature=0, deterministic (in reality -- not quite)
```r
# Same input → same output
# Unlike humans who vary day-to-day
```

## Can LLMs match human inter-rater reliability?**

Our data includes human double-coding (Coder 1 & Coder 2), so we can directly compare:

- **Human-Human agreement:** How often do C1 and C2 agree?
- **LLM-Human agreement:** How often does Claude match C1? Match C2?


## Let's Get Started

Now that you understand the approach, let's make our [**first API call**](first-call.html) and see Claude code a survey response!

