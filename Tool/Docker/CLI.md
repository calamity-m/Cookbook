
# Delete all images

```bash
# Force delete all images and their volumes
docker rm -vf $(docker ps -aq)

# Safer prune of everything
docker system prune -a --volumes
```

# Get shell on container

```bash
docker exec -it contanier /bin/sh
```
# Run container with bash

```bash
# Override entrypoint
docker run --rm -it --entrypoint bash $IMAGE
```

# Load image from tar

```bash
docker image load < mg.tar
```