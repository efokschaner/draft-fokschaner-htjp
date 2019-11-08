# Source for RFC 8565 Hypertext Jeopardy Protocol (HTJP/1.0)
This repo is the markdown and build pipeline for the RFC. Minor caution, the published RFC has edits not yet propagatged back to this source.

## Building on Windows

    # Needed once, or whenever you modify the build process
    docker build -t efokschaner/rfctools:latest ./docker
    # Iterate on the doc by running:
    docker run --rm -v "%cd%:/rfc" efokschaner/rfctools
