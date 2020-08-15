
Traefik is a open source reverse proxy / load balancer which is raising in popularity because of its ease to setup, integration with Docker and Let’s encrypt and much more features.

- Prerequisite
  - Basic Docker knowledge. You should know how to run a docker image with docker-compose.
  - Docker environment. We will deploy Traefik with Docker. You need a working Docker environment.

- Docker
  - First, to deploy Traefik we need a `docker-compose.yml` to configure Docker image and `traefik.toml` to configure Traefik.
  
 ```
 # docker-compose.yml
version: "3"

services:
  traefik:
    image: traefik:alpine
    ports:
      - 80:80
    volumes:
      - /docker/traefik:/etc/traefik
      - /var/run/docker.sock:/var/run/docker.sock
  whoami:
    image: containous/whoami
    labels:
      - traefik.enable=true
      - traefik.frontend.rule=Host:whoami.docker.localhost
 

 ```
 ```
 # traefik.toml
defaultEntryPoints = ["http"]

[entryPoints]
[entryPoints.http]
address = ":80"

[docker]
endpoint = "unix:///var/run/docker.sock"
domain = "docker.localhost"
exposedByDefault = false

```

In `docker-compose.yml`, we have two containers: traefik and whoami. Traefik has a official image, so we will use that for our container. traefik mount two volume. The first one is the configuration directory which should contains `traefik.toml`. The second one is Docker socket. Mounting it will allow Traefik to detect changes on Docker daemon.

- whoami uses `containous/whoami` which return the details of a request in HTML. Note that whoami has labels which is used for Traefik to detect per container configuration.
    - `traefik.enable=true` is needed if Traefik does not expose container by default which we disabled it in `traefik.toml`.
    - `traefik.frontend.rule=Host:whoami.docker.localhost` means only requests with a host header equals to `whoami.docker.localhost` is directed to this container.
    
- In `traefik.toml`, we create a entry point call http on port 80. This means Traefik will listen on port 80 for traffic. `[docker]` enables Docker integration.  
    -`endpoint = "unix:///var/run/docker.sock" `configure Docker socket location that we mount. (It is possible to connect a remote socket.
    - `domain = "docker.localhost"` is the default base domain use for docker containers. (Not related in this case because we have an label of the host rule on whoami .
    - `exposedByDefault = false` because we do not want Traefik to expose ALL containers by default.

Then, you can start the Docker containers with `docker-compose up -d`. Assume you have `whoami.docker.localhost`` mapped to your Docker machine, when you go to whoami.docker.localhost` on your broswer, you should see you request including IP address, headers, hostname.

# Let’s Encrypt

Traefik has build-in ACME integration which means it can request SSL certificate and resolve HTTPS request for you.
To get a valid SSL certificate, you must own a domain. You can skip this part if you do not.
Let’s Encrypt provides two kinds of challenges: HTTP-01 and DNS-01.

# HTTP-01 Challenge
We need to add a new https entry point and acme section. Also, the new 443 port (HTTPS) have to be exposed.

```
# docker-compose.yml
version: "3"

services:
  traefik:
    image: traefik:alpine
    ports:
      - 80:80
      - 443:443
    volumes:
      - /docker/traefik:/etc/traefik
      - /var/run/docker.sock:/var/run/docker.sock
  whoami:
    image: containous/whoami
    labels:
      - traefik.enable=true
      - traefik.frontend.rule=Host:whoami.docker.localhost


```

```
# traefik.toml
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.http]
  address = ":80"
  [entryPoints.https]
  address = ":443"
    [entryPoints.https.tls]

[acme]
entryPoint = "https"
storage = "/etc/traefik/acme/acme.json"
onHostRule = true
  [acme.httpChallenge]
  entryPoint = "http"

[docker]
endpoint = "unix:///var/run/docker.sock"
domain = "docker.localhost"
exposedByDefault = false


```
`storage` under `acme` represent where the SSL certificate is store because Let’s Encrypt has rate limit, you should store the certificate on volume. onHostRule means the certificate is generated base the host rule. In the example above, because we have `traefik.frontend.rule=Host:whoami.docker.localhost` , Traefik will try to request a SSL certificate for `whoami.docker.localhost`.


# DNS-01 Challenge

Your DNS provider must be supported by Traefik for it to work.

```
# traefik.toml
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.http]
  address = ":80"
  [entryPoints.https]
  address = ":443"
    [entryPoints.https.tls]

[acme]
entryPoint = "https"
storage = "/etc/traefik/acme/acme.json"
caServer = "https://acme-v02.api.letsencrypt.org/directory"
  [acme.dnsChallenge]
  provider = "cloudflare"
  delayBeforeCheck = 0
  [[acme.domains]]
  main = "*.docker.localhost"

[docker]
endpoint = "unix:///var/run/docker.sock"
domain = "docker.localhost"
exposedByDefault = false
```

DNS-01 challenge is needed for request wildcard certificate. Also, you need to change caServer to `https://acme-v02.api.letsencrypt.org/directory` because only v2 API supports wildcard certificate.

# File

File provider is more or less a traditional reverse proxy. You define the frontend rules and which backend to route to.

```
# traefik.toml
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.http]
  address = ":80"
  [entryPoints.https]
  address = ":443"
    [entryPoints.https.tls]

[acme]
entryPoint = "https"
storage = "/etc/traefik/acme/acme.json"
caServer = "https://acme-v02.api.letsencrypt.org/directory"
  [acme.dnsChallenge]
  provider = "cloudflare"
  delayBeforeCheck = 0
  [[acme.domains]]
  main = "*.docker.localhost"

[docker]
endpoint = "unix:///var/run/docker.sock"
domain = "docker.localhost"
exposedByDefault = false

[file]
watch = true

[backends]
  [backends.backend1]
    [backends.backend1.servers]
      [backends.backend1.servers.server0]
      url = "http://192.168.1.40"
      weight = 10

[frontends]
  [frontends.frontend1]
  backend = "backend1"
    [frontends.frontend1.routes]
      [frontends.frontend1.routes.route0]
      rule = "Host: frontend.docker.localhost"



```

We have added three new sections: file, frontends and backends. `watch = true` in file means any configuration changes will be hot reload. Note that you must mount the `traefik.toml` in a directory instead of mount it directly for it to work in Docker.

In backends, we create a new backend called backend1 which consist of one server server0 which point to `http://192.168.1.40`. `weight = 10` represent the weight in loading balancing. It does not matter if you only have one server.

In frontends, we create a new frontend called frontend1 which route the traffic if the request has a host header of `frontend.docker.localhost`. The traffic is routed to backend1 which is defined above.

By using file provider, we can now route the request to any server in our backends.

# Dashboard

Last but not least, Traefik provide a dashboard to monitor the traffic and show the frontends and backends in single place.

```
# traefik.toml
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.http]
  address = ":80"
  [entryPoints.https]
  address = ":443"
    [entryPoints.https.tls]
  [entryPoints.api]
  address = ":8080"

[api]
entryPoint = "api"

[acme]
entryPoint = "https"
storage = "/etc/traefik/acme/acme.json"
caServer = "https://acme-v02.api.letsencrypt.org/directory"
  [acme.dnsChallenge]
  provider = "cloudflare"
  delayBeforeCheck = 0
  [[acme.domains]]
  main = "*.docker.localhost"

[docker]
endpoint = "unix:///var/run/docker.sock"
domain = "docker.localhost"
exposedByDefault = false

[file]
watch = true

[backends]
  [backends.backend1]
    [backends.backend1.servers]
      [backends.backend1.servers.server0]
      url = "http://192.168.1.40"
      weight = 10

[frontends]
  [frontends.frontend1]
  backend = "backend1"
    [frontends.frontend1.routes]
      [frontends.frontend1.routes.route0]
      rule = "Host: frontend.docker.localhost"


```
We created a new entry point api on port 8080 which show our dashboard. After recreate your container, you should see the following:

![Docker page of Traefik Dashboard](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/docker-page-traefik.jpg)
![Health page of Traefik Dashboard](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/health-page-traefik.jpg)





    
    


