# kunihir0 Aidoku Sources

This repository hosts a collection of high-quality sources tailored for the Aidoku app.

## Overview

The sources in this repository are built automatically using GitHub Actions. The build process packages the source extensions and generates a repository manifest (`index.min.json`) that can be added directly to your Aidoku application.

This repository also contains a modern, beautiful landing page serving the source list so users can easily browse and add the repository with a single click.

## Usage

To use these sources, simply add the repository to your Aidoku app:
1. Visit the [Source List Website](https://kunihir0.github.io/aidoku-sources/) (adjust URL if different).
2. Click the **"Add Repository to Aidoku"** button.

Alternatively, you can manually add this URL as a source list in Aidoku:
`https://kunihir0.github.io/aidoku-sources/index.min.json`

## Adding New Sources

To add a new Aidoku source:
1. Add the source repository as a Git submodule under the `sources/` directory.
   ```bash
   git submodule add <repository-url> sources/<source-name>
   ```
2. Commit and push the changes. The GitHub Action will automatically build and deploy the new source.

## License

Please refer to the individual source submodules for licensing information.
