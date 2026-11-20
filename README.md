# Client site

A minimal static website served by Nginx, used to demonstrate a separate
client repository and automated Docker deployment.

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
DOCKERHUB_USERNAME=your-dockerhub-user-or-org
IMAGE_TAG=latest
```

Then run:

```sh
docker compose pull
docker compose up -d
docker compose ps
```

## GitHub Actions CI/CD

`.github/workflows/deploy.yml` verifies the site and builds the image on pull
requests targeting `main`; pull requests do not publish or deploy. A push to
`main` or a manual workflow run publishes two public Docker Hub tags:

- `DOCKERHUB_USERNAME/client:<commit-sha>`
- `DOCKERHUB_USERNAME/client:latest`

After publishing, the workflow runs on a private self-hosted GitHub Actions
runner on the client VM. It writes the commit SHA into `.env`, pulls that exact
image, recreates the Compose service, and sends a Slack notification if
configured.

### GitHub configuration

In this repository's **Settings → Secrets and variables → Actions**, add:

| Type | Name | Value |
| --- | --- | --- |
| Variable | `DOCKERHUB_USERNAME` | Docker Hub user or organization that owns the public `client` image |
| Secret | `DOCKERHUB_TOKEN` | Docker Hub access token with permission to push |
| Secret (optional) | `SLACK_WEBHOOK_URL` | Slack incoming webhook URL |

Create the public Docker Hub repository `client` under that user or
organization. Use an access token instead of a Docker Hub password.

### Client VM runner

Install Docker Engine and the Docker Compose v2 plugin on the Linux client VM.
In this repository, go to **Settings → Actions → Runners → New self-hosted
runner**, follow GitHub's registration instructions on the VM, add the custom
runner label `client-vm`, and install/start the runner as a service. The
workflow requires labels `self-hosted`, `linux`, and `client-vm`; confirm the
runner is online before deploying. The runner account must be able to run
`docker compose` non-interactively.

The VM must be able to access GitHub and Docker Hub over HTTPS. Open port `8080`
in the VM/network firewall only if the website should be reachable directly.
Do not run untrusted pull request code on this deployment runner.

## Roll back

To redeploy an older version, set `IMAGE_TAG` in the VM checkout's `.env` to a
previously published commit SHA and run `docker compose pull` followed by
`docker compose up -d`. The normal workflow always deploys the current commit's
immutable SHA tag.
