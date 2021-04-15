# Local test

## Build images

1. Clone this repo
2. `cd symfony_demo`
3. `docker build -t demo-php:tag . --file=./docker/php/Dockerfile`
4. `docker build -t demo-nginx:tag . --file=./docker/nginx/Dockerfile`
5. Update images in docker-compose.yaml

## Test demo app

1. `docker-compose up`