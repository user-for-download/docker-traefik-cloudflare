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
nano .env
```
> change `SITE.DOMAIN `, `USER`, `PUID` and `PGID`

## Chmod acme.json
```bash
chmod 600 appdata/traefik/acme/acme.json
```

## Network create for traefik and socket_proxy 
```bash
docker network create -d bridge socket_proxy --subnet 172.16.91.0/24
docker network create -d bridge t2_proxy --subnet 172.16.90.0/24
```
## Delete dynamic.yaml
```bash
rm appdata\traefik\rules\dynamic.yaml
```

## Deploy traefik and socket-proxy
```bash
docker-compose up -d
```

Check logs
```bash
docker-compose logs
cat logs/traefik/traefik.log
```
> Wait to create a certificate and go https://traefic.SITE.DOMAIN

## Deploy services
```bash
docker-compose -p svc -f docker-compose-svc.yml up -d
docker-compose -p crds -f docker-compose-crwd.yml up -d
docker-compose -p mntr -f docker-compose-mntr.yml up -d
```

## If you need authorization, configure Authelia
Authella works only with HTTPS! Get a certificate through Let’s Encrypt. 
```bash
nano appdata/authelia/configuration.yml
```
and replase all `site.domain` in configuration.yml 

> Note: also change appdata/traefik/rules/middlewares.toml 

```yaml
[http.middlewares.middlewares-authelia.forwardAuth]
    address = "http://authelia:9091/api/verify?rd=https://auth.<SITE.DOMAIN>"
```
Check users Authelia
```bash
cat appdata/authelia/users_database.yml
```

> go https://auth.SITE.DOMAIN

## Deploy
```bash
docker-compose -p auth -f docker-compose-auth.yml up -d
```

Uncomment in docker-compose-*.yml files
```yaml
#traefik.http.routers.<SERVICES>.middlewares: middlewares-authelia@file
```

## If you changed then run the docker-compose down and up 
```bash
docker-compose -p <SERVICES> -f docker-compose-<FILE>.yml down
docker-compose -p <SERVICES> -f docker-compose-<FILE>.yml up -d
```
