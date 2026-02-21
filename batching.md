---
layout: default
title: "Efficient Batch Processing"
nav_order: 6
---

# Efficient Batch Processing
{: .no_toc }

## Optimizing Cost and Speed
{: .no_toc }

<div class="code-example" markdown="1">
**Estimated time:** 25 minutes
</div>

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The Cost Problem

If we code responses one-by-one:

```r
# Coding 1,944 responses individually
for (i in 1:1944) {
  code_one_response(responses[i])
}
```

**Problems:**
- 🐌 **Slow:** 1,944 API calls × 2 seconds = ~1 hour
- 💸 **Expensive:** Codebook overhead repeated 1,944 times
- 📊 **Wasteful:** ~800 tokens of codebook text × 1,944 = 1.5M tokens

## The Batching Solution

Instead, send multiple responses per call:

```r
# Coding in batches of 20
for (batch in 1:98) {
  code_batch_of_20(responses[batch_start:batch_end])
}
```

**Benefits:**
- ⚡ **Fast:** 98 API calls × 2 seconds = ~3 minutes
- 💰 **Cheap:** Codebook overhead repeated only 98 times
- 📉 **Efficient:** ~800 tokens × 98 = 78K tokens (20x reduction!)

## Understanding Token Costs

### Input Tokens
- **Fixed overhead:** ~800 tokens (codebook + instructions)
- **Variable:** ~50 tokens per response (average)
- **Total for batch of 20:** 800 + (20 × 50) = 1,800 tokens

### Output Tokens
- **Per response:** ~120 tokens (JSON structure)
- **Total for batch of 20:** 20 × 120 = 2,400 tokens

### Cost Calculation

**Haiku 4.5 pricing:**
- Input: $0.25 per 1M tokens
- Output: $1.25 per 1M tokens

**For one batch of 20:**
```
Input:  1,800 tokens × $0.25/1M = $0.00045
Output: 2,400 tokens × $1.25/1M = $0.00300
Total per batch: $0.00345
```

**For all 1,944 responses:**
```
98 batches × $0.00345 = $0.34
```

That's remarkably cheap!

## Comparing Batch Sizes

Let's analyze different batch sizes:

| Batch Size | API Calls | Input Tokens | Output Tokens | Total Cost | Time |
|------------|-----------|--------------|---------------|------------|------|
| 5 | 389 | 389K | 467K | $0.68 | ~13 min |
| 10 | 194 | 213K | 233K | $0.34 | ~6.5 min |
| **20** | **98** | **137K** | **117K** | **$0.18** | **~3 min** |
| 30 | 65 | 110K | 78K | $0.13 | ~2 min |
| 50 | 39 | 95K | 47K | $0.10 | ~1.3 min |

**Interpretation:**
- **Batch 5 → 20:** 73% cost reduction, 4x fewer calls
- **Batch 20 → 50:** Only 44% additional savings
- **Diminishing returns** after batch size ~20-25

## Finding Your Optimal Batch Size

Three factors to consider:

### 1. Cost Efficiency
Larger batches = lower cost per response

**Recommendation:** At least 15-20

### 2. Error Recovery
If an API call fails, you lose the entire batch

**Recommendation:** Not more than 30-40

### 3. Quality/Attention
Can Claude maintain quality with 50 items at once?

**Recommendation:** Test empirically, but 20-25 is safe

## The Sweet Spot: Batch Size 20

**Why 20 is ideal:**
- ✅ 73% cost savings vs. batch=5
- ✅ Manageable error recovery
- ✅ Provably good quality (we tested this!)
- ✅ Well under context limits
- ✅ Fast enough for interactive work

## Implementing Efficient Batching

```r
code_all_responses_batched <- function(survey_data, 
                                       question_col = "Q43",
                                       batch_size = 20) {
  
  # Get non-empty responses
  responses_df <- survey_data |>
    mutate(original_row = row_number()) |>
    select(original_row, response_text = !!sym(question_col)) |>
    filter(!is.na(response_text), response_text != "") |>
    mutate(response_id = row_number())
  
  total_responses <- nrow(responses_df)
  n_batches <- ceiling(total_responses / batch_size)
  
  cat(sprintf("Coding %d responses in %d batches of ~%d\n", 
              total_responses, n_batches, batch_size))
  
  all_codes <- tibble()
  
  # Process each batch
  for (batch_num in 1:n_batches) {
    batch_start <- (batch_num - 1) * batch_size + 1
    batch_end <- min(batch_start + batch_size - 1, total_responses)
    batch_ids <- batch_start:batch_end
    
    cat(sprintf("Batch %d/%d: responses %d-%d... ", 
                batch_num, n_batches, batch_start, batch_end))
    
    # Get this batch's data
    batch_data <- responses_df |>
      filter(response_id %in% batch_ids)
    
    # Code it
    batch_codes <- code_batch(
      responses = batch_data$response_text,
      response_ids = batch_data$response_id
    )
    
    # Accumulate
    all_codes <- bind_rows(all_codes, batch_codes)
    
    cat("✓\n")
    
    # Brief pause (be nice to the API)
    Sys.sleep(1)
  }
  
  return(all_codes)
}
```

## Rate Limits: The Hidden Challenge

Even with batching, you might hit rate limits.

### Understanding Rate Limits

**Tier 1 limits (typical starting tier):**
- ✅ 50 requests per minute
- ⚠️ 50,000 INPUT tokens per minute
- ⚠️ 50,000 OUTPUT tokens per minute

### The Token Problem

With batch_size=20:
- ~4,200 tokens per request (input + output)
- At 20 requests/min: **84,000 tokens/min** ❌

You'll hit the **token-per-minute (TPM) limit**, not request limit!

### Solution: Slower Processing

```r
# Add delay between batches
for (batch_num in 1:n_batches) {
  # ... code batch ...
  
  # Wait 5 seconds between batches
  if (batch_num < n_batches) {
    Sys.sleep(5)  # ~12 requests/min = ~50K TPM
  }
}
```

### Alternative: Reduce Batch Size

```r
# Smaller batches = fewer tokens per request
code_all_responses_batched(survey_data, batch_size = 15)

# 15 responses = ~3,600 tokens
# At 12 req/min: ~43K TPM ✓
```

## Handling Rate Limit Errors

Add automatic retry with exponential backoff:

```r
call_claude_with_retry <- function(prompt, max_retries = 5, ...) {
  
  attempt <- 1
  wait_time <- 2
  
  while (attempt <= max_retries) {
    
    result <- tryCatch({
      call_claude(prompt, ...)
    }, error = function(e) {
      return(e)
    })
    
    # Check if it's a 429 rate limit error
    if (inherits(result, "error") && grepl("429", as.character(result))) {
      
      if (attempt < max_retries) {
        cat(sprintf("⚠ Rate limit hit. Waiting %ds (retry %d/%d)...\n", 
                    wait_time, attempt, max_retries))
        Sys.sleep(wait_time)
        wait_time <- wait_time * 2  # Exponential backoff
        attempt <- attempt + 1
        next
      } else {
        stop("Rate limit exceeded after maximum retries")
      }
    } else if (inherits(result, "error")) {
      stop(result)  # Different error, re-throw
    } else {
      return(result)  # Success!
    }
  }
}
```

**What this does:**
- Detects 429 errors automatically
- Waits 2s, then 4s, then 8s, then 16s...
- Retries up to 5 times
- You see progress: `⚠ Rate limit hit. Waiting 4s (retry 2/5)...`

## Estimating Total Time

With our optimized settings:

```r
# For 1,944 responses:
n_batches <- ceiling(1944 / 20)  # 98 batches
time_per_batch <- 2 + 5  # 2s API call + 5s delay
total_time <- n_batches * time_per_batch / 60  # minutes

cat(sprintf("Estimated time: %.1f minutes\n", total_time))
# Output: ~11 minutes
```

**Not bad** for coding 1,944 responses!

## Cost-Time Tradeoff

| Batch Size | Cost | Time | Notes |
|------------|------|------|-------|
| 10 | $0.34 | ~16 min | Safest for rate limits |
| 15 | $0.22 | ~13 min | Good balance |
| **20** | **$0.18** | **~11 min** | **Recommended** |
| 25 | $0.15 | ~9 min | Occasional rate limits |
| 30 | $0.13 | ~7.5 min | Frequent rate limits |

**Recommendation:** Start with 20, adjust if needed.

## Exercise: Test Different Batch Sizes

Try coding 100 responses with different batch sizes:

```r
test_data <- survey_data |>
  filter(!is.na(Q43), Q43 != "") |>
  slice(1:100)

# Test batch size 10
system.time({
  codes_10 <- code_all_responses_batched(test_data, batch_size = 10)
})

# Test batch size 20
system.time({
  codes_20 <- code_all_responses_batched(test_data, batch_size = 20)
})

# Compare results
identical(codes_10, codes_20)  # Should be TRUE (same codes)
```

**Observe:**
- Time difference
- Any rate limit warnings?
- Cost difference (track token usage)

## Monitoring Your Usage

Check your actual usage:

```r
# Track tokens across batches
total_input <- 0
total_output <- 0

for (batch_num in 1:n_batches) {
  result <- call_claude_with_usage(prompt)
  
  total_input <- total_input + result$input_tokens
  total_output <- total_output + result$output_tokens
  
  cat(sprintf("Batch %d: %d in, %d out\n", 
              batch_num, result$input_tokens, result$output_tokens))
}

cat(sprintf("\nTotal: %d input, %d output\n", total_input, total_output))
cat(sprintf("Cost: $%.4f\n", 
            (total_input / 1e6 * 0.25) + (total_output / 1e6 * 1.25)))
```

## Best Practices

**Do:**
- ✅ Use batch_size=20 as default
- ✅ Add 5-second delays between batches
- ✅ Implement retry logic for 429 errors
- ✅ Track token usage to monitor costs
- ✅ Test on small subset first

**Don't:**
- ❌ Process one-by-one (wasteful)
- ❌ Use batch_size > 40 (risky)
- ❌ Ignore rate limit errors
- ❌ Forget delays between batches
- ❌ Process all data without testing first

## What's Next?

We can now efficiently code large datasets! But what if something goes wrong mid-process?

In the [next section](production.html), we'll add progress saving and error recovery to create a production-ready system.

---

## Key Takeaways

- 📦 **Batch size 20** is the sweet spot (cost + reliability)
- 💰 **Cost scales with overhead:** Batching reduces by ~70%
- ⏱️ **Token limits** are usually the constraint, not requests
- ⚠️ **Add delays:** 5 seconds between batches prevents rate limits
- 🔄 **Retry logic:** Handle 429 errors automatically
- 📊 **~$0.18 + 11 minutes** for 1,944 responses

## Quick Reference

```r
# Optimal settings
batch_size <- 20
delay_seconds <- 5

# Process with retry
for (batch in batches) {
  codes <- call_claude_with_retry(prompt)
  Sys.sleep(delay_seconds)
}

# Estimate cost
cost <- (n_batches × 4200 tokens) × $1.50 / 1M tokens
```
