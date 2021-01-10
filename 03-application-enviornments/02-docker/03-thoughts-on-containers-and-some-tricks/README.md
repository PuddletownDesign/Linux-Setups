### external_links

Link to containers started outside this docker-compose.yml or even outside of Compose, especially for containers that provide shared or common services. external_links follow semantics similar to the legacy option links when specifying both the container name and the link alias (CONTAINER:ALIAS).

```docker
external_links:
-   redis_1
-   project_db_1:mysql
-   project_db_1:postgresql
```
