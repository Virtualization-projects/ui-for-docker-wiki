## Quickstart
Make sure you have Docker installed on your machine. Instructions for this are available [on the Docker website](https://docs.docker.com/installation/).
Also, you need to install nodejs, npm and grunt on your host machine.
However, you do not need to install golang on your host machine. A dockerized golang compiler will be automatically downloaded to build the ui-for-docker binary.

Clone the repo and run the following commands to get DockerUI running locally:
```
git clone https://github.com/kevana/ui-for-docker.git
cd ui-for-docker
npm install
grunt  # Build ui-for-docker binary, Run unit tests
grunt run-dev  # Build and start an auto-reloading UI for Docker container
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

The actual docker commands for `run` and `run-dev` are in the Gruntfile under the `shell` section, this is the place to make changes, like pointing at a different Docker host or changing the port that UI for Docker is exposed on.

## Notes

### bower
Bower needs to be installed.

npm install -g bower

### grunt
Grunt needs to be installed as well

npm install -g grunt-cli

### symlink for node
If grunt does not run with an error about node is not executable, you may need to make a symlink like

ln -s /usr/bin/nodejs /usr/sbin/node