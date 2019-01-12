# Table of Contents


fix/rackuten-tracking-affiliate-conversions-with-coupon-code

- [Used technologies:](#used-technologies)
- [Docker Installation](#docker-installation)
- [Running unit tests](#running-unit-tests)
- [Running Drush, Composer](#running-drush-composer)
- [Running Drupal Console](#running-drupal-console)
- [Gulp](#gulp)
  - [Docker commands](#docker-commands)
  - [Gulp tasks](#gulp-tasks)
  - [Files structure](#files-structure)
    - [JS](#js)
- [Backups](#backups)
  - [Dumping database](#dumping-database)

# Used technologies:

- PHP
- Drupal 8
- Docker
- ES6
- Babel
- Gulp
- SASS
- ESlint

# Docker Installation

1.  Clone repo

```bash
git clone git@github.com:creativepropulsionlabs/built-story-marketing.git builtstorymarketing
```

2.  Go to the project root folder and run
    `docker-compose up`

```bash
cd builtstorymarketing && cp docker-compose.override.local.yml docker-compose.override.yml && docker-compose up -d --build
```

3.  Run `composer install`:

```bash
docker-compose exec php composer install
```

4.  Adjust your `traefik.yml` file.

```bash
cd .. && nano ./trafik.yml
```

    You will most like end up with something like this:

```yaml
version: '2'

services:
  traefik:
    image: traefik
    restart: unless-stopped
    command: -c /dev/null --web --docker --logLevel=DEBUG
    networks:
      - builtstorymarketing
    ports:
      - '80:80'
      - '8080:8080'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

networks:
  builtstorymarketing:
    external:
      name: builtstorymarketing_default

```

5.  Start traefik container.

```bash
docker-compose -f ./traefik.yml up -d
```

More info may be found at [Docker4Drupal docs](https://wodby.com/stacks/drupal/docs/local/multiple-projects/).

The website will be available at the following URL: [http://builtstorymarketing.docker.localhost](http://builtstorymarketing.docker.localhost/)

# Running unit tests

Execute a shell:

```bash
docker-compose exec php sh
```

`cd` to the root dir of the code (that's important!) and run PHPUnit:

```bash
cd ..
./vendor/bin/phpunit  web/modules/custom
```

# Running Drush, Composer

Just make sure to run your commands in the `php` container, like this:

```bash
docker-compose exec php composer install
docker-compose exec php drush updb
```

You _don't_ have to install or run anything locally.

# Running Drupal Console

Execute a shell:

```bash
docker-compose exec php sh
```

Run via `../vendor/bin/drupal`

```bash
../vendor/bin/drupal
```

# Gulp

By default, the gulp watcher started in builtstory theme.
To change the theme, you need to change ENV variable in gulp service, inside docker-compose.yml file
`GULP_THEME: cpl_base` -> `GULP_THEME: [theme]`
and run
```sh
docker-compose up -d --build
```

For get gulp log you could run
```sh
docker-compose -f gulp
```
and for reload gulp service if you needed
```sh
docker-compose restart gulp
```

## Docker commands
* ```docker-compose logs -f gulp``` - watch gulp service log.
* ```docker-compose restart gulp``` - restart gulp service.
* ```docker-compose exec gulp recompile``` - recompile all themes.


## Gulp tasks
* ```gulp test``` - task for check is gulp working.
* ```gulp compile``` - task for compile styles and scripts.
* ```gulp compile:styles``` - task for compile styles.
* ```gulp compile:scripts``` - task for compile scripts.
* ```gulp assets:fonts``` - task for create font from svg icons.
* ```gulp scripts:lint``` - task for lint js files.
* ```gulp watch``` - for start watching project directory to scripts or styles changes.
* ```gulp serve``` - for start project and browsersync.

Gulp automatically handle new/removed files, and you don't need to restart watch task manually.

## Files structure

### JS

For build JS files you need to create (if folder not exist) behavior folder inside theme js, folder, and place your files there.
You don't need to make one file with many behaviors, separate your behaviors to files, and gulp automatically build one file with all files scripts.

For example file with name test.js

```javascript
console.log('Hello World!', context);
```

will be compile to

```javascript
Drupal.behaviors[{themeName}Test] = {
    attach: function attach(context, settings) {
        console.log('Hello World!', context);
    }
};
```


# Backups

## Dumping database

You may want to dump your database at some point. To do so, run:

```bash
docker-compose exec php sh
../vendor/bin/drush sql-dump --structure-tables-list=cache,cache_*,watchdog |gzip > /tmp/dump.sql.gz
exit
docker cp `docker-compose ps -q php`:/tmp/dump.sql.gz /tmp/dump.sql.gz
```
