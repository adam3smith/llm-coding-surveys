---
layout: default
title: "Next Steps & Beyond"
nav_order: 10
---

# Next Steps & Beyond
{: .no_toc }

## Extending and Adapting This Approach
{: .no_toc }

<div class="code-example" markdown="1">
**Estimated time:** 10 minutes
</div>

---


---

## Adapting to Your Own Survey Data

### Step 1: Prepare Your Data

```r
# Your data structure
my_survey <- tibble(
  respondent_id = 1:n,
  open_ended_q1 = c("response 1", "response 2", ...),
  open_ended_q2 = c("response 1", "response 2", ...),
  # ... other columns
)
```

### Step 2: Create Your Codebook

```r
my_codebook <- '
CODEBOOK FOR [YOUR QUESTION]:

For each response, assign codes for:

1. theme (Main theme)
   0 = off-topic
   1 = theme A
   2 = theme B
   3 = theme C

2. sentiment (Overall sentiment)
   0 = negative
   1 = neutral
   2 = positive

3. [your variable]
   [your codes]
'
```

### Step 3: Adapt the Functions

```r
# Update create_codebook_prompt() with your codebook
# Update parse_coding_response() with your variable names
# Use code_all_responses() with your column names

my_codes <- code_all_responses(
  survey_data = my_survey,
  question_col = "open_ended_q1",
  prefix = "AI_",
  batch_size = 20
)
```

## Beyond Survey Coding: Text Classification

The same approach works for many text classification tasks!

### Use Case: Large Scale Analysis of Journal articles


> In this article, we use large language models to extract granular and validated data on about 100,000 articles published
in over 150 political science journals from 2010 to 2024. 

Briggs, Ryan C., Jonathan Mellon, and Vincent Arel-Bundock. 2026. "It Must Be Very Hard to Publish Null Results." Zr5vf_v1. Preprint, SocArXiv, February 11. https://osf.io/preprints/socarxiv/zr5vf_v1/

> In sum, through an extensive (and costly) validation process, we have demonstrated that GPT-5 mini
performs very well at recovering the ground truth data. It is clearly better than highly trained graduate
students at this specific information retrieval task. 


## Handling Longer Texts

Survey responses are short (~50 tokens). What about longer documents?

### Challenge: Context Window Limits

**Haiku 4.5 limits:**
- 200K input tokens total
- Codebook (~800) + batch texts must fit

**Example:**
- If documents average 2,000 tokens each
- Batch size must be: (200,000 - 800) / 2,000 ≈ 99 documents


### Sentiment + Reasoning

```r
# Combine coding with explanation
codebook <- '
For each response:
1. Code the sentiment: 0=negative, 1=neutral, 2=positive
2. Provide a 1-sentence justification for your code

Return as JSON:
{
  "sentiment": 2,
  "reasoning": "Respondent explicitly praises the product quality"
}
'
```

## Practices to Consider

### 1. Cache Common Prompts (Advanced)

Anthropic offers prompt caching to reduce costs:

```r
# If processing 100+ batches with same codebook
# First batch: Full cost
# Subsequent batches: 90% discount on cached portion

# Requires beta feature (see Anthropic docs)
```

### 2. Use Haiku for Initial Pass, Sonnet for Uncertain Cases

```r
# Code everything with Haiku
haiku_codes <- code_all_responses(survey_data)

# Flag low-confidence responses
uncertain <- haiku_codes |>
  filter(confidence_flag)  # Define your criteria

# Re-code uncertain cases with more powerful model
sonnet_codes <- code_batch(
  uncertain$text,
  uncertain$id,
  model = "claude-sonnet-4-6"
)
```


### 3. Version Control Your Prompts

```r
# Save prompt alongside results
codebook_v1 <- "..."  # Your codebook text

results <- list(
  codes = coded_data,
  codebook = codebook_v1,
  model = "claude-haiku-4-5-20251001",
  temperature = 0,
  date = Sys.Date()
)

saveRDS(results, "coding_results_v1.rds")
```



## Ethical Considerations

### Transparency in Reporting

**Always disclose:**
- Which model you used
- Your exact prompt/codebook
- Temperature setting
- Validation approach
- Agreement rates with human codes

### Limitations to Acknowledge

**LLMs cannot:**
- Replace human judgment entirely
- Understand deep cultural context
- Detect subtle sarcasm/irony consistently
- Be held accountable for decisions

**Best for:**
- High-volume initial coding
- Standardized, clear categories
- Research where reproducibility is key
- Cases with human validation available

### Data Privacy

 If working with sensitive data:
 - Consider local models
 - Consult SU ITS on agreement with Claude
 - Include in IRB and potentially consent


## Other Words of Caution

- Don't let the AI remove you too far from the data -- knowing your data well is essential
- Classification != interpretation. If your goal in studying text is meaning making, you can't use AIs as a shortcut
- Proprietary models currently outperform open models, but have significant drawbacks (pricing, lock-in, ethics): 
  Barrie, Christopher, Alexis Palmer, and Arthur Spirling. n.d. “Replication for Language Models.” Working Paper. https://arthurspirling.org/documents/BarriePalmerSpirling_TrustMeBro.pdf
  Spirling, Arthur. 2023. “Why Open-Source Generative AI Models Are an Ethical Way Forward for Science.” Nature 616 (7957): 413–413. https://doi.org/10.1038/d41586-023-01295-4
- LLMs drop significantly in inter-coder reliability for more complex annotation tasks. Notably, you can't benchmark LLMs against each others: they are much more similar to other LLMs then to humans:
  Yang, Eddie, Zoey Wang, Carl Zhou, and Yaosheng Xu. 2025. “Data Annotation with Large Language Models: Lessons from a Large Empirical Evaluation.” Working Paper. https://www.eddieyang.net/research/llm_annotation.pdf.

 




