[WIP]

A sample to restore old WordPress with multi-container on Azure Web App
====

Incidentally, if you added some changes to restored WordPress, the changes affect only in the container instances not be saved. So please recreate your container images, or save them in another way.

References
----
- [Create a multi-container (preview) app in Web App for Containers | Microsoft Docs](https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-multi-container-app)
- [Manual Plugin Installation - Managing Plugins « WordPress Codex](https://codex.wordpress.org/Managing_Plugins#Manual_Plugin_Installation)
- [library/mysql - Docker Hub](https://hub.docker.com/_/mysql/)
    - Initializing a fresh instance

Prerequisite
----

- Docker environment
    - [Docker - Build, Ship, and Run Any App, Anywhere](https://www.docker.com/)
- Docker compose
    - [Docker Compose | Docker Documentation](https://docs.docker.com/compose/)
- Azure Account
    - [Create your Azure free account today | Microsoft Azure](https://azure.microsoft.com/en-us/free/)
- Azure CLI
    - [Azure CLI 2.0 | Microsoft Docs](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)

Roughly steps
----

1. Build containers
    - a wordpress container with plugins
    - a database container with initial data
2. Push these images to Azure Container Registry
3. Deploy them to Azure Web App

How it works
=====

Prepare this environment
----

```bash
# Prepare variables
cp .env.sample .env
sed -i -E 's/^(DB_PASSWORD)=.*$/\1=<database password>/' .env
sed -i -E 's/^(DB_NAME)=.*$/\1=<database name>/' .env

# Put your dump file to initialize your database
cp <your dum file> ./services/db/dump.sql

# Prepare a volume
docker volume create --name=azurewebapp-sample-multicontainers-db-data
```

Create an WordPress container that plugins pre-installed
----

Download plugin zip files to `./services/wordpress/plugins/` from [WordPress Plugins | WordPress.org](https://wordpress.org/plugins/) and unzip them. When build customized wordpress container image, these plugins are copied to default plugin directory `/usr/src/wordpress/wp-content/plugins/` in the container.

For example,
- [SyntaxHighlighter Evolved | WordPress.org](https://wordpress.org/plugins/syntaxhighlighter/)

Create a database(mysql) contaimer that is initialized by data
----

Put your dump file (`.sql`) as `dump.sql` in `./services/mysql/`. When build customized mysql container image, the dump file is placed under `/docker-entrypoint-initdb.d` in the container, then the container use the file for initialization. If you want to use `.sh` or `.sql.gz` as dump file, update `./services/mysql/Dockerfile`. For detail, see _Initializing a fresh instance_ in [library/mysql - Docker Hub](https://hub.docker.com/_/mysql/).

```bash
# Put your dump file
cp <your dump file> ./services/mysql/dump.sql

# Build a customized mysql contaimer image
docker-compose -f docker-compose.local.yml build mysql
```

If you want to setup the database with updated data, dump the updated data, put that and build again. 

```bash
docker exec -it custom-mysql mysqldump -hlocalhost --databases <database name> -p > dump_updated.sql
cp services/mysql/dump.sql services/mysql/dump.sql.backup
mv dump_updated.sql services/mysql/dump.sql
```


Check the containers
----

```bash
docker-compose.exe -f docker-compose.local.yml up -d
```

Open `http://localhost/wp-admin`. Then if the page is _white_ or something is wrong, some errors occurred. check your `siteurl` and `home` in `wp-options` table.


```bash
# Access to the database and change the siteurl and home url
source .env
docker exec -it custom-mysql mysql -hlocalhost -p${DB_PASSWORD} ${DB_NAME}

mysql> select * from wp_options where option_name = 'siteurl' or option_name = 'home';
+-----------+---------+-------------+----------------------------+----------+
| option_id | blog_id | option_name | option_value               | autoload |
+-----------+---------+-------------+----------------------------+----------+
|        37 |       0 | home        | http://<your previous url> | yes      |
|         1 |       0 | siteurl     | http://<your previous url> | yes      |
+-----------+---------+-------------+----------------------------+----------+
1 row in set (0.00 sec)

mysql> update wp_options set option_value = 'http://localhost' where option_name = 'siteurl';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> update wp_options set option_value = 'http://localhost' where option_name = 'home';
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from wp_options where option_name = 'siteurl' or option_name = 'home';
+-----------+---------+-------------+------------------+----------+
| option_id | blog_id | option_name | option_value     | autoload |
+-----------+---------+-------------+------------------+----------+
|        37 |       0 | home        | http://localhost | yes      |
|         1 |       0 | siteurl     | http://localhost | yes      |
+-----------+---------+-------------+------------------+----------+
2 rows in set (0.00 sec)
```

Then open `http://localhost/wp-admin`, you would see the page below if you need to update the database structure.

> Database Update Required
> 
> WordPress has been updated! Before we send you on your way, we have to update your database to the newest version.
The database update process may take a little while, so please be patient.

Press the _Update WordPress Database_ and _Continue_ buttons.

![](./docs/images/screen_database-update-required.png)
![](./docs/images/screen_database-update-complete.png)

Let's log in to your WordPress. After activating the necessary plugins, check the your site `http://localhost`. Can you see correct your site? Congrats!

![](./docs/images/screen_wordpress-login.png)


Push built container images to Azure Container Registry
----


Deploy these containers to Azure Web App
----


Notes
====

```bash
# Delete container instances
docker-compose down
```