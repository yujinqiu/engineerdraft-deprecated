---
title: "Deploying blog with docker and hugo"
description: "Deploying blog with docker and hugo."
date: "2014-05-04"
categories:
    - "docker"
    - "hugo"
---

#### Prelude

Recently I moved my jabber-server to DigitalOcean VPS. Run Prosody in docker was
so easy, that I decided create this blog. And of course deploy it with docker!

#### Content

At first we need container with templates and content for blog generation.
I used next dockerfile:

```
FROM debian:jessie

RUN apt-get update && apt-get install --no-install-recommends -y ca-certificates git-core
RUN git clone http://github.com/LK4D4/lk4d4.darth.io.git /src
VOLUME ["/src"]
WORKDIR /src
ENTRYPOINT ["git"]
CMD ["pull"]
```
There is no magic here: just clone repo to `/src` (it will be used below),
and update it on container start.

Build image:

```
docker build -t blog/content .
```

Create data container:

```
docker run --name blog_content blog/content
```

For updating content and templates from github we need just:

```
docker start blog_content
```

#### Hugo

[Hugo](http://hugo.spf13.com) -- very fast static site generator, written in Go (so
many cool things written in Go btw).

Idea is to run hugo in docker container so it reads contents from one directory
and writes generated blog to another.

Hugo dockerfile:

```
FROM crosbymichael/golang

RUN apt-get update && apt-get install --no-install-recommends -y bzr

RUN go get github.com/spf13/hugo

VOLUME ["/var/www/blog"]

ENTRYPOINT ["hugo"]
CMD ["-w", "-s", "/src", "-d", "/var/www/blog"]
```
So, here we `go get` hugo and use /src(remember this from content container?)
as source directory for it and `/var/www/blog` as destination.

Now build image and run container with hugo:

```
docker build -t blog/rendered .
docker run --name blog --volumes-from blog_content blog/rendered
```

Here the trick with
[`--volumes-from`](http://docs.docker.io/use/working_with_volumes/) -- we used
`/src` from `blog_content` container, and yeah, we're going to use
`/var/www/blog` from `blog` container.

#### Nginx

So, now we have container with templates and content `blog_content`, content
with ready to use blog `blog`, it's time to show this blog to the world.

I write simple config for nginx:

```
server {
    listen 80;
    server_name lk4d4.darth.io;
    location / {
        root /var/www/blog;
    }
}
```

Put it to sites-enabled directory, which used in this pretty dockerfile:

```
FROM dockerfile/nginx

ADD sites-enabled/ /etc/nginx/sites-enabled
```

Build image and run container with nginx:

```
docker build -t nginx .
docker run -p 80:80 -d --name=nginx --volumes-from=blog nginx
```

That's it, now blog is running on [lk4d4.darth.io](http://lk4d4.darth.io) and
you can read it :) I can update it just with `docker start blog_content`.

#### Conclusions

It's really fun to use docker. You don't need to install and remove
tons of crap on host machine, docker can handle it all for you.

![docker](https://www.docker.io/static/img/homepage-docker-logo.png)
