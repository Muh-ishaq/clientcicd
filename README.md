# Client — Always Here to Help

A simple, self-contained flower welcome page served by Nginx in Docker.

## Run locally

Build and run the image:

```sh
docker build -t client:local .
docker run --rm -p 8080:80 client:local
```

Open `http://localhost:8080/`.

The Compose service uses the published Docker Hub image and the `client`
container name. After the first successful workflow publish, create `.env` in
this directory and set `IMAGE_TAG` to a published numeric version:

```dotenv
DOCKERHUB_USERNAME=ishaq0925
IMAGE_NAME=client
IMAGE_TAG=1.1
```

Then run:

```sh
docker compose pull
docker compose up -d
docker compose ps
```

## Push-to-deploy workflow

Every push to `main` validates the page, builds and publishes the next numeric
image tag to Docker Hub, then pulls it onto the Windows self-hosted runner and
replaces the local `client` container. The deployment waits for the health
check before succeeding. The site is served at `http://localhost:8080/`.

`VERSION` (`1.0`) is the starting baseline. The first published image will be
`ishaq0925/client:1.1`; subsequent successful pushes increment the minor
number. A matching numeric Git tag records each published version.

### Required GitHub setup

1. Add a Docker Hub personal access token with **Read & Write** permission as
   the `DOCKERHUB_TOKEN` repository secret in **Settings → Secrets and
   variables → Actions**. Never commit the token or use an account password.
2. Register the Windows machine that runs Docker as a **self-hosted GitHub
   Actions runner** for this repository. Leave the `self-hosted`, `windows`,
   and `x64` labels enabled.
3. Keep Docker Desktop running and ensure the runner account can use Docker
   and Docker Compose. The deployment runner needs outbound access to GitHub
   and Docker Hub.
4. Ensure port `8080` is available. The deployment replaces the existing
   `client` container and serves the page at `http://localhost:8080/`.
5. To receive deployment result emails, add these repository Actions secrets:
   `DEPLOYMENT_NOTIFICATION_EMAIL` (recipient), `MAIL_SERVER` (SMTP host),
   `MAIL_PORT` (usually `587`), `MAIL_USERNAME` (sender account), and
   `MAIL_PASSWORD` (SMTP password or app password). For Gmail, use
   `smtp.gmail.com` and a Google App Password; do not use your regular account
   password. Set the recipient secret to the email address you provided.

The workflow sends an email after publishing and deployment finish, including
the image version, job results, and a link to the Actions run. Missing mail
secrets produce a visible warning and skip sending; SMTP authentication or
delivery errors fail the notification job visibly.

The workflow runs on pushes to `main`; pull requests do not execute on the
self-hosted machine. Since the runner can control Docker on your machine,
restrict write access to this repository to trusted collaborators.
