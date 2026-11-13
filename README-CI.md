# Part 1 - Docker container image

### Website Content

Small static website with a home page to fill the requirements

- [index.html](web-content/index.html) - home page with a link to "other" page
- [other.html](web-content/other.html) - "other" sample page
- [style.css](web-content/style.css) - stylesheet

### Dockerfile

[Dockerfile](Dockerfile) builds an Apache server image that serves the website.

- `FROM` sets the base image
- `COPY` puts the site into a default httpd web folder `/usr/local/apache2/htdocs`

### Building the Image

From the root folder:
```bash
docker build --platform linux/amd64 -t malakhovtsev/ceg3120-cicd:latest .
```

- `-t` -tags the image
- `--platform linux/amd64` is needed to build for x86 servers on AWS (since my machine is ARM MacBook its incompatible by default)

Login with Personal Access Token (PAT) that can be created in settings on Docker Hub.

```bash
docker login -u malakhovtsev
docker push malakhovtsev/ceg3120-cicd:latest
```

DockerHub Repository: https://hub.docker.com/r/malakhovtsev/ceg3120-cicd

### Running the Container

Pull the image from DockerHub and run it:
```bash
docker pull --platform linux/amd64 malakhovtsev/ceg3120-cicd:latest
docker run -d --platform linux/amd64 --name cicd-site -p 8080:80 malakhovtsev/ceg3120-cicd:latest
```

- `-d` runs container in the background
- `-p 8080:80` maps port 8080 on local computer to port 80 in the container (where Apache listens)
- `--platform linux/amd64` is needed to build for x86 servers on AWS (since my machine is ARM MacBook its incompatible by default)

![docker ps output](images/docker-ps.png)
> `docker ps` shows the `cicd-site` container running from the DockerHub image with port 8080 mapped to 80

![home page in the browser](images/home-page-in-browser.png)
> Home page is served my the container on the port 8080, as seen in the browser, with stylesheets applied

![other page in the browser](images/other-page-in-browser.png)
> Second required page also loads with CSS applied

# Part 2 - GitHub Actions and DockerHub

### Repository secrets

#### To create new PAT:
1. Click on the profile icon in the top right
2. Click `Account settings`
3. On the left sidebar click `Personal access tokens`
4. Click `Generate new token`
5. Enter **name**, expiration date I set as 90 days since it is a course repository (so it does expire at some point)
6. Repository permissions: `Repo Read & Write` for needed access.

#### To setup repository secrets:
1. On the repository on GitHub click `Settings` on the top panel.
2. In `Secrets and variables` section click `Actions`.
3. Click `New repository secret`.
4. Enter according name and enter/paste the secret.

![docker secrets in github secrets](images/repo-secrets.png)
> Secrets visible no the secrets page on GitHub repository settings

- `DOCKER_TOKEN` is for personal access token allowing to login to DockerHub.
- `DOCKER_USERNAME` is the username of Docker account to know who to connect from to DockerHub.

![successful CI job](images/successful-ci-job.png)
> Workflow run trigger by a push to `main` that is completed successfully.

![main last commit before branch negative test](images/ci-job-last-commit-main.png)
> Latest commit on `main` branch has green check showing the CI job is working

![no ci job on non main branch commit](images/no-ci-job-test-branch.png)
> No green checkmark on a `test-branch` on a commit confirms the CI job only runs on `main` branch

![no workflow started on another branch](images/no-workflow-started-another-branch.png)
> No workflow started on `test-branch` as required

![CI job with updated text](images/ci-job-with-updated-text.png)
> Updated text in the web-content and committed and pushed to `main`, confirmed with green check on a commit

![terminal showing docker pull after CI job](images/terminal-showing-docker-pull-after-CI-job.png)
> After removing old image, `docker pull` shows that it downloaded newer image confirming workflow pushed updated image to DockerHub

![browser showing updated text after CI job](images/browser-showing-updated-text-after-CI-job.png)
> Container from updated pulled image has the updated content, as visible with updated text in the browser, confirming the image is updated by CI workflow