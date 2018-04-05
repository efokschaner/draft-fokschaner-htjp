# RFC on the Hypertext Jeopardy Protocol

## Building on Windows

    # Needed once, or whenever you modify the build process
    docker build -t efokschaner/rfctools:latest ./docker
    # Iterate on the doc by running:
    docker run --rm -v "%cd%:/rfc" efokschaner/rfctools
