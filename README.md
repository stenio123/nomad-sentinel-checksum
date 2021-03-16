# Secure Production Deployments with Nomad + Sentinel
Sentinel can be used to track an 'allow list' of checksums for the container images or artifacts that are allowed in a deployment.

This can be used to increase security in production environments, ensuring that only container images or artifacts with an approved checksum registered by a trusted CICD can be approved for deployment.

## Container Images

```
# CICD pipeline that promotes to prod pulls image
export IMAGE=nginx:1.19.8-alpine
docker pull $IMAGE

# calculate checksum
export IMAGEVERSIONSHA=$(docker inspect --format='{{index .RepoDigests 0}}' $IMAGE)
# or to include the version
# https://github.com/hashicorp/nomad/issues/9644
export IMAGEVERSIONSHA=$(docker inspect --format='{{index .RepoTags 0}}@{{ .Id}}' $IMAGE)

# Updates Nomad server Sentinel policy
sed -i "s/allowed_images =\[/allowed_images =\[\n\"$IMAGEVERSIONSHA\",/g" ../sentinel/restrict-docker-images.sentinel

nomad sentinel apply -level=hard-mandatory restrict-docker-images ../sentinel/restrict-docker-images.sentinel
nomad job run webserver-test.nomad
```

## Artifacts Images

TODO: Is there a way to make Sentinel policy dynamic, or to track checksums in a separate file?