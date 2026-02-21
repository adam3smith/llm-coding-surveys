---
layout: default
title: "Troubleshooting"
nav_order: 10
---

# Troubleshooting Guide
{: .no_toc }

## Common Issues and Solutions

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## API Connection Issues

### Error: "x-api-key header is required"

**Cause:** API key not set or not being passed correctly

**Solutions:**
```r
# Check if key is set
Sys.getenv("ANTHROPIC_API_KEY")

# If empty, set it
Sys.setenv(ANTHROPIC_API_KEY = "sk-ant-api-your-key-here")

# Or add to .Renviron (recommended)
usethis::edit_r_environ()
# Add line: ANTHROPIC_API_KEY=sk-ant-api-your-key-here
# Restart R
```

### Error: "Invalid API key"

**Cause:** Key is incorrect or expired

**Solutions:**
1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Check your API keys under Settings
3. Generate a new key if needed
4. Make sure you copied the entire key including `sk-ant-api` prefix

### Error: "Failed to connect to api.anthropic.com"

**Cause:** Network connectivity issues

**Solutions:**
- Check your internet connection
- Try accessing https://api.anthropic.com in browser
- Check firewall/proxy settings
- Try from different network

## Rate Limit Errors

### Error: "429 - Rate limit exceeded"

**Cause:** Hitting tokens-per-minute or requests-per-minute limits

**Solutions:**

**1. Reduce batch size:**
```r
# Instead of 20
code_all_responses(survey_data, batch_size = 15)
```

**2. Increase delays:**
```r
# Add longer pause between batches
# (modify in the code_all_responses function)
Sys.sleep(7)  # Instead of 5
```

**3. Check your rate limits:**
- Go to console.anthropic.com/settings/limits
- Verify your tier
- Request tier upgrade if needed

**4. Use retry logic:**
```r
# The call_claude_with_retry function handles this automatically
# Just re-run if it fails after max retries
```

## JSON Parsing Errors

### Error: "premature EOF" or "unexpected end of input"

**Cause:** Response was truncated (max_tokens too small)

**Solutions:**
```r
# Increase max_tokens based on batch size
max_tokens_needed <- length(responses) * 120 + 500

call_claude(prompt, max_tokens = max_tokens_needed)
```

### Error: "lexical error: invalid char in json text"

**Cause:** Response contains invalid JSON characters

**Solutions:**
```r
# The parse_coding_response function should handle this
# But you can add additional cleaning:

clean_json <- function(text) {
  text |>
    str_remove_all("```json") |>
    str_remove_all("```") |>
    str_replace_all("\\\\", "\\\\\\\\") |>  # Escape backslashes
    str_trim()
}

result <- clean_json(claude_response)
parsed <- fromJSON(result)
```

### Error: "unexpected character at line X"

**Cause:** Claude added text before/after JSON

**Solutions:**
```r
# Extract just the JSON part
extract_json <- function(text) {
  # Find content between first { and last }
  start <- str_locate(text, "\\{")[1]
  end <- str_locate_all(text, "\\}")[[1]][nrow(str_locate_all(text, "\\}")[[1]]), 1]
  str_sub(text, start, end)
}

json_only <- extract_json(claude_response)
```

## Code Quality Issues

### Issue: Low agreement with human codes

**Possible causes & solutions:**

**1. Ambiguous codebook:**
```r
# ❌ Vague
"1 = positive attitude"

# ✅ Specific
"1 = positive attitude (explicitly says blacks are hardworking, 
     successful, or overcoming adversity)"
```

**2. Need examples:**
```r
# Add examples to codebook
codebook <- '
1 = positive attitude
   Example: "blacks have worked their way up successfully"
   Example: "they are hardworking and self-reliant"

3 = negative attitude
   Example: "blacks are lazy and dependent on welfare"
   Example: "they make excuses instead of trying"
'
```

**3. Temperature not set to 0:**
```r
# Always use temperature = 0
code_batch(responses, ids, temperature = 0)
```

**4. Batch size too large:**
```r
# Try smaller batches if quality is low
code_all_responses(survey_data, batch_size = 15)
```

### Issue: Inconsistent codes across runs

**Cause:** Temperature not set to 0

**Solution:**
```r
# Verify temperature
call_claude(prompt, temperature = 0)  # Must be 0!

# Test consistency
run1 <- code_batch(test_responses, 1:10, temperature = 0)
run2 <- code_batch(test_responses, 1:10, temperature = 0)
identical(run1, run2)  # Should be TRUE
```

### Issue: Claude marks too many responses as "not codeable"

**Cause:** Codebook definition of "codeable" is too strict

**Solution:**
```r
# Revise the "code" variable definition
"0 = no (blank, 'don't know', or ONLY generic agreement/disagreement 
         with no elaboration)
 1 = yes (any substantive reaction, even brief)"
```

## Performance Issues

### Issue: Processing is very slow

**Possible causes & solutions:**

**1. Too small batch size:**
```r
# Increase from 10 to 20
code_all_responses(survey_data, batch_size = 20)
```

**2. Too long delays:**
```r
# Reduce if not hitting rate limits
# (modify Sys.sleep() in code_all_responses)
Sys.sleep(3)  # Instead of 5
```

**3. Network latency:**
```r
# Not much you can do except:
# - Use faster internet connection
# - Run during off-peak hours
```

### Issue: Running out of memory

**Cause:** Accumulating too much data in memory

**Solution:**
```r
# Save more frequently
code_all_responses(
  survey_data,
  save_every = 3  # Instead of 5
)

# Or process in smaller chunks
chunk1 <- code_all_responses(survey_data |> slice(1:1000))
chunk2 <- code_all_responses(survey_data |> slice(1001:2000))
```

## Data Issues

### Issue: Codes not merging properly

**Cause:** Row number misalignment

**Solution:**
```r
# Verify original_row column exists
coded_results |> select(response_id, original_row)

# Use explicit merge
survey_data <- survey_data |>
  mutate(original_row = row_number()) |>
  left_join(coded_results |> select(-response_id), 
            by = "original_row") |>
  select(-original_row)
```

### Issue: Missing codes (NAs) after merge

**Cause:** Some responses weren't coded

**Solutions:**
```r
# Check which responses are missing codes
missing <- survey_data |>
  filter(!is.na(Q43), Q43 != "", is.na(AI_Q1code))

cat(sprintf("%d responses missing codes\n", nrow(missing)))

# Re-code just the missing ones
missing_codes <- code_batch(
  missing$Q43,
  missing |> mutate(row = row_number()) |> pull(row)
)
```

### Issue: Duplicate response_ids

**Cause:** Coded same data twice

**Solution:**
```r
# Check for duplicates
coded_results |>
  count(response_id) |>
  filter(n > 1)

# Keep only unique (most recent)
coded_results <- coded_results |>
  distinct(response_id, .keep_all = TRUE)
```

## Resume/Progress Issues

### Issue: Resume not working

**Cause:** Progress file corrupt or wrong format

**Solutions:**
```r
# Check if file exists
file.exists("progress_Q43.rds")

# Try to load it
progress <- readRDS("progress_Q43.rds")
str(progress)  # Should have $codes and $next_batch

# If corrupted, delete and start fresh
file.remove("progress_Q43.rds")
```

### Issue: Want to start over from beginning

**Solution:**
```r
# Delete progress file
file.remove("progress_Q43.rds")

# Or explicitly set resume=FALSE
code_all_responses(survey_data, resume = FALSE)
```

## Cost Issues

### Issue: Costs higher than expected

**Possible causes:**

**1. Batch size too small:**
```r
# Increases overhead
# Solution: Use batch_size=20
```

**2. Using wrong model:**
```r
# Check model string
# Should be: "claude-haiku-4-5-20251001"
# Not: "claude-sonnet-4-5-20250929" (12x more expensive!)
```

**3. Retrying failed batches multiple times:**
```r
# Check logs for repeated 429 errors
# Solution: Reduce batch size or increase delays
```

### Issue: Running out of free credits

**Solution:**
```r
# Add more credits at console.anthropic.com/settings/billing
# Or optimize:
# - Test on sample first (100 responses)
# - Use smaller batch sizes (cheaper per call)
# - Cache common prompts (advanced, see Anthropic docs)
```

## Getting Additional Help

### Before asking for help:

1. ✅ Check this troubleshooting guide
2. ✅ Review the error message carefully
3. ✅ Try the suggested solutions
4. ✅ Test with a small sample (10 responses)
5. ✅ Check Anthropic documentation

### When asking for help:

**Include:**
- Exact error message (full text)
- Code you're running (minimal example)
- What you've already tried
- System info: `sessionInfo()`

**Where to ask:**
- **R-related:** StackOverflow (tag: `r`, `claude-api`)
- **API-related:** Anthropic support or Discord
- **Workshop-specific:** [add contact info]

---

## Quick Diagnostic Checklist

Run through this to identify issues:

```r
# 1. API key set?
cat("API key:", substr(Sys.getenv("ANTHROPIC_API_KEY"), 1, 20), "...\n")

# 2. Packages loaded?
library(tidyverse); library(httr2); library(jsonlite)

# 3. Data loaded?
cat("Survey data rows:", nrow(survey_data), "\n")

# 4. Simple API call works?
test <- call_claude("Say 'test'", max_tokens = 50, temperature = 0)
cat("Test response:", test, "\n")

# 5. One response codes successfully?
test_code <- code_batch(survey_data$Q43[1], 1, temperature = 0)
print(test_code)

cat("\n✅ If all above work, your setup is fine!\n")
```

---

## Still Stuck?

If you've tried everything and still have issues:

1. **Simplify:** Start with absolute minimum code
2. **Isolate:** Test each component separately
3. **Document:** Screenshot the exact error
4. **Ask:** Provide full context when seeking help

Remember: Most issues have simple solutions once you identify the root cause!
