# Lando-Custom-Recipes-Tools-Services

[_TOC_]

## Services

### [Socketi](https://docs.soketi.app/):
```yaml
services:
  soketi:
    type: compose
    scanner: false
    services:
      image: "node:16"
      ports:
        - "6001:6001"
        - "9601:9601"
      command: /bin/sh /start
    build_as_root:
      - npm install -g @soketi/soketi
      - |
        echo "while [ ! -f /certs/cert.crt ]; do sleep 1; done
        while [ ! -f /certs/cert.key ]; do sleep 1; done
        soketi start --config /app/soketi.json
        " > /start
      - chmod a+x /start
```

### Unleash
```yaml
services:
  unleashdb:
    type: postgres
    portforward: true
    creds:
      database: unleash

  unleash:
    type: compose
    app_mount: false
    portforward: true
    ssl: true
    services:
      image: unleashorg/unleash-server
      depends_on:
        - unleashdb
      ports:
        - "4242:443"
      environment:
        UNLEASH_URL: "unleash.towardstruth.lndo.site"
        DATABASE_HOST: "unleashdb"
        DATABASE_NAME: "unleash"
        DATABASE_USERNAME: "postgres"
        DATABASE_PASSWORD: "user"
        DATABASE_PORT: 5432
        DATABASE_SSL: "false"
        # username: admin
        # password: unleash4all
      command: "node index.js"
      
  proxy:
    unleash:
    - unleash.towardstruth.lndo.site:4242
```

### [Meilisearch](https://www.meilisearch.com/)
```yaml
services:
  meilisearch:
    type: compose
    app_mount: false
    services:
      image: getmeili/meilisearch:latest
      command: tini -- /bin/sh -c /meilisearch
      ports:
        - "7700:7700"
      volumes:
        - meilisearch:/data.ms
    portforward: true
    volumes:
      meilisearch:
  socketi:
```

### Mini-io
```yaml
  s3:
    type: compose
    services:
      image: bitnami/minio:latest
      command:
        - /opt/bitnami/scripts/minio/entrypoint.sh
        - /opt/bitnami/scripts/minio/run.sh
      environment:
        MINIO_DEFAULT_BUCKETS: "default:public,test:public"
```

## Recipes

### CraftCMS
```yaml
name: mycraftsite
recipe: lamp
config:
  webroot: web
  php: "8.0"
  
services:
  node:
    type: node:16
    build:
      - "cd $LANDO_MOUNT && yarn"
  mail:
    type: mailhog
    
tooling:
  node:
    service: node
  npm:
    service: node
  yarn:
    service: node
  craft:
    service: appserver
    cmd: php craft
 ```

## Tooling
 
### [Database tools](https://github.com/tanc/lando-db-tools)

The intention of the plugin is to make working with Lando database containers and external tools a little easier. The plugin comes with three Lando 'tasks':

1. `lando dbport` - prints the external port and copies it to the clipboard
2. `lando workbench` - opens a connection using the MySQL Workbench GUI
3. `lando dbeaver` - opens a connection using the dbeaver GUI

Currently TablePlus and Sequelpro/sequelace are in a pull request and avaialable on our fork: https://github.com/phpsa/lando-db-tools

## Extra Services / Tips

### wkhtmltopdf

```yaml
services:
 appserver:
    build_as_root:
      - "wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.bullseye_amd64.deb && apt install -y ./wkhtmltox_0.12.6.1-2.bullseye_amd64.deb && rm ./wkhtmltox_0.12.6.1-2.bullseye_amd64.deb"
```

