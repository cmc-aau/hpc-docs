# Documentation website for the BioCloud HPC at AAU
Built using [MkDocs](https://www.mkdocs.org/) with the [material theme](https://squidfunk.github.io/mkdocs-material/getting-started/).

# Installation and usage
Everything is markdown so `mkdocs` is not strictly needed locally. But in order to render the site locally before pushing to GitHub use the development container or install `mkdocs` and `mkdocs-material` using `pip`, then run `mkdocs serve`. Any changes pushed to GitHub will trigger a GitHub action that will automatically build and publish the website, so don't use `mkdocs build` or `mkdocs gh-deploy`.

This is a public repository and website, so don't publish anything sensitive.
