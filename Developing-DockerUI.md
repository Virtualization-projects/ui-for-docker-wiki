## Quickstart
Make sure you have Docker installed on your machine. Instructions for this are available [on the Docker website](https://docs.docker.com/installation/).

Clone the repo and run the following commands to get DockerUI running locally:
```
git clone https://github.com/crosbymichael/dockerui.git
cd dockerui
npm install
grunt  # Run unit tests
grunt run-dev
```

## Convenient Grunt targets
### default
Runs jshint, builds the app, and runs tests.

### build
Builds the golang binary and Angular app with no minification.

### release
Build the golang binary and Angular app with minification. The output of `grunt release` is committed to the `dist` branch of this repo and is used by the Docker Hub to build the official image.

### test-watch
Watch files for changes and run unit tests on every change.

### run
Build and run a Docker image with the app.

### run-dev
Build and run a Docker image with the app, watch source files and refresh on changes.

The actual docker commands for `run` and `run-dev` are in the Gruntfile under the `shell` section, this is the place to make changes, like pointing at a different Docker host or changing the port that DockerUI is exposed on.