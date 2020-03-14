
TeamCity
=========

Ansible role for [Trickster by comcast](https://github.com/Comcast/trickster)

https://github.com/Comcast/trickster/blob/master/cmd/trickster/conf/example.conf

This role will install and configure trickster

## Compatibility
This role is compatible with any modern systemd-based distro.  

## Role Variables
| Variable name                           | Default value                                                      | Description                      |
|-----------------------------------------|--------------------------------------------------------------------|----------------------------------|
| trickster_user                    | `trickster-sv`                                                         | system user         |
| trickster_version                   | `1.0.1`                                                         | version of trickster       |

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
      - filename = 'trickster.db'
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