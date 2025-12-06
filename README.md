# Overview
This repo contains development certificates used through mtnvencenzo docker compose stacks and applications.  These certs are for local development purposes only.

## Docker local
The main purpose of the [docker-local](./docker-local/README.md) certs was to have a common certificate for all my containers


## Azurite Local
This separate certificate for [azurite](./azurite-local/README.md) was added because azurite doesn't support multipart hostnames.  The single part host name could have been added to the main docker-local cert but I didn't want to have to keep adding to it for one-off situations.  