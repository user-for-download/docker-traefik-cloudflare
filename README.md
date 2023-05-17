>20.04.5 LTS (Focal Fossa); Docker version 20.10.22, build 3a2c30b; Mon Feb  6 15:06:44 2023; default user ubuntu (1000,1000)

The latest versions of dockers were used on the publication mark. Now they may not be compatible! Be careful.
## Features
- traefik
- socket-proxy
- whoami
- grafana
- portainer
- alertmanager
- authelia
- crowdsec
- crowdsec-dashboard

## Create new secrets
```bash
cd secrets/ && for i in *; do openssl rand -hex 16 > $i; done && cd ..
```
## Change config .env
```bash
cp .env.example .env
touch appdata/traefik/acme/acme.json
chmod 600 
```
> change `SITE.DOMAIN `, `USER`, `PUID` and `PGID`

## Change config acme
```bash
touch appdata/traefik/acme/acme.json
chmod 600 appdata/traefik/acme/acme.json
```

## Deploy first
```bash
docker network create -d bridge socket_proxy --subnet 172.16.91.0/24
docker network create -d bridge t2_proxy --subnet 172.16.90.0/24
```

## Deploy traefik and docker-socket-proxy
```bash
docker-compose up -d
```

# check logs
```bash
docker-compose logs
```
> cat logs/traefik/traefik.log

go https://traefic.SITE.DOMAIN

## Deploy services
```bash
docker-compose -p svc -f docker-compose-svc.yml up -d
docker-compose -p crds -f docker-compose-crwd.yml up -d
docker-compose -p mntr -f docker-compose-mntr.yml up -d
```

## If you need authorization, configure 
authella works only with HTTPS! Сreate a proxy in nginx-proxy-manager and get a certificate through Lets. 
```bash
cd appdata/authelia/
cp configuration.example.yml .configuration.yml 
```
and replase all `site.domain` in configuration.yml 

> Note: also change appdata/traefik/rules/middlewares.toml 
> address:  https://auth. `site.domain`

## Deploy
```bash
docker-compose -p auth -f docker-compose-auth.yml up -d
```
and uncomment 
> #traefik.http.routers.SERVICES.middlewares: middlewares-authelia@file
in docker-compose-*.yml files
