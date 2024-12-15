# Docker Commands

## Commands

```bash
## Start docker service if not running
sudo dockerd &

## Start container if folder and run in background
docker compose up -d
docker compose up -d --force-recreate

## Clear docker unused data
docker system prune -af

## Create docker network
docker network create -d bridge mynetwork
docker network create -d bridge mynetwork --subnet=172.22.0.0/24 --internal
```



## Example docker compose file

