## Multi Stage Build Targets

```bash
$ docker build -t sample-image .
$ docker build -t sample-image -f Dockerfile-dev .
$ docker build -t sample-image --target build -f Dockerfile-prod .

```
