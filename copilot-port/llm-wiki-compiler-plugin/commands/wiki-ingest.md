# Ingest Web Resources Into Wiki

Download URLs into `./wiki-ingest/` and treat it as a source directory, convert the content to markdown, and run compilation.

## Instructions

1. Download each URL in parallel via a sub agent.

2. Convert each downloaded page to markdown.

3. Store converted to markdown files as original page name but with `.md` extension.

4. **Invoke the `wiki-compile` command** to initiate compilation.

## Arguments

- `--urls`: a list of urls, comma separated or one per line.
