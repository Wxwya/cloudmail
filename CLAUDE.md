# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the CloudMailin documentation site, built with Nanoc (Ruby static site generator). It serves documentation for CloudMailin's inbound and outbound email services.

## Development Commands

### Local Development
```bash
# Install dependencies
bundle install

# Start development server with live reload
bundle exec guard

# Or use nanoc directly
bundle exec nanoc compile -e development
bundle exec nanoc view
```

### Building and Testing
```bash
# Compile the site
bundle exec nanoc compile

# Run validation checks (internal links, external links, etc.)
bundle exec nanoc check

# Run checks with deploy validation
bundle exec nanoc check --deploy
```

### Docker Build
```bash
# Build Docker image
docker build -t docs-cloudmailin .

# Compile using Docker
docker run --rm -v $(pwd):/app docs-cloudmailin bundle exec nanoc compile
```

## Architecture

### Content Structure
- `content/` - All documentation content in Markdown format
  - `receiving_email/` - Inbound email documentation
  - `outbound/` - Outbound email documentation
  - `features/` - Feature-specific docs (DKIM, DMARC, callbacks, etc.)
  - `http_post_formats/` - API format specifications
  - `getting_started/` - Getting started guides
  - `guides/` - Additional guides

### Layouts
- `layouts/application.haml` - Main layout wrapper
- `layouts/sidebar.haml` - Navigation sidebar (manually maintained)
- `layouts/*.erb` - Third-party integrations (analytics, comments, etc.)

### Custom Filters (lib/)
- `lib/common_marker_filter.rb` - Markdown to HTML conversion with:
  - Syntax highlighting via Rouge
  - Custom link handling (external links get `rel="nofollow"` and `target="_blank"`)
  - Image captioning in development mode
  - Auto-includes `_common_links.md` for link definitions
- `lib/algolia.rb` - Algolia search indexing (production only)
- `lib/checks.rb` - Custom nanoc checks for unprocessed ERB and references

### Helpers (lib/helpers/)
- `item_helpers.rb` - Item metadata helpers (title, description, preview, social_image, redirect_to)
- `section_helpers.rb` - Navigation helpers (items_for_section, in_section?, find_item, link_to_item)
- `openapi_helpers.rb` - OpenAPI spec handling

### Nanoc Configuration
- `config.yaml` - Nanoc configuration
- `Guardfile` - Guard configuration for live reload
- `Rules` - Compilation rules defining how content is processed

### Deployment
- GitHub Actions workflow (`.github/workflows/main.yml`) builds and deploys to S3
- Production builds index content in Algolia
- Staging and production branches deploy to different S3 buckets

## Important Notes

### Development Mode
Set `NANOC_ENV=development` to:
- Skip Algolia indexing
- Show debug warnings (missing descriptions, no_index flags)
- Display image captions
- Show item attributes at bottom of pages

### Content Frontmatter
Markdown files should include frontmatter:
```yaml
---
title: Page Title
description: Page description for SEO and previews
---
```

### Navigation
The sidebar navigation in `layouts/sidebar.haml` is manually maintained. When adding new pages, update the sidebar accordingly.

### Redirects
To create a redirect, add to frontmatter:
```yaml
---
redirect_to: /target/path/
---
```

### No Index
To exclude from search index:
```yaml
---
no_index: true
---
```

### Deprecated Content
Mark deprecated pages:
```yaml
---
deprecated: true
---
```

### Link References
Use the `_common_links.md` file for link definitions that are reused across the site.

### Code Blocks
Use fenced code blocks with language specification for syntax highlighting:
````markdown
```ruby
def example
  "code"
end
```
````

### External Links
External links automatically get `rel="nofollow"` and `target="_blank"` attributes unless they contain "cloudmailin" in the URL.
