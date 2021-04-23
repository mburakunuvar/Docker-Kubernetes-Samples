### deployment-01 test on local machine => development

```bash
$ docker build -t node-deployment-sample-image .
$ docker run -d --rm --name node-deployment-sample-container -p 80:80 node-deployment-sample-image
```

### deployment-02 test on remote host => production

- no need for bind mounts
- make sure COPY ing code snapshots into the image
- use docker hub or other repos to host images

```bash
# no need for bind mounts
$ docker run -v
# build and push image
$ docker build -t node-deployment-sample-image .
$ docker tag node-deployment-sample-image burakunuvar/node-deployment-sample-image
$ docker push burakunuvar/node-deployment-sample-image

# run the image
$ docker run -d --rm --name node-deployment-sample-container -p 80:80 burakunuvar/node-deployment-sample-image

# re-run updated image
$ docker pull burakunuvar/node-deployment-sample-image
docker run -d --rm -p 80:80 burakunuvar/node-deployment-sample-image

```
