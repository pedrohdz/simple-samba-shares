# simple-samba-shares-docker

![Continuous Integration & Deployment](https://github.com/pedrohdz/simple-samba-shares-docker/workflows/Continuous%20Integration%20&%20Deployment/badge.svg)

Build and push:

```bash
docker build --tag pedrohdz/delete:latest .
docker push pedrohdz/delete:latest
```


Quick build & run:

```bach
docker run -ti --rm -p 8139:8139 -p 8445:8445 \
    -v $PWD/files:/opt/SHARE \
    $(docker build -q .) \
    /bin/sh
```


