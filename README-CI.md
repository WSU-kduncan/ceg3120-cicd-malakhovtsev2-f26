# Part 1 - Docker container image

```mermaid
flowchart LR
    subgraph Local["Local machine (cloned repository)"]
        direction TB
        A["Edit web-content / Dockerfile"] --> B["git commit + git push origin main"]
        B --> C["git tag -a vX.Y.Z"]
        C --> D["git push origin vX.Y.Z"]
        P["docker pull / docker run<br/>any version tag"]
    end

    subgraph GitHub["GitHub"]
        direction TB
        E{"Pushed ref is a<br/>v*.*.* tag?"}
        E -- "no (plain commit)" --> N["No workflow run"]
        E -- yes --> F["GitHub Actions workflow starts"]
        F --> G["metadata-action<br/>generates latest, X, X.Y"]
        G --> H["login-action<br/>DOCKER_USERNAME + DOCKER_TOKEN secrets"]
        H --> I["setup-buildx-action"]
        I --> J["build-push-action<br/>builds image from Dockerfile"]
    end

    subgraph DockerHub["DockerHub"]
        direction TB
        K["malakhovtsev/ceg3120-cicd<br/>tags: latest, X, X.Y<br/>older X.Y tags kept"]
    end

    B --> E
    D --> E
    J -- push image --> K
    K -- pull --> P
```

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

### Workflow Trigger
Workflow runs only when commit is pushed to `main` branch:

```yaml
on:
  push:
    branches:
        - main
```

### Workflow Steps
Workflow runs `build` job:
1. Sign in DockerHub: `docker/login-action@v4`
2. Logs into DockerHub with `DOCKER_USERNAME` and `DOCKER_TOKEN` GitHub repository secrets.
3. Builds and pushes Docker image: `docker/push-action@v7` builds the image from the repository's Dockerfile and pushes it to DockerHub as hardcoded `malakhovtsev/ceg3120-cicd:latest`.

### Setup: Values to Change for Another Repository
[Part 2 workflow file (permalink)](https://github.com/WSU-kduncan/ceg3120-cicd-malakhovtsev2-f26/blob/a4794af35cad3ed2dc9733249e3d808c24b615d6/.github/workflows/docker-build.yml)

#### In the workflow file
- change hardcoded `images: malakhovtsev/ceg3120-cicd` to own `<dockerhub-username>/<repository>`.
- if repository has different main/CI branch, change `branches: - main` to a branch name accordingly
- Secrets: `secrets.DOCKER_USERNAME` and `secrets.DOCKER_TOKEN` can be changed if repository has different secrets name.

#### In the repository
- Create secrets `DOCKER_USERNAME` and `DOCKER_TOKEN` with own DockerHub username and PAT.
- Create DockerHub repository for the Docker image.
- Create/keep workflow file in `.github/workflows/` for GitHub.

### Testing & Validating
#### Testing that workflow does its job:
1. Push any commit to `main` branch
2. Open **Actions** tab on the repository on GitHub. Run for that commit should have a green checkmark.
3. On DockerHub, the `latest` tag should have the latest push time recent accordingly to the push time.

#### Testing that Docker image works
DockerHub repository: https://hub.docker.com/repository/docker/malakhovtsev/ceg3120-cicd
```bash
docker pull --platform linux/amd64 malakhovtsev/ceg3120-cicd:latest
docker run -d --platform linux/amd64 --name cicd-site -p 8080:80 malakhovtsev/ceg3120-cicd:latest
```

### Main trigger
![successful CI job](images/successful-ci-job.png)
> Workflow run trigger by a push to `main` that is completed successfully.

### Branch negative test

![main last commit before branch negative test](images/ci-job-last-commit-main.png)
> Latest commit on `main` branch has green check showing the CI job is working

![no ci job on non main branch commit](images/no-ci-job-test-branch.png)
> No green checkmark on a `test-branch` on a commit confirms the CI job only runs on `main` branch

![no workflow started on another branch](images/no-workflow-started-another-branch.png)
> No workflow started on `test-branch` as required

### Change to Docker image

![CI job with updated text](images/ci-job-with-updated-text.png)
> Updated text in the web-content and committed and pushed to `main`, confirmed with green check on a commit

![terminal showing docker pull after CI job](images/terminal-showing-docker-pull-after-CI-job.png)
> After removing old image, `docker pull` shows that it downloaded newer image confirming workflow pushed updated image to DockerHub

![browser showing updated text after CI job](images/browser-showing-updated-text-after-CI-job.png)
> Container from updated pulled image has the updated content, as visible with updated text in the browser, confirming the image is updated by CI workflow

# Part 3 - Semantic Versioning
Tags are going to be used instead of overwriting `latest` on every push, images are only going to be built when tag is pushed to not trigger the workflow for every small unrelated commit. This also keeps older versions on DockerHub to roll back in case needed.

### Generating Tags
#### See existing tags:
```bash
git tag -l
```

#### Create and annotated tag on the latest commit:
```bash
git tag -a v1.1.0 -m "Second release: version label on home page"
```

#### Push the tag to GitHub:
```bash
git push origin v1.1.0
```

### Workflow Trigger
Workflow only runs when a tag is matching the structure: `v*.*.*` is pushed:

```yaml
on:
  push:
    tags:
      - 'v*.*.*'
```

### Workflow Steps

1. **Generate Docker metadata**: `docker/metadata-action@v6`, reads pushed git tag and generates image tags.
    - `type=semver,pattern={{major}}` -> `1`
    - `type=semver,pattern={{major}}.{{minor}}` -> `1.1`
    - `latest` is added automatically for tag pushes
2. **Sign in to DockerHub**: `docker/login-action@v4` logs in with the `DOCKER_USERNAME` and `DOCKER_TOKEN` repository secrets.
3. **Set up Docker Buildx**: `docker/setup-buildx-action@v4` setups up the builder
4. **Build and push Docker image**: `docker/build-push-action@v7` builds the image and pushes it with all the tags generated in step 1.

For a `v1.1.0` Docker receives `latest`, `1` and `1.1`.

[Workflow file](.github/workflows/docker-build.yml)

### Setup: Values to change for another repository

#### In the workflow file
- change hardcoded `tags: malakhovtsev/ceg3120-cicd:latest` to own `<dockerhub-username>/<repository>:<tag>`.
- Secrets: `secrets.DOCKER_USERNAME` and `secrets.DOCKER_TOKEN` can be changed if repository has different secrets name.
- Tags: `tags: 'v*.*.*'` if using different format

#### In the repository
- Create secrets `DOCKER_USERNAME` and `DOCKER_TOKEN` with own DockerHub username and PAT.
- Create DockerHub repository for the Docker image.
- Create/keep workflow file in `.github/workflows/` for GitHub
- Create and push tags in `vMAJOR.MINOR.PATCH` format to trigger a build

### Testing & Validating
#### Testing that workflow does its job:
1. Push a tag
2. Open **Actions** tab on the repository on GitHub. Run for that commit should have a green checkmark and no `main` tag
3. On DockerHub, the **Tags** tab should show `latest`, `1` and `1.1` with recent push times according to push to GitHub time.

#### Testing that Docker image works
DockerHub repository tags: https://hub.docker.com/r/malakhovtsev/ceg3120-cicd/tags

```bash
docker run -d --platform linux/amd64 --name site1.0 -p 8080:80 malakhovtsev/ceg3120-cicd:1.0
docker run -d --platform linux/amd64 --name site1.1 -p 8081:80 malakhovtsev/ceg3120-cicd:1.1
```
Then opening each `localhost` with according ports will shows different tags/versions of the image.

#### Tag trigger
![git tag -l input](images/git-tag-terminal-input.png)
![git tag -l output](images/git-tag-terminal-output.png)
> `git tag -l` shows tag `v1.0.0` in the local repository

![summary tag triggered workflow](images/summary-tag-triggered-workflow.png)
> The workflow run was triggered by pushed tag `v1.0.0` and completed successfully

#### Tag negative test
![tag negative commit](images/tag-negative-commit.png)
> Tag negative commit. As visible, the commit to `main` after previous tagged one did not trigger the workflow

#### Tag set
![tag on dockerhub](images/tag-on-dockerhub.png)
> After `v1.0.0` release, DockerHub shows all tags: `latest`, `1`, `1.0` pushed by the workflow

#### Two releases
![dockerhub releases information](images/dockerhub-releases-information.png)
> After releasing `v1.1.0`, `latest`, `1` and `1.1` point to the new image `7267d8c203b3`, while `1.0` still points to the original image `d93b2fb0a513`.
>

#### Rollback
![docker ps of different tags](images/docker-ps-of-different-tags.png)
> Containers from image tags `1.0` and `1.1` run at the same time on ports `8080` and `8081` accordingly

![website from 1.0 tag image](images/website-from-10-tag-image.png)
> `1.0` image still shows original heading

![website from 1.1 tag image](images/website-from-11-tag-image.png)
> `1.1` shows updated heading of second release