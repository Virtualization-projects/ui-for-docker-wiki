Merge the latest code from feature branches into `master`, the tip of `master` is always the most recent code ready for release.

Releases are tagged from the `dist` branch.

1. Merge `master` into `dist`, pulling in all changes except the `.gitignore`.

2. Run `grunt release` and commit changes in `dist/` directory

3. Tag the commit with the release version.

4. Create a new tag release in the [Docker Hub repo](https://hub.docker.com/r/uifd/ui-for-docker/~/settings/automated-builds/). (e.g. Type: `Tag`, Name: `v0.11.0`, Dockerfile Location: `/`, Docker Tag name: `0.11.0`. Make sure you click 'Save Changes'.


5. Manually trigger build of the new release.

## Pre-Releases
Between tagged releases, create a beta tag that is built from the head of the `dist` branch.

E.g. After creating the tagged release `0.7.0`, create a `0.8.0-beta` branch that always contains the latest code in the repo.