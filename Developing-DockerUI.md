First, make sure you have Docker installed on your machine. Instructions for this are available [on the Docker website](https://docs.docker.com/installation/).

Clone the repo and run the following commands to get DockerUI running locally:
```
git clone https://github.com/crosbymichael/dockerui.git
cd dockerui
make install # May need to use sudo
grunt # Run tests
make build-release # Create the dockerui binary
make build run # (re)build image and container
```

After making changes, run: `make build run` to update the container.


Note: Builds are a messy combo of make and grunt right now with no easy way to do live updates during dev. @mplewis is [experimenting with a WebPack build](https://github.com/mplewis/dockerui/tree/webpack), but it's also fairly complicated.