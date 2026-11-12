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
