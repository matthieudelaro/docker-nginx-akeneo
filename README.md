# docker-nginx-akeneo
Akeneo PIM docker using Nginx

## Overview
This is a simple Akeneo PIM Docker image running PHP7, Nginx with PHP-FPM.

This borrows largely from two worlds [Akeneo PIM](https://docs.akeneo.com/1.6/developer_guide/installation/system_requirements/system_install_ubuntu_1604.html) and [nginx-php-fpm](https://github.com/ngineered/nginx-php-fpm). Pull requests are encouraged.

First checkout Akeneo PIM repository
```
git clone https://github.com/akeneo/pim-community-standard.git preferred-directory
```

## # Running Service Clusters
It's easy as:
1. Clone this repository
```BASH
git clone https://github.com/theiconic/docker-nginx-akeneo.git
cd docker-nginx-akeneo
```
2. Then provision the systems. Now, this could take up to half an hour depending on your connection. **TODO.** improve this. Possibly due to slow composer install
```
$ > cp .env.dist -uv .env
# Update the .env with your custom environment variables
$ > vim .env # Or whatever
$ > ./install.sh /path/to/preferred-directory
```
**Sample .env file**
```
# .env
PHP_MEM_LIMIT=512
WEBROOT=/var/www/pim
# Keep this empty
RUN_SCRIPTS=
PIM_WEB_PROCESS_USER=1000
# Set this as 1 to enable provisioning
PIM_PROVISION=1
MYSQL_ROOT_PASSWORD=l0r3m
WORKING_DIR=/home/rover/Workspace/Projects/akeneo.dev

# PIM configs
PIM_DB_HOST=mysql
PIM_DB_PORT=3306
PIM_DB_NAME=pim
PIM_DB_USER=p1m_user
PIM_DB_PASSWORD=
```

3. Subsequent runs cab be done just by running:
```
$ > ./run.sh
```

***That's it!*** Remember to add the IP to your `/etc/hosts` for convinience.


## # Running Image as stand-alone (Optional)
Create an image tag
```BASH
git clone https://github.com/theiconic/docker-nginx-akeneo.git
cd docker-nginx-akeneo
docker build -t theiconic/docker-nginx-akeneo . # Note the "."

```

Then create a simple run file to encapsulate all options:
```BASH
cat > run_pim.sh <<-EOF
SOURCE_FOLDER=$1
# Run the image
exec docker run --detach    \\
    --name akeneo_pim       \\
    -v "\${SOURCE_FOLDER}:/var/www/pim/"       \\
    -v "\${PWD}/scripts:/var/www/html/scripts" \\
    -v "\${PWD}/conf:/var/www/html/conf"       \\
    -e "PIM_WEB_PROCESS_USER=\$(id -u)"         \\
    -e "PHP_MEM_LIMIT=512"  \\
    -e "RUN_SCRIPTS=1"      \\
    -e "PIM_DB_HOST=~~~"    \\
    -e "PIM_DB_PORT=3306"   \\
    -e "PIM_DB_NAME=~~~~"   \\
    -e "PIM_DB_USER=~~~~"   \\
    -e "PIM_DB_PASSWORD=~~~~" \\
    -e "PIM_PROVISION=1"    \\
     theiconic/docker-nginx-akeneo
EOF

chmod u+x run_pim.sh

./run_pim.sh
```


#### Setting environment variable
***PIM_PROVISION=1*** is only necessasry for the first run. For subsequent run, this could be removed.
***PIM_WEB_PROCESS_USER*** is the ID for the repo user, or user running the `docker run`. Provisioning will fail if user id `alpine` is mapped to invalid ID. Defaults to 1000.
***RUN_SCRIPTS*** should always be set to 1. Hopefully in future iterations this will be on by default.



 Now you can sit back and run this:
 ```BASH
 run.sh /path/to/akeneo/pim/repo
 ```
That's it. Don't forget to **modify** your `run.sh` with relevant values.


### Known Issues
In case you get an exception:
```BASH
request.CRITICAL: Uncaught PHP Exception Twig_Error_Runtime: "An exception has been thrown during the rendering of a template ("Error during translations file generation for locale "en_US"")." at /var/www/pim/src/Oro/Bundle/TranslationBundle/Resources/views/requirejs.config.js.twig line 4 {"exception":"[object] (Twig_Error_Runtime(code: 0): An exception has been thrown during the rendering of a template (\"Error during translations file generation for locale \"en_US\"\"). at /var/www/pim/src/Oro/Bundle/TranslationBundle/Resources/views/requirejs.config.js.twig:4, RuntimeException(code: 0): Error during translations file generation for locale \"en_US\" at /var/www/pim/src/Pim/Bundle/EnrichBundle/Twig/TranslationsExtension.php:70)"} []
```

You can attempt to recompile the locale assests:
```
docker exec  <docker-ID> php /var/www/pim/app/console  --env=dev oro:translation:dump en_US
```
