source "https://rubygems.org"

# GitHub Pages gem - includes Jekyll and common plugins
gem "github-pages", group: :jekyll_plugins

# Uncomment below for local development without GitHub Pages dependency
# gem "jekyll", "~> 3.9"
# gem "jekyll-remote-theme"
# gem "jekyll-seo-tag"
# gem "jekyll-sitemap"

# Windows-specific
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Lock http_parser.rb to version compatible with JRuby
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
