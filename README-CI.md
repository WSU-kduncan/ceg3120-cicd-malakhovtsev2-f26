# Part 1 - Docker container image

### Website Content
- [index.html](web-content/index.html)
- [other.html](web-content/other.html)
- [style.css](web-content/style.css)

### Dockerfile

[Dockerfile](Dockerfile)

- `FROM` sets the base image
- `COPY` puts the site into a default httpd web folder `/usr/local/apache2/htdocs`

