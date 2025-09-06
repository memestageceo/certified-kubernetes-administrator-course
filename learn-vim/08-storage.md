# Storage in Kubernetes

```bash

# default path for volumes on host
/var/lib/docker/volumes

docker run -it -v demo_volume:/data ubuntu:22.04

docker volume create app_data

# ro - read only volume
docker run -it -v app_data:/app:ro alpine:latest

# --volumes-from use volume set up for another container
docker run -d --name backup --volumes-from db backup-image:latest

# explicitly state all params for volume mount
docker run \
  --mount type=bind,source=/data/mysql,target=/var/lib/mysql \
  mysql

# use a volume driver
docker run -it \
  --name mysql \
  --volume-driver rexray/ebs \
  --mount src=ebs-vol,target=/var/lib/mysql \
  mysql
```

```yaml
services:
  app:
    image: app-image:latest
    volumes:
      - app_data:/data
volumes:
  app_data:
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
    - image: alpine
      name: alpine
      command: ["/bin/sh", "-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
      volumeMounts:
        - mountPath: /opt
          name: data-volume
  volumes:
    - name: data-volume
      hostPath:
        path: /data
        type: Directory
```

## Persistent Volume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```
