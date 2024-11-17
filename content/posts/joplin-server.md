+++
title = "Synchronizing Joplin notes between devices"
date = 2024-11-16
draft = false
+++

After doing a bit of research on several of the note taking apps available, I decided on [Joplin](https://joplinapp.org/) to be the next one I intend to use for several reasons.

> Joplin is an excellent open source note taking application with plenty of features. You can take notes, make to-do list and sync your notes across devices by linking it with cloud services. The synchronization is protected with end to end encryption.

<!--more-->

Having the ability to sync notes between devices is one of the main reasons I wanted to learn more about taking notes using Joplin, which can use more than one sync target, that being either a Joplin server, WebDAV server or Nextcloud instance.

I decided to go with Joplin server installed on a machine in my internal network. These are the steps required to get that up and running, assuming dedicated LXC container already exists and has [Docker](https://www.docker.com/) installed:

-   Create a `compose.yaml` file with the following default content:

<!--listend-->

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: always
    environment:
      - POSTGRES_PASSWORD=strong-password
      - POSTGRES_USER=joplin-user
      - POSTGRES_DB=joplindb
  app:
    image: joplin/server:latest
    container_name: joplin-server
    depends_on:
      - db
    ports:
      - "8080:8080"
    restart: always
    environment:
      - APP_PORT=8080
      - APP_BASE_URL=https://joplin.example.com
      - DB_CLIENT=pg
      - POSTGRES_PASSWORD=strong-password
      - POSTGRES_DATABASE=joplindb
      - POSTGRES_USER=joplin-user
      - POSTGRES_PORT=5432
      - POSTGRES_HOST=db
```

Modify the database credentials and base url to suit your own setup.

-   Start the container services:

<!--listend-->

```bash
docker compose up -d
```

-   Login to the web UI via browser on the url defined in `APP_BASE_URL`. The default username and password are `admin@example.com` and `admin` respectively.
    At this point you can and should change the default credentials to something more secure.


## Syncing between devices {#syncing-between-devices}

In order to sync notes between devices, go to `Tools > Options > Synchronisation` from within the Joplin app on each device and enter the appropriate Joplin server url and user credentials.

More details on the synchronization process can be found in the [official docs here](https://joplinapp.org/help/dev/spec/sync).
