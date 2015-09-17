Releases are tagged from the `dist` branch.

1. Merge `master` into `dist`, pulling in all changes except the `.gitignore`.

2. Run `grunt release` and commit changes in `dist/` directory

3. Tag the commit with the release version.

4. Add the tag in the Docker Hub repo and kick off an image build.

## Pre-Releases
Between tagged releases, create a beta tag that is built from the head of the `dist` branch.

E.g. After creating the tagged release `0.7.0`, create a `0.8.0-beta` branch that always contains the latest code in the repo.