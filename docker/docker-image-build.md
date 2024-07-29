### How To Create and Push a Docker Image

`dockerfile` example

```txt
FROM nginx:latest
RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginx-default.conf /etc/nginx/conf.d/default.conf
COPY html /usr/share/nginx/html
```

Create a Docker Image 

```bash
docker build -t nginx-image .
```

Run the Docker Image 

```bash
docker run --name custom-nginx -d -p 8080:80 nginx-image:latest
```

Login to a Docker image Repo

```bash
docker login
```