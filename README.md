
TeamCity
=========

Ansible role for [Trickster by comcast](https://github.com/Comcast/trickster)

For possible settings and format look at [example config file in trickster repo](https://github.com/Comcast/trickster/blob/master/cmd/trickster/conf/example.conf)

## Compatibility
This role is compatible with any modern systemd-based distro.  

## Role Variables
| Variable name                           | Default value                                                      | Description                      |
|-----------------------------------------|--------------------------------------------------------------------|----------------------------------|
| trickster_user                        | `trickster-sv`      | system user                                 |
| trickster_version                     | `1.0.1`             | version of trickster                        |
| trickster_cli_flags                   | `{}`                | arguments passed to trickster               |
| trickster_main_instance_id            | `0`                 | instance id when running multiple instances |
| trickster_main_config_handler_path    | `/trickster/config` | config URL                                  |
| trickster_main_ping_handler_path      | `/trickster/ping`   | ping URL                                    |
| trickster_frontend_listen_address     | `0.0.0.0`           | address to listen on                        |
| trickster_frontend_listen_port        | `9099`              | port to listen on                           |
| trickster_frontend_tls_listen_address | `0.0.0.0`           | address to listen on (TLS)                  |
| trickster_frontend_tls_listen_port    | `0`                 | port to listen on (TLS)                     |
| trickster_frontend_connections_limit  | `0`                 | connection limit on tls/plain               |
| trickster_metrics_listen_port         | `8082`              | Port to listen on for metrics               |
| trickster_metrics_listen_address      | `127.0.0.1`         | address to listen on for metrics            |
| trickster_logging_log_level           | `info`              | Log level                                   |
| trickster_origins                     | `default dict`      | Dict with trickster origins                 |
| trickster_tracing                     | `default dict`      | Dict with trickster tracing                 |
| trickster_caching                     | `default dict`      | Dict with trickster caching                 |
## Origins
First thing you need to configure is origin which points to prometheus / etc TSDB.
Example variable:
```yaml
trickster_origins:
  - origin: default
    config:
      - is_default = true
      - origin_type = 'prometheus'
      - origin_url = 'http://localhost:9090'
      - cache_name = 'bbolt'
    health_check_headers:
      - Authorization = 'Basic SomeHash'
    tls:
      - full_chain_cert_path = '/path/to/your/cert.pem'
      - private_key_path = '/path/to/your/key.pem'
    paths:
      no_metrics_for_admin:
        - path = '/api/v1/admin/'
        - no_metrics = true
        - methods = [ '*' ] 
        - match_type = 'prefix'  
        - handler = 'localresponse' 
        - response_code = 401
        - response_body = 'No soup for you!'
```
As you can see, in this dict we configure prometheus upstream, set health_check_headers, configure TLS and lockdown /api/v1/admin. 

Possible subdicts for trickster_origins:

* config (main configuration block)
* health_check_headers
* tls
* paths
   - response_headers (NOT SUPPORTED ATM)
   - request_headers (NOT SUPPORTED ATM)
   - request_params (NOT SUPPORTED ATM)

### Caches
Next you will need to configure caches.  
Sample dict where we configure bbolt cache and set some optional parameters:
```yaml
trickster_caches:
  - cache: bbolt
    config:
      - cache_type = 'bbolt'
    index:
      - max_size_backoff_objects = 100
    bbolt:
      - filename = '/tmp/trickster.db'
      - bucket = 'trickster'
```
Subdicts for trickster_caches:

* config (main configuration block)
* redis
* bbolt
* badger
* index

### Tracing
Tracing can also be configured with simple dict:
```yaml
trickster_tracing:
  - tracing: default
    config:
      - implementation = 'opentelemetry'
      - exporter = 'noop'
      - collector = ''
      - sample_rate = 1
```