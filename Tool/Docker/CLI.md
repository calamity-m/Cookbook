
# Delete all images

```bash
# Force delete all images and their volumes
docker rm -vf $(docker ps -aq)

# Safer prune of everything
docker system prune -a --volumes
```

# Run container with bash

```bash
# Runs the image with whatever the default shell is
docker exec --rm -it $IMAGE

# Run with specific shell
docker exec -rm -it $IMAGE /bin/sh

# Override entrypoint
docker run --rm -it --entrypoint bash $IMAGE
```

# Load image from tar

```bash
docker image load < mg.tar
```