# 2020-10-31 PostgreSQL in Docker

## Aufgabe

Erstelle `docker-compose` Konfiguration, um lokal einen aktuellen PostgreSQL-Container zusammen mit einem `pgAdmin`-Container laufen zu lassen.
Dabei sollen die Inhalte (zunächst nur für PostgreSQL) in ein lokales Volume persistiert werden.
Kurzes googlen fand die folgenden Links als Ausagngspunkt:

- [https://www.pgadmin.org/download/pgadmin-4-container/](https://www.pgadmin.org/download/pgadmin-4-container/)
- [https://hub.docker.com/r/dpage/pgadmin4/](https://hub.docker.com/r/dpage/pgadmin4/)
- [https://linuxhint.com/postgresql_docker/](https://linuxhint.com/postgresql_docker/)


Die initiale Installation findet unter Linux (Ubuntu 20.04) statt, sollte aber, ggf. mit Anpassung der Volume-Pfade, auch unter Windows 10 laufen.
Sollte die Entscheidung fallen, die Persistierung unter dem aktuellen Verzeichnis vorzunehmen, sollten die Pfade betriebssystemunabhängig sein.
Allerdings halte ich dies für keine gute Idee, da ich es gewohnt bin, ausgecheckte Repositories als transient zu betrachten.

Bei den exponierten Ports ist darauf zu achten, dass diese nicht anderen, lokal schon genutzten, in die Quere kommen.

Auch ist zu beachten, dass `docker-compose` ggf. ein eigenes Netzwerk einrichtet, welches per Default gemäß des Verzeichnisnamens benamt ist.
Hat man also im aktuellen Verzeichnis mehrere `docker-compose` YAML Fils liegen, können diese sich in die Quere kommen.


## Installationsvoraussetzungen

- docker
- docker-compose
- ggf. curl


## Konfiguration

```bash
version: "3.7"

services:
  pg12:
    container_name: pg12
    image: postgres:12.2
    restart: always
    environment:
      POSTGRES_DB: talks
      POSTGRES_USER: jimmy
      POSTGRES_PASSWORD: banana
      PGDATA: /var/lib/postgresql/data
    volumes:
      - /var/postgres:/var/lib/postgresql/data
    ports:
      - "5432:5432"
 
  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:4.27
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: <email>
      PGADMIN_DEFAULT_PASSWORD: <pwd>
      PGADMIN_LISTEN_PORT: 80
    ports:
      - "8080:80"
    volumes:
      - /var/pgadmin:/var/lib/pgadmin
    links:
      - "pg12:pgsql-server"
```

Achtung: Es war notwendig, per `chmod 777 /var/pgadmin` die Zugriffsrechte anzupassen, da `pgAdmin` mit dem dockerinternem User `pgadmin` versucht, unter diesem Verzeichnis zu schreiben. Durch den Start über `docker-compose` wird es aber initial für `root:root` mit `755`angelegt, so dass hier nur `root` schreibberechtigt ist.
