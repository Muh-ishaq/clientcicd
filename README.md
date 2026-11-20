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

The workflow checks that `index.html` exists and builds the image on pull
requests without publishing. Pushes to `main` and manual runs publish
`ishaq0925/client:<commit-sha>` and `ishaq0925/client:latest` to Docker Hub.
It does not deploy to a VM.

Add a Docker Hub access token as the `DOCKERHUB_TOKEN` secret in this
repository's **Settings → Secrets and variables → Actions**. The workflow
uses the token only for non-pull-request builds. Never commit the token or
use your account password.
