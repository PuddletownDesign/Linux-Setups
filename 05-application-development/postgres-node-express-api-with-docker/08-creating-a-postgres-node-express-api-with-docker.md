# Setting up a postgresql container and working with it

In this guide we are going to explore working with a postgresql container

## First steps

Make sure that you have a VM set up to develop in and that docker-ce and docker-compose are installed.

Please reference these two guides to get everything ready.

1.  [Setting up a local development server with Virtual box and Debian and SSH access](01-local-development-server-with-virtualbox.md)
2.  [Installing Docker on a production server or local testing VM](12-Installing-Docker.md)

## Creating the project structure

We first need a top level directory to hold our project. I'll create it in a `~/Projects` folder in my main user folder, You can call it anything you like, I'll name mine `pern-api`.

_as you may have guess PERN stands for Postges, Express, React & Node_... and well we are making an API from it...

```bash
take pern-api

touch docker-compose.yml
```

Now we will need a couple folders within our project folder to work out of. The first will store the contents of the database. In the project folder I'll create a new folder called `db`

```bash
mkdir db
```

### Setting up the database

The first docker container we will create will be the postgres install. 

-   postgres is at: `postgres:13-alpine`
    and then one other folder for later to hold all of our node data.

```bash
mkdir api
```

All of our backend project files will go into the `api` folder. 

### Fleshing out the `docker-compose.yml` file

```yaml
version: "3.7"

services:
    postgres:
        container_name: postgres
        image: postgres:13-alpine
        network_mode: host
        environment:
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: postgres
            POSTGRES_DATABSE: postgresTest
        volumes:
            - ./db:/var/lib/postgresql/data
        ports:
            - "5432:5432"
        restart: unless-stopped
```

The first thing we do is tell docker-compose what version of syntax we will be using. Right now the most current one is `3.7`.

Then we will create a section for `services`. And then add `postgres` as the first service.

1.  `container_name`  - what we want the container to be identified as.
2.  `image` - what image we will be creating the container from.
3.  Some `enviornment`al variables to hold our database login.
4.  `volumes` - Directories that the container will share with us back to the host. Here i'm going to create a folder called `db` and have database persistence be stored in there. \_Please note that we don't even need to create the `db` directory itself... It will be created for us the first time that we run `docker-compose`. 

    > In volumes the first value is the local path on your machine.
    >
    > The second value (after the colon) is the path within the container to the data you would like.

5.  `ports` - we tell the container what ports to serve the service from. in this case it's only on `5432`.
6.  `restart` - How we want to handle it if our application or container crashes. If something goes wrong, I would like the container to restart, Unless it was intentionally stopped.

### Connecting the manual way

To connect to the postgres container and run some commands the basic gist is:

1.  #### connect to the postgress container
    ```bash
    docker exec -it postgres psql -U postgres
    ```
    which translates to:
    > `docker exec -it [container_name] psql -U [username]`
    >
    > -   `docker exec -it`  is the command to connect to the container
    > -   `pgsql` is the database admin tool located within the postgres container.
    > -   `-U` is the flag to connect to `pgsql` with a defined username. (remember in the `docker-compose` file we gave our user the name of `postgres`, which is honestly kind of some confusing bullshit since it's the same as the container name. Sorry about that.)
2.  #### Create a database called `test1`
    ```sql
    CREATE DATABASE test1;
    ```
3.  #### Connect to the new database (`test1`)

    ```bash
    /c test1
    ```

    `/c` - is the command to connect, followed by the name of the database that you want to connect to.

    You should see the prompt change again to `test1=#`

4.  #### Make a table in `test1`
    ```sql
    CREATE TABLE somebullshit (column1 VARCHAR(255));
    ```
    > Here we have create a table called `somebullshit` and created a column called column1 and given it a data type of `VARCHAR(255)`
5.  #### Insert some bullshit data into the `somebullshit` table we just created.

    ```sql
    INSERT INTO somebullshit (column1) values ('some bullshit data');
    ```

    > Insert into `column1` of the `somebullshit` table the value of `some bullshit data`

6.  #### Fetch the bullshit data you just inserted with a `SELECT` statement

    ```sql
    SELECT * FROM somebullshit;
    ```

    > select everything that's in the `somebullshit` table.

    and we should get the output:

    ```sql
    column1
    --------------------
    some bullshit data
    (1 row)
    ```

So there's the quick and not so easy way of working with the postgres container. Let's next look at a slightly better way of connecting and managing databases.

## Creating a container for `adminer`
