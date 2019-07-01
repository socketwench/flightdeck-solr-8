![Flight Deck](https://raw.githubusercontent.com/ten7/flight-deck/master/flightdeck-logo.png)

# Flight Deck Solr

Flight Deck Solr is a minimalist Solr container for Drupal sites on Kubernetes and Docker. You can use it both for local development and production.

Features:
* ConfigMap-friendly YAML configuration
* Supports multiple cores
* Create [Search API Solr](https://www.drupal.org/project/search_api_solr) and [Apachesolr module](https://www.drupal.org/project/apachesolr) compatible cores without providing a schema
* Supports custom cores via ConfigMaps or volumes

## Tags and versions

There are several tags available for this container, each with different Solr and module support:

| Solr version | Tags | Search API | Apachesolr | Custom cores |
| ------------ | ---- | ---------- | ---------- | ------------ |
| 6.6.6 | 6.6, 6, latest | 8.x-3.x | *none* | yes |
| 5.5.5 | 5.5, 5 | 7.x-1.x | 7.x-1.x | yes |

### 4.x version

There is a 4.x version included in the repo for legacy reasons. It does not use YAML configuration, nor is it suitable for production. 

## Configuration

This container does not use environment variables for configuration. Instead, the `/config/flight-deck-solr.yml` file is used to handle all configuration.

```yaml
---
flightdeck_solr:
  port: 8983
  cores: []
```

Where:
* **flightdeck_solr.port** is the port on which to run Solr. Optional, defailts to `8983`.
* **flightdeck_solr.cores** is a list of cores to create.

### Defining cores

This container supports multiple cores by defining an item under the `flightdeck_solr.cores` item:

```yaml
---
flightdeck_solr:
  port: 8983
  cores:
    - name: "my_sapi_docker"
      type: "search_api_solr"
    - name: "my_apachesolr_core"
      type: "apachesolr"
    - name: "my_custom_core"
      type: "custom"
      confDir: "/solr-conf"
      dataDir: "/solr-data"
```

Each item has the following variables:

* **name** is the name of the solr core. Required.
* **type** is the type of the core to create. Can be `search_api_solr`, `apachesolr`, or `custom`. Required.
* **confDir** is the path inside the container at which the schema files are mounted. Required for `custom` cores.
* **dataDir** is the path inside the container at which the core data is to be stored. Optional, defaults to `/data/&lt;name_of_the_core&gt;`.

## Deployment on Kubernetes

Use the [`ten7.flightdeck_cluster`](https://galaxy.ansible.com/ten7/flightdeck_cluster) role on Ansible Galaxy to deploy Solr as a statefulset:

```yaml
flightdeck_cluster:
  namespace: "search"
  configMaps:
    - name: "solr_config"
      files:
        - name: "flight-deck-solr.yml"
          content: |
            flightdeck_solr:
              cores:
                - name: "my_sapi_docker"
                  type: "search_api_solr"
  solr:
    configMaps:
      - name: "solr_config"
        path: "/config"
```

## Debugging

If you need to get verbose output from the entrypoint, set `flightdeck_debug` to `true` or `yes` in the config file.

```yaml
---
flightdeck_debug: yes
```

If something has seriously gone wrong, can the container will not start due to a failure of the entrypoint, set the `FLIGHTDECK_SKIP_ENTRYPOINT` to `true` or `1`, then restart the container.

## Part of Flight Deck

This container is part of the [Flight Deck library](https://github.com/ten7/flight-deck) of containers for Drupal local development and production workloads on Docker, Swarm, and Kubernetes.

Flight Deck is used and supported by [TEN7](https://ten7.com/).

## License

Flight Deck is licensed under GPLv3. See [LICENSE](https://raw.githubusercontent.com/ten7/flight-deck/master/LICENSE) for the complete language.
