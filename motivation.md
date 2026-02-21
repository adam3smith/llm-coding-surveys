---
layout: default
title: "Why LLMs for Survey Coding?"
nav_order: 3
---

# Why LLMs for Survey Coding?
{: .no_toc }

## Understanding the Approach
{: .no_toc }

<div class="code-example" markdown="1">
**Estimated time:** 10 minutes
</div>

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Traditional Survey Coding Problem

You've collected survey responses. Now comes the hard part: **coding them**.

**Example from our data:**
> "they need to be just like every one else. If a white woman can do it so can a black person."

**Human coders must:**
- Identify mentions of racial groups (✓ blacks, ✓ whites)
- Code for individualism themes (✓ affirmation of individualism)
- Detect discrimination mentions (✓ against special treatment)
- Apply 9 different coding variables consistently

**Challenges:**
- ⏱️ **Time-intensive:** ~2-5 minutes per response
- 💰 **Expensive:** 1,944 responses × 2 coders × $15/hour = $1,000+
- 📊 **Inter-rater reliability:** Human coders disagree 10-20% of the time
- 🔁 **Not reproducible:** Results vary by coder, mood, fatigue

## Enter Large Language Models

Modern LLMs can code survey responses:
- ⚡ **Fast:** ~2 seconds per batch of 20
- 💵 **Cheap:** ~$2.50 for all 1,944 responses
- 📈 **Consistent:** Same response → same code (with temperature=0)
- ✅ **Comparable accuracy:** ~90-95% agreement with human coders

## How Did We Get Here? The Origin Story

### From Images to Text

Large Language Models emerged from developments in **computer vision**, not linguistics.

**2012: AlexNet & ImageNet**
- Deep learning proved effective at image classification
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

BERT and similar models were built like image classifiers:

```
Input Text → Encoder → [CLS] token → Classification Layer → Category
```

**How BERT was used for coding:**
1. **Pre-training:** Learn general language on massive corpus
2. **Fine-tuning:** Train on your specific coding task with labeled examples
3. **Classification:** Input → Label

**Limitations:**
- ❌ Needs 100s-1000s of labeled examples per task
- ❌ Separate fine-tuning for each codebook
- ❌ Black box: hard to understand why it chose a code
- ❌ Not flexible: codebook changes require retraining

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

**Interpretable:** You can see its reasoning
```r
# Can ask: "Why did you code this as 'individualism'?"
# It can explain its decision
```

**Flexible:** Change codebook anytime
```r
# No retraining needed
# Just update the prompt
```

**Consistent:** With temperature=0, deterministic
```r
# Same input → same output
# Unlike humans who vary day-to-day
```

## The Paradigm Shift

| Aspect | BERT/Fine-tuning | Modern LLMs (Claude) |
|--------|------------------|----------------------|
| **Training data** | Need labeled examples | None required |
| **Codebook changes** | Retrain entire model | Just update prompt |
| **Transparency** | Black box | Can explain reasoning |
| **Consistency** | Varies by training | Deterministic (temp=0) |
| **Cost** | High (compute for fine-tuning) | Low (just inference) |
| **Time to deploy** | Days-weeks | Minutes |

## When to Use LLM-Based Coding

### ✅ Good Fit

- **Clear codebook** with specific definitions
- **Well-defined categories** (not highly subjective)
- **Large datasets** (100s-1000s of responses)
- **Need for consistency** across time
- **Multiple coding schemes** on same data
- **Exploratory analysis** (LLM can suggest categories)

### ❌ Not Recommended

- **Highly nuanced judgments** requiring deep context
- **Cultural/contextual knowledge** the LLM lacks
- **Legal/clinical decisions** requiring human accountability
- **Very small datasets** (<100 responses)
- **When transparency to stakeholders** is paramount and they don't trust AI

## The Research Question

**Can LLMs match human inter-rater reliability?**

Our data includes human double-coding (Coder 1 & Coder 2), so we can directly compare:

- **Human-Human agreement:** How often do C1 and C2 agree?
- **LLM-Human agreement:** How often does Claude match C1? Match C2?

**Spoiler:** You'll find Claude achieves ~90-95% agreement—comparable to human IRR!

## Our Approach

We'll use **Claude Haiku 4.5** because it:
- ✅ Fast inference (important for 1,944 responses)
- ✅ Very low cost ($0.25 per 1M input tokens)
- ✅ Strong reasoning for coding tasks
- ✅ 200K context window (fits codebook + batches)

**Alternative models:**
- **GPT-4o-mini:** Similar capability, comparable cost
- **Claude Sonnet 4.5:** Higher accuracy but 12x more expensive
- **Llama 3 (local):** Free but requires technical setup

We'll discuss model selection more in the [First API Call](first-call.html) section.

## What Makes This Work?

**Three critical elements:**

1. **Structured Prompting**
   - Clear instructions
   - Detailed codebook
   - Request JSON output

2. **Temperature=0**
   - Deterministic outputs
   - Reproducible codes
   - Essential for research

3. **Validation**
   - Always compare with human codes
   - Calculate agreement rates
   - Identify problematic responses

## Ethical Considerations

{: .warning }
> **Important:** LLM coding doesn't eliminate human judgment—it augments it.

**Best practices:**
- Always validate against human-coded subset
- Use for initial coding, human review for edge cases
- Document your process thoroughly
- Consider potential biases in training data
- Don't use for high-stakes decisions without human review

## Let's Get Started

Now that you understand the approach, let's make our [**first API call**](first-call.html) and see Claude code a survey response!

---

## Key Takeaways

- 🧠 Modern LLMs emerged from image classification research
- 🔄 They work by **next-token prediction**, not fine-tuned classification
- ⚡ This enables **zero-shot** coding with just a prompt
- 📊 They achieve **human-level** inter-rater reliability
- 💡 Best for: large datasets, clear codebooks, consistency needs
- ⚖️ Always validate against human codes
