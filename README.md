# Client site

A minimal static website served by Nginx, used to demonstrate a separate
client repository and Docker image build.

## Run locally

Build and run the image:

```sh
docker build -t client:local .
docker run --rm -p 8080:80 client:local
```

Open `http://localhost:8080/`.

The included Compose file uses a public Docker Hub image. For a local pull or
deployment, create `.env` in this directory:

```dotenv
DOCKERHUB_USERNAME=ishaq0925
IMAGE_TAG=latest
```

Then run:

```sh
docker compose pull
docker compose up -d
docker compose ps
```

## GitHub Actions CI

The workflow runs on pull requests, pushes to `main`, and manual runs. It
checks that `index.html` exists, builds the Docker image, and loads it into
the runner's local Docker engine. It does not log in to Docker Hub, upload an
image, or deploy to a VM. The GitHub-hosted runner is temporary, so the local
image is available only during that workflow job.
