---
layout: default
title: "Next Steps & Beyond"
nav_order: 9
---

# Next Steps & Beyond
{: .no_toc }

## Extending and Adapting This Approach
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

## Coding Your Other Questions

You've successfully coded Q43. Now extend to Q45, Q47, and Q49:

```r
# Code all four questions
q43_codes <- code_Q43(survey_data)
survey_data <- merge_codes_into_survey(survey_data, q43_codes)

q45_codes <- code_Q45(survey_data)
survey_data <- merge_codes_into_survey(survey_data, q45_codes)

q47_codes <- code_Q47(survey_data)
survey_data <- merge_codes_into_survey(survey_data, q47_codes)

q49_codes <- code_Q49(survey_data)
survey_data <- merge_codes_into_survey(survey_data, q49_codes)

# Save complete dataset
write_csv(survey_data, "survey_data_complete.csv")
```

**Total cost:** ~$8-10 for all 4 questions  
**Total time:** ~45 minutes

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

### Use Case 1: Customer Feedback Analysis

```r
# Classify customer reviews
codebook <- '
For each review, code:
1. sentiment: 0=negative, 1=neutral, 2=positive
2. topic: 0=product quality, 1=customer service, 2=shipping, 3=price
3. action_needed: 0=no, 1=yes (requires follow-up)
'

feedback_codes <- code_all_responses(
  customer_reviews,
  question_col = "review_text",
  prefix = "AI_"
)
```

### Use Case 2: Social Media Content Moderation

```r
# Flag problematic content
codebook <- '
For each post, code:
1. contains_hate_speech: 0=no, 1=yes
2. contains_misinformation: 0=no, 1=yes
3. severity: 0=none, 1=low, 2=medium, 3=high
4. action: 0=allow, 1=flag, 2=remove
'
```

### Use Case 3: Academic Literature Screening

```r
# Screen abstracts for systematic review
codebook <- '
For each abstract, code:
1. relevant_to_research_question: 0=no, 1=maybe, 2=yes
2. study_design: 0=qualitative, 1=quantitative, 2=mixed, 3=review
3. population: 0=children, 1=adults, 2=elderly, 3=not specified
'
```

### Use Case 4: Legislative Text Analysis

```r
# Code policy documents
codebook <- '
For each bill summary, code:
1. policy_area: 0=health, 1=education, 2=economy, 3=environment, 4=other
2. partisan_lean: 0=left, 1=center, 2=right
3. implementation_cost: 0=low, 1=medium, 2=high
'
```

## Handling Longer Texts

Survey responses are short (~50 tokens). What about longer documents?

### Challenge: Context Window Limits

**Haiku 4.5 limits:**
- 200K input tokens total
- Codebook (~800) + batch texts must fit

**Example:**
- If documents average 2,000 tokens each
- Batch size must be: (200,000 - 800) / 2,000 ≈ 99 documents

### Strategy 1: Reduce Batch Size

```r
# For longer texts, use smaller batches
code_all_responses(
  long_documents,
  question_col = "full_text",
  batch_size = 5  # Instead of 20
)
```

### Strategy 2: Extract Relevant Sections

```r
# Pre-process to extract relevant paragraphs
survey_data <- survey_data |>
  mutate(
    relevant_excerpt = str_sub(long_text, 1, 1000)  # First 1000 chars
  )

# Code the excerpts
codes <- code_all_responses(
  survey_data,
  question_col = "relevant_excerpt"
)
```

### Strategy 3: Hierarchical Coding

For very long documents, use two-stage approach:

**Stage 1: Summarize**
```r
# First pass: Generate summaries
summaries <- map_chr(long_docs, function(doc) {
  prompt <- sprintf("Summarize this document in 200 words:\n\n%s", doc)
  call_claude(prompt, max_tokens = 300)
})
```

**Stage 2: Code Summaries**
```r
# Second pass: Code the summaries
codes <- code_all_responses(
  tibble(text = summaries),
  question_col = "text"
)
```

### Strategy 4: Chunk and Aggregate

```r
# Break long document into chunks
chunks <- str_split(long_doc, "(?<=\\. )")[[1]]
chunk_batches <- split(chunks, ceiling(seq_along(chunks) / 10))

# Code each chunk
chunk_codes <- map_dfr(chunk_batches, function(batch) {
  code_batch(batch, seq_along(batch))
})

# Aggregate codes (e.g., most common code, or "present in any chunk")
final_code <- chunk_codes |>
  count(theme) |>
  slice_max(n, n = 1) |>
  pull(theme)
```

## Specialized Applications

### Multilingual Coding

Claude supports 100+ languages:

```r
# Code responses in Spanish
codebook_spanish <- '
Para cada respuesta, asigna códigos para:
1. sentimiento: 0=negativo, 1=neutral, 2=positivo
2. tema: [etc.]
'

# Works the same way!
codes <- code_all_responses(
  spanish_survey,
  question_col = "respuesta_abierta"
)
```

### Multi-Label Classification

```r
# Allow multiple categories per response
codebook <- '
For each response, identify ALL that apply (0=not present, 1=present):
- mentions_cost: 0 or 1
- mentions_quality: 0 or 1
- mentions_convenience: 0 or 1
- mentions_customer_service: 0 or 1
'

# Parse as separate binary variables
```

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

## Cost Optimization Strategies

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
  model = "claude-sonnet-4-5-20250929"
)
```

### 3. Sample Instead of Full Dataset

```r
# For exploratory analysis, code a sample
sample_codes <- code_all_responses(
  survey_data |> slice_sample(n = 500),
  batch_size = 20
)

# Extrapolate distributions to full dataset
```

## Quality Assurance Best Practices

### 1. Always Start with Small Test

```r
# Never run full dataset first!
test_codes <- code_all_responses(
  survey_data |> slice_head(n = 100),
  batch_size = 20
)

# Validate thoroughly before scaling
```

### 2. Version Control Your Prompts

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

### 3. Document Everything

```r
# Create metadata file
metadata <- tibble(
  dataset = "Kam_Burge.csv",
  question = "Q43",
  n_responses = nrow(survey_data),
  model = "claude-haiku-4-5-20251001",
  temperature = 0,
  batch_size = 20,
  date_coded = Sys.Date(),
  cost_total = "$2.47",
  agreement_with_humans = "88.2%",
  notes = "Validated against double-coded subset"
)

write_csv(metadata, "coding_metadata.csv")
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

```r
# If working with sensitive data:
# - Check if API provider stores requests
# - Use on-premises models if needed
# - Anonymize data before sending

survey_data <- survey_data |>
  mutate(
    Q43_anon = str_remove_all(Q43, "\\b[A-Z][a-z]+ [A-Z][a-z]+\\b")  # Remove names
  )
```

## Getting Help

### Resources

- **Anthropic Documentation:** [docs.anthropic.com](https://docs.anthropic.com)
- **Prompt Engineering Guide:** [prompting-guide.com](https://www.promptingguide.ai/)
- **R API Packages:** `httr2`, `anthropic` (unofficial)

### Community

- **R Users:** StackOverflow tag [claude-api]
- **Anthropic Discord:** Community support
- **Academic Groups:** Computational Social Science networks

### Troubleshooting

Common issues:
1. **Rate limits** → Reduce batch size, increase delays
2. **JSON parsing errors** → Check max_tokens, validate format
3. **Low agreement** → Refine codebook, add examples
4. **Inconsistent codes** → Verify temperature=0

See [Troubleshooting](troubleshooting.html) for detailed solutions.

## Share Your Experience

Did you use this approach in your research?

- Cite this workshop: [add citation]
- Share your adaptations
- Report agreement rates in your domain
- Contribute to methodological literature

## Final Thoughts

You've learned to:
- ✅ Set up and use the Claude API
- ✅ Design effective coding prompts
- ✅ Optimize for cost and reliability
- ✅ Validate against human codes
- ✅ Build production-ready systems

**This is just the beginning!** LLMs for text analysis will only get better and cheaper.

**Key principle:** Use LLMs as a tool, not a replacement for human insight. They excel at consistent application of clear rules, but research still requires human judgment, validation, and interpretation.

---

## Key Takeaways

- 🔄 **Same approach works** for many text classification tasks
- 📚 **Longer texts** require adjusted batch sizes or chunking
- 🌍 **Multilingual** coding works out of the box
- 💰 **Further optimization** possible with caching and sampling
- 📝 **Always document** your process thoroughly
- ⚖️ **Ethical use** requires transparency and validation

## Where to Go from Here

1. **Code your other survey questions**
2. **Adapt to your own research data**
3. **Experiment with different use cases**
4. **Contribute to methodology discussions**
5. **Stay updated on new models and techniques**

**Thank you for participating in this workshop!**

For questions or feedback: [add contact info]
