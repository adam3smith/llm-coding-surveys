---
layout: default
title: "Validation & Comparison"
nav_order: 8
---

# Validation & Comparison with Human Coders
{: .no_toc }

## Assessing Quality and Reliability
{: .no_toc }

<div class="code-example" markdown="1">
**Estimated time:** 15 minutes
</div>

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Validation Matters

{: .warning }
> **Critical:** Never deploy LLM coding without validation against human codes.

**Questions we need to answer:**
- How often does Claude agree with human coders?
- Is Claude's performance comparable to human inter-rater reliability?
- Which variables or responses are problematic?
- When can we trust Claude's codes?

## Our Data: Double-Coded by Humans

Our survey data includes human coding by two coders:
- **Coder 1:** Variables prefixed `C1Q1*`
- **Coder 2:** Variables prefixed `C2Q1*`
- **Claude:** Variables prefixed `AI_Q1*`

This lets us calculate:
- **Human IRR:** How often C1 and C2 agree
- **AI-Human agreement:** How often Claude matches each coder

## Calculating Agreement Rates

### Simple Percent Agreement

```r
library(tidyverse)

# Load data with codes
survey_data <- read_csv("survey_data_with_Q43_codes.csv")

# Calculate agreement for one variable
calculate_agreement <- function(col1, col2, data) {
  valid <- !is.na(data[[col1]]) & !is.na(data[[col2]])
  mean(data[[col1]][valid] == data[[col2]][valid])
}

# Human agreement on 'individ'
human_agreement <- calculate_agreement("C1Q1individ", "C2Q1individ", survey_data)
cat(sprintf("Human coders (C1-C2): %.1f%%\n", human_agreement * 100))

# AI-Human agreement
ai_c1 <- calculate_agreement("AI_Q1individ", "C1Q1individ", survey_data)
ai_c2 <- calculate_agreement("AI_Q1individ", "C2Q1individ", survey_data)

cat(sprintf("Claude-C1: %.1f%%\n", ai_c1 * 100))
cat(sprintf("Claude-C2: %.1f%%\n", ai_c2 * 100))
```

**Expected output:**
```
Human coders (C1-C2): 89.3%
Claude-C1: 87.8%
Claude-C2: 88.5%
```

## Agreement Across All Variables

```r
compare_all_variables <- function(survey_data, prefix_c1 = "C1Q1", 
                                  prefix_c2 = "C2Q1", prefix_ai = "AI_Q1") {
  
  variables <- c("code", "bexpment", "wexpment", "tblk", "individ", 
                "discrim", "welf", "obama", "rxn")
  
  results <- map_dfr(variables, function(var) {
    c1_col <- paste0(prefix_c1, var)
    c2_col <- paste0(prefix_c2, var)
    ai_col <- paste0(prefix_ai, var)
    
    # Skip if columns don't exist
    if (!all(c(c1_col, c2_col, ai_col) %in% names(survey_data))) {
      return(NULL)
    }
    
    # Only compare valid cases
    valid <- !is.na(survey_data[[c1_col]]) & 
             !is.na(survey_data[[c2_col]]) &
             !is.na(survey_data[[ai_col]])
    
    if (sum(valid) == 0) return(NULL)
    
    tibble(
      variable = var,
      n_cases = sum(valid),
      C1_C2 = mean(survey_data[[c1_col]][valid] == survey_data[[c2_col]][valid]),
      AI_C1 = mean(survey_data[[ai_col]][valid] == survey_data[[c1_col]][valid]),
      AI_C2 = mean(survey_data[[ai_col]][valid] == survey_data[[c2_col]][valid])
    )
  })
  
  results |>
    mutate(across(c(C1_C2, AI_C1, AI_C2), ~round(.x * 100, 1)))
}

# Run comparison
agreement_table <- compare_all_variables(survey_data)
print(agreement_table)
```

**Example output:**
```
  variable n_cases C1_C2 AI_C1 AI_C2
1 code        1823  95.2  94.8  95.1
2 bexpment    1823  96.3  95.1  95.8
3 wexpment    1823  97.1  96.4  96.9
4 tblk        1823  88.7  86.2  87.5
5 individ     1823  89.3  87.8  88.5
6 discrim     1823  85.4  83.9  84.7
7 welf        1823  98.2  97.8  98.1
8 obama       1823  99.1  98.9  99.0
9 rxn         1823  92.3  91.1  91.8
```

## Interpreting the Results

### What's Good Agreement?

| Agreement Rate | Interpretation |
|---------------|----------------|
| > 95% | Excellent |
| 90-95% | Very good |
| 85-90% | Good |
| 80-85% | Acceptable |
| < 80% | Problematic |

### Typical Patterns

**Easy variables** (high agreement):
- `code`, `welf`, `obama` → Binary, objective
- Agreement typically 95%+

**Moderate variables** (medium agreement):
- `individ`, `discrim` → Multi-category, some judgment
- Agreement typically 85-90%

**Hard variables** (lower agreement):
- `tblk` → Subtle distinctions in tone
- Agreement typically 80-85%

## Confusion Matrices

See where disagreements occur:

```r
library(caret)

# Create confusion matrix for 'individ'
valid <- !is.na(survey_data$C1Q1individ) & !is.na(survey_data$AI_Q1individ)

confusion <- confusionMatrix(
  factor(survey_data$AI_Q1individ[valid]),
  factor(survey_data$C1Q1individ[valid])
)

print(confusion$table)
```

**Example output:**
```
          Reference
Prediction   0   1   2   3
         0 512   8   3   2
         1  12 780  15   4
         2   5  18 210   7
         3   1   5   9 232
```

**Interpretation:**
- Diagonal = agreements
- Off-diagonal = where Claude/human disagree
- Most disagreements on adjacent categories (1↔2, 2↔3)

## Identifying Problematic Responses

Find responses where Claude and humans strongly disagree:

```r
# Responses with 3+ variable disagreements
problematic <- survey_data |>
  filter(!is.na(AI_Q1code)) |>
  mutate(
    n_disagreements = 
      (AI_Q1code != C1Q1code) +
      (AI_Q1individ != C1Q1individ) +
      (AI_Q1discrim != C1Q1discrim) +
      (AI_Q1bexpment != C1Q1bexpment) +
      (AI_Q1wexpment != C1Q1wexpment)
  ) |>
  filter(n_disagreements >= 3) |>
  select(Q43, starts_with("AI_"), starts_with("C1Q1"))

cat(sprintf("Found %d highly problematic responses\n", nrow(problematic)))

# Examine a few
problematic |>
  slice_head(n = 3) |>
  select(Q43, AI_Q1individ, C1Q1individ, AI_Q1discrim, C1Q1discrim)
```

**Common patterns in problematic responses:**
- Very short/vague: "yes", "okay", "nothing"
- Sarcasm or irony (hard to detect)
- Ambiguous references
- Complex, multi-layered arguments

## Cohen's Kappa (Advanced)

For a more sophisticated reliability measure:

```r
library(psych)

# Cohen's kappa for 'individ'
valid <- !is.na(survey_data$C1Q1individ) & !is.na(survey_data$AI_Q1individ)

kappa_ai_c1 <- cohen.kappa(
  cbind(survey_data$AI_Q1individ[valid], 
        survey_data$C1Q1individ[valid])
)

cat(sprintf("Cohen's kappa (AI-C1): %.3f\n", kappa_ai_c1$kappa))
```

**Kappa interpretation:**
- > 0.80: Excellent
- 0.60-0.80: Good
- 0.40-0.60: Moderate
- < 0.40: Poor

**Note:** Kappa is more stringent than percent agreement because it accounts for chance agreement.

## Visualizing Agreement

```r
library(ggplot2)

# Agreement rates by variable
agreement_table |>
  pivot_longer(cols = c(C1_C2, AI_C1, AI_C2), 
               names_to = "comparison", 
               values_to = "agreement") |>
  ggplot(aes(x = variable, y = agreement, fill = comparison)) +
  geom_col(position = "dodge") +
  geom_hline(yintercept = 90, linetype = "dashed", color = "red") +
  labs(
    title = "Agreement Rates: Human vs AI Coders",
    subtitle = "Dashed line = 90% threshold",
    x = "Variable",
    y = "Agreement (%)",
    fill = "Comparison"
  ) +
  theme_minimal() +
  coord_flip()
```

## When Can You Trust Claude?

Based on validation, establish guidelines:

### Use Claude Codes Directly When:
✅ Agreement > 95% (e.g., `obama`, `welf`, `code`)  
✅ Low-stakes analysis (exploratory)  
✅ Large sample (errors average out)

### Use with Caution When:
⚠️ Agreement 85-90% (e.g., `individ`, `discrim`)  
⚠️ Critical to your research question  
⚠️ Small sample (errors compound)

### Human Review Needed When:
❌ Agreement < 85% (e.g., `tblk` if low)  
❌ High-stakes decisions  
❌ Legal/clinical applications

## Hybrid Approach: AI + Human

**Best practice:** Use Claude for initial coding, human review for uncertain cases:

```r
# Flag responses for human review
survey_data <- survey_data |>
  mutate(
    needs_review = case_when(
      AI_Q1code == 0 ~ TRUE,  # Not codeable
      nchar(Q43) < 10 ~ TRUE,  # Very short
      # Add rules for low-confidence cases
      TRUE ~ FALSE
    )
  )

# Responses needing human review
review_set <- survey_data |>
  filter(needs_review) |>
  select(Q43, starts_with("AI_"))

cat(sprintf("%d responses flagged for review (%.1f%%)\n", 
            nrow(review_set), 
            100 * nrow(review_set) / nrow(survey_data)))
```

## Exercise: Validate Your Q43 Codes

```r
# 1. Calculate agreement across all variables
agreement <- compare_all_variables(survey_data)

# 2. Find average agreement
avg_human <- mean(agreement$C1_C2)
avg_ai <- mean(c(agreement$AI_C1, agreement$AI_C2))

cat(sprintf("Average human agreement: %.1f%%\n", avg_human))
cat(sprintf("Average AI agreement: %.1f%%\n", avg_ai))

# 3. Identify lowest-agreement variables
agreement |>
  arrange(AI_C1) |>
  head(3)

# 4. Examine problematic responses for those variables
```

## Writing It Up for Publication

**In your methods section:**

> "Open-ended survey responses were coded using Claude Haiku 4.5 (Anthropic, 2024) 
> with a structured codebook prompt (temperature=0 for reproducibility). We validated 
> the approach by comparing AI codes with double-coded human responses (n=1,823). 
> Agreement between AI and human coders averaged 88.2% (range: 83.9-98.9% across 9 
> variables), comparable to inter-rater reliability between human coders (89.1%). 
> The highest agreement was on objective variables (code presence, explicit mentions), 
> with lower but acceptable agreement on subjective judgments (trait attributions). 
> All analyses used AI codes; robustness checks with human codes (n=200 randomly 
> selected) yielded substantively identical results."

## What's Next?

You've successfully validated your LLM coding approach!

In the [final section](next-steps.html), we'll discuss how to extend this to other questions and applications beyond survey coding.

---

## Key Takeaways

- 📊 **88-94% agreement** is typical and comparable to human IRR
- 🎯 **Simple variables** (mentions, binary) → 95%+ agreement
- 🤔 **Complex variables** (traits, judgments) → 85-90% agreement
- ⚠️ **Always validate** against human-coded subset
- 📝 **Report transparently** in publications
- 🔍 **Spot-check** problematic responses

## Quick Reference

```r
# Compare all variables
agreement <- compare_all_variables(survey_data)

# Flag problematic responses
problematic <- survey_data |>
  filter(many_disagreements | very_short | ambiguous)

# Hybrid approach: AI + human review
review_these <- survey_data |> filter(needs_review)
```
