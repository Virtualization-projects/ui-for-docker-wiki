Clone the repo, then run the following commands to get DockerUI running locally:
```
make install
grunt # Run tests
make build-release # Create the dockerui binary
make build run # (re)build image and container
```

After making changes, run: `make build run` to update the container.


Note: Builds are a messy combo of make and grunt right now with no easy way to do live updates during dev. @mplewis is [experimenting with a WebPack build](https://github.com/mplewis/dockerui/tree/webpack), but it's also fairly complicated.