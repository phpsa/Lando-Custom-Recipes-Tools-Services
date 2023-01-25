# Lando-Custom-Recipes-Tools-Services

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

## Recipes

## Tooling
