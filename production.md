---
layout: default
title: "Building a robust system"
nav_order: 8
---

# Building a Production System
{: .no_toc }

## Making It Robust and Recoverable
{: .no_toc }

<div class="code-example" markdown="1">
**Estimated time:** 30 minutes
</div>

---

## What Could Go Wrong?

You're 30 minutes into coding 1,944 responses when:

- Your internet drops
- Your laptop battery dies
- You hit an unexpected API error
- Rate limits kick in and exhaust retries

**Problem:** You lose all progress and have to start over.

**Solution:** Save progress incrementally

## Progress Saving Architecture

The key insight: **Save after every N batches**

```r
# Instead of this:
all_codes <- code_all_batches()
save(all_codes)  # ❌ Lose everything if crash

# Do this:
for (batch in batches) {
  codes <- code_one_batch()
  all_codes <- bind_rows(all_codes, codes)
  
  if (batch %% 5 == 0) {  # Every 5 batches
    save(all_codes)  # ✅ Can resume from here
  }
}
```

## Implementation: Progress Files

We'll save two things:
1. **Codes collected so far**
2. **Next batch number** to process

```r
# Progress file structure
progress_data <- list(
  codes = all_codes_so_far,  # Tibble of coded responses
  next_batch = 23            # Resume from batch 23
)

saveRDS(progress_data, "progress_Q43.rds")
```

## Complete Production Function

```r
code_all_responses <- function(survey_data,
                               question_col = "Q43",
                               batch_size = 20,
                               save_every = 5,
                               output_file = "progress_Q43.rds",
                               resume = TRUE) {
  
  cat(strrep("=", 70), "\n")
  cat(sprintf("CODING ALL RESPONSES FOR %s\n", question_col))
  cat(strrep("=", 70), "\n\n")
  
  # Prepare data
  responses_df <- survey_data |>
    mutate(original_row = row_number()) |>
    select(original_row, response_text = !!sym(question_col)) |>
    filter(!is.na(response_text), response_text != "") |>
    mutate(response_id = row_number())
  
  total_responses <- nrow(responses_df)
  n_batches <- ceiling(total_responses / batch_size)
  
  cat(sprintf("Total responses: %d\n", total_responses))
  cat(sprintf("Batches: %d (size %d)\n", n_batches, batch_size))
  cat(sprintf("Saving every %d batches to: %s\n\n", save_every, output_file))
  
  # Try to resume from saved progress
  start_batch <- 1
  all_codes <- tibble()
  
  if (resume && file.exists(output_file)) {
    cat("Found existing progress file. Resuming...\n")
    saved_data <- readRDS(output_file)
    all_codes <- saved_data$codes
    start_batch <- saved_data$next_batch
    cat(sprintf("Resuming from batch %d (%d responses already coded)\n\n",
                start_batch, nrow(all_codes)))
  }
  
  # Track timing
  start_time <- Sys.time()
  
  # Process each batch
  for (batch_num in start_batch:n_batches) {
    batch_start <- (batch_num - 1) * batch_size + 1
    batch_end <- min(batch_start + batch_size - 1, total_responses)
    batch_ids <- batch_start:batch_end
    
    cat(sprintf("Batch %d/%d: responses %d-%d... ", 
                batch_num, n_batches, batch_start, batch_end))
    
    # Get this batch
    batch_data <- responses_df |>
      filter(response_id %in% batch_ids)
    
    # Code with error handling
    batch_codes <- tryCatch({
      code_batch(
        responses = batch_data$response_text,
        response_ids = batch_data$response_id
      )
    }, error = function(e) {
      cat("\n❌ ERROR in batch", batch_num, ":", e$message, "\n")
      cat("Saving progress before stopping...\n")
      saveRDS(list(codes = all_codes, next_batch = batch_num), output_file)
      stop(sprintf("Error in batch %d. Progress saved.", batch_num))
    })
    
    # Add original row numbers
    batch_codes <- batch_codes |>
      left_join(
        batch_data |> select(response_id, original_row),
        by = "response_id"
      )
    
    # Accumulate
    all_codes <- bind_rows(all_codes, batch_codes)
    
    # Progress update
    elapsed <- as.numeric(difftime(Sys.time(), start_time, units = "secs"))
    batches_done <- batch_num - start_batch + 1
    avg_per_batch <- elapsed / batches_done
    remaining <- (n_batches - batch_num) * avg_per_batch
    
    cat(sprintf("✓ (%d/%d done, ETA: %.1f min)\n",
                nrow(all_codes), total_responses, remaining / 60))
    
    # Save progress periodically
    if (batch_num %% save_every == 0 || batch_num == n_batches) {
      saveRDS(list(codes = all_codes, next_batch = batch_num + 1), output_file)
      cat(sprintf("   💾 Progress saved (%d coded)\n", nrow(all_codes)))
    }
    
    # Delay between batches
    if (batch_num < n_batches) {
      Sys.sleep(5)
    }
  }
  
  # Final save
  cat("\n")
  cat(strrep("=", 70), "\n")
  cat("✅ CODING COMPLETE!\n")
  cat(strrep("=", 70), "\n")
  cat(sprintf("Total time: %.1f minutes\n", elapsed / 60))
  
  final_file <- gsub("\\.rds$", "_FINAL.rds", output_file)
  saveRDS(all_codes, final_file)
  cat(sprintf("Results saved to: %s\n\n", final_file))
  
  return(all_codes)
}
```

## Using the Production System

### First Run

```r
# Start fresh
q43_codes <- code_all_responses(
  survey_data = survey_data,
  question_col = "Q43",
  batch_size = 20,
  save_every = 5,
  output_file = "progress_Q43.rds",
  resume = TRUE  # Will check for existing progress
)
```

**Output:**
```
======================================================================
CODING ALL RESPONSES FOR Q43
======================================================================

Total responses: 1823
Batches: 92 (size 20)
Saving every 5 batches to: progress_Q43.rds

Batch 1/92: responses 1-20... ✓ (20/1823 done, ETA: 7.5 min)
Batch 2/92: responses 21-40... ✓ (40/1823 done, ETA: 7.3 min)
...
Batch 5/92: responses 81-100... ✓ (100/1823 done, ETA: 7.1 min)
   💾 Progress saved (100 coded)
...
```

### If Something Goes Wrong

**Scenario:** Your internet drops at batch 37.

**What happens:**
```
Batch 37/92: responses 721-740... ❌ ERROR: Failed to connect
Saving progress before stopping...
Error in batch 37. Progress saved.
```

**Resume from where you left off:**
```r
# Just run the same command again!
q43_codes <- code_all_responses(
  survey_data = survey_data,
  question_col = "Q43",
  output_file = "progress_Q43.rds",
  resume = TRUE  # Tells the function to start where you left off
)
```

**Output:**
```
Found existing progress file. Resuming...
Resuming from batch 37 (720 responses already coded)

Batch 37/92: responses 721-740... ✓ (740/1823 done, ETA: 4.5 min)
...
```

## Merging Codes Back into Survey Data

Once coding is complete, merge back:

```r
merge_codes_into_survey <- function(survey_data, coded_results) {
  
  cat("Merging AI codes into survey data...\n")
  
  merged_data <- survey_data |>
    mutate(original_row = row_number()) |>
    left_join(
      coded_results |> select(-response_id),
      by = "original_row"
    ) |>
    select(-original_row)
  
  # Report
  code_cols <- names(coded_results)[str_starts(names(coded_results), "AI_")]
  n_coded <- sum(!is.na(merged_data[[code_cols[1]]]))
  
  cat(sprintf("✓ %d responses now have AI codes\n", n_coded))
  cat(sprintf("  Added columns: %s\n", paste(code_cols, collapse = ", ")))
  
  return(merged_data)
}

# Use it
survey_data <- merge_codes_into_survey(survey_data, q43_codes)

# Save
write_csv(survey_data, "survey_data_with_Q43_codes.csv")
```

## Processing All Questions

Create convenience wrappers:

**These won't currently work -- we haven't implemented the prefix option into our batch functions**
```r
# Q43 (Special Favors)
code_Q43 <- function(survey_data) {
  code_all_responses(
    survey_data,
    question_col = "Q43",
    prefix = "AI_Q1",
    output_file = "progress_Q43.rds"
  )
}

# Q45 (Generations of Slavery)
code_Q45 <- function(survey_data) {
  code_all_responses(
    survey_data,
    question_col = "Q45",
    prefix = "AI_Q2",
    output_file = "progress_Q45.rds"
  )
}

# Q47 (Less Than Deserve)
code_Q47 <- function(survey_data) {
  code_all_responses(
    survey_data,
    question_col = "Q47",
    prefix = "AI_Q3",
    output_file = "progress_Q47.rds"
  )
}

# Q49 (Try Harder)
code_Q49 <- function(survey_data) {
  code_all_responses(
    survey_data,
    question_col = "Q49",
    prefix = "AI_Q4",
    output_file = "progress_Q49.rds"
  )
}
```

### Process All Sequentially

```r
# Code all 4 questions
q43_codes <- code_Q43(survey_data)
survey_data <- merge_codes_into_survey(survey_data, q43_codes)

q45_codes <- code_Q45(survey_data)
survey_data <- merge_codes_into_survey(survey_data, q45_codes)

q47_codes <- code_Q47(survey_data)
survey_data <- merge_codes_into_survey(survey_data, q47_codes)

q49_codes <- code_Q49(survey_data)
survey_data <- merge_codes_into_survey(survey_data, q49_codes)

# Save complete dataset
write_csv(survey_data, "survey_data_with_ALL_AI_codes.csv")
```


## Troubleshooting

### Issue: "Resume" Not Working

**Check:**
```r
file.exists("progress_Q43.rds")  # Should be TRUE
progress <- readRDS("progress_Q43.rds")
str(progress)  # Should have $codes and $next_batch
```

### Issue: Codes Don't Match Row Numbers

**Solution:** Use `original_row` to track:
```r
# The function already does this!
batch_codes |>
  left_join(batch_data |> select(response_id, original_row))
```

### Issue: Want to Start Fresh

```r
# Delete progress file
file.remove("progress_Q43.rds")

# Or set resume=FALSE
code_Q43(survey_data, resume = FALSE)
```

## What's Next?

You now have a production system that can code all your survey responses!

In the [next section](validation.html), we'll co