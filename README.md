# LLM-Powered Survey Coding Workshop

A 2.5-hour hands-on workshop teaching social scientists how to use Claude API for systematic coding of open-ended survey responses.

## Quick Start

### For GitHub Pages

1. Fork this repository
2. Go to Settings → Pages
3. Select "Deploy from branch" → `main` → `/ (root)`
4. Your site will be at `https://yourusername.github.io/llm-survey-coding-workshop/`

### For Local Development

```bash
# Install Jekyll
gem install bundler jekyll

# Install dependencies
bundle install

# Serve locally
bundle exec jekyll serve

# View at http://localhost:4000
```

## Workshop Structure

**Duration:** 2.5 hours

1. **Setup** (15 min) - Get API key and environment ready
2. **Motivation** (10 min) - Why LLMs for survey coding?
3. **First API Call** (20 min) - Basic API usage
4. **Codebook** (30 min) - Structured coding with JSON
5. **Batching** (25 min) - Efficient processing
6. **Production** (30 min) - Robust system with progress saving
7. **Validation** (15 min) - Compare with human codes
8. **Next Steps** (10 min) - Extensions and applications

## Materials Included

- ✅ 11 comprehensive tutorial pages
- ✅ Sample survey data (1,944 responses)
- ✅ Complete R scripts
- ✅ 9-variable codebook
- ✅ Troubleshooting guide

## Requirements

**Software:**
- R 4.0+
- RStudio (recommended)
- R packages: `tidyverse`, `httr2`, `jsonlite`

**API Access:**
- Anthropic API key (free $5 credit)
- Sign up at [console.anthropic.com](https://console.anthropic.com)

**Knowledge:**
- Intermediate R programming
- Basic survey research concepts
- Familiarity with API concepts helpful but not required

## Customization

### Update Site Settings

Edit `_config.yml`:
```yaml
title: "Your Workshop Title"
url: "https://yourusername.github.io"
baseurl: "/your-repo-name"
```

### Modify Content

All workshop pages are in the root directory:
- `index.md` - Home page
- `setup.md` - Installation guide
- `motivation.md` - Background
- `first-call.md` - First API call
- ...and so on

### Add Your Data

Replace placeholder links in `resources.md`:
- Your survey data
- Your scripts
- Your contact information

## License

MIT License - free to use, modify, and distribute with attribution.

## Citation

```bibtex
@misc{llm_survey_coding_workshop,
  title = {LLM-Powered Survey Coding with Claude: A Hands-On Workshop},
  author = {Your Name},
  year = {2026},
  howpublished = {\url{https://yourusername.github.io/llm-survey-coding-workshop/}},
}
```

## Contact

- **Instructor:** Your Name (your.email@institution.edu)
- **Issues:** [GitHub Issues](https://github.com/yourusername/llm-survey-coding-workshop/issues)

## Acknowledgments

Built using:
- Jekyll static site generator
- Dinky theme by GitHub Pages
- Claude API by Anthropic
