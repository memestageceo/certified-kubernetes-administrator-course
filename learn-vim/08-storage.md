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
  persistentVolumeReclaimPolicy: Retain
```

### Persistent Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

Reclaim policy in PVC:

- **Retain**: The PV remains in the cluster after the PVC is deleted. An administrator must manually reclaim it.
- **Delete**: The PV is automatically deleted along with the PVC, releasing the storage on the physical device.
- **Recycle**: The PV data is scrubbed before reuse by new claims.

```yaml
volumes:
  - name: log-data
    persistentVolumeClaim:
      claimName: log-data
```

### storage class

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: platinum
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: regional-pd
volumeBindingMode: WaitForFirstConsumer
```
