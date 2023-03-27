# rpx_nginx

<!--[![CI](https://github.com/xolyu/ansible-role-rpx_nginx/actions/workflows/ci.yml/badge.svg)](https://github.com/xolyu/ansible-role-rpx_nginx/actions/workflows/ci.yml)-->

Installs and configures a Reverse Proxy based on nginx with acme&middot;sh.  
Creates some snippets for sites in nginx and configures sites itself.

Certificates are automatically retrieved using the `xolyu.acmesh` role and included for the particular site.


## Requirements

None.


## Dependencies

`xolyu.acmesh` - For requesting certificates from Let's Encrypt.


## Role Variables

* **`rpx_role_execution`**  
  Controls the scope of role execution.  
  With `normal` everything is executed, first the installation and configuration of nginx is ensured, then the sites are configured. With `without_sites` the sites configuration is disabled, the first part is executed normally. With `only_sites` only the sites are configured, make sure that the installation and configuration of nginx has already taken place earlier.  
  Choices: `normal`, `only_sites`, `without_sites`  
  Default: `normal`

* **`nginx_repo`**  
  Defines the sources repository of nginx.  
  Default: *OS based, uses nginx Mainline repository*

* **`nginx_logdir`**  
  Defines the log directory of nginx. Has to be an absolute path.  
  Default: *OS based* (Debian based systems: `/var/log/nginx`)

* **`nginx_dir`**  
  Defines the config directory of nginx. Has to be an absolute path.  
  Default: *OS based* (Debian based systems: `/etc/nginx`)

* **`nginx_ssl_dir`**  
  Defines the SSL directory of nginx. A relative path is considered relative to `nginx_dir`. Can also be an absolute path.  
  Default: `ssl`

* **`nginx_snippets_dir`**  
  Defines the SSL directory of nginx. A relative path is considered relative to `nginx_dir`. Can also be an absolute path.  
  Default: `snippets`

* **`nginx_sites_dir`**  
  Defines the SSL directory of nginx. A relative path is considered relative to `nginx_dir`. Can also be an absolute path.  
  Default: `sites.d`

* **`rpx_cert_dir`**  
  Path where the certificates are stored by acme&middot;sh. If acme&middot;sh is pre-integrated as a role, the value in the variable `acmesh_cert_dir` is available, this is used if `rpx_cert_dir` is not explicitly defined.  
  Default: *uses value of variable `acmesh_cert_dir` otherwise fallback to `/etc/acme-certs`*


### nginx & SSL/TLS configuration

* **`nginx_worker_processes`**  
  Number of worker processes from nginx. See [worker_processes in nginx docs](http://nginx.org/en/docs/ngx_core_module.html#worker_processes).  
  Default: `1`

* **`nginx_ssl_protocols`**  
  SSL protocols nginx works with. See [ssl_protocols in nginx docs](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols).  
  Choices: `TLSv1`, `TLSv1.1`, `TLSv1.2`, `TLSv1.3`  
  Type: List of Strings  
  Default: `['TLSv1.2', 'TLSv1.3']`

* **`nginx_ssl_ecdh_curve`**  
  ECDH curves nginx uses. See [ssl_ecdh_curve in nginx docs](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_ecdh_curve).  
  Choices: *Depends on OpenSSL library*  
  Type: List of Strings  
  Default: `['X25519', 'secp521r1', 'secp384r1', 'prime256v1']`

* **`nginx_ssl_ciphers`**  
  Ciphers in nginx notation, separated by `:`.  
  Type: String (spaces will be eliminated)  
  Default: *see defaults/main.yml*

* **`nginx_ssl_session_cache_size`**  
  Default: `20m`

* **`nginx_ssl_session_timeout`**  
  Default: `12h`

* **`rpx_dhparam_source`**  
  Defines how the dhparam is created, whether it is generated on-demand by OpenSSL, which can take some time, or a 2048-bit dhparam is [downloaded from Mozilla](https://ssl-config.mozilla.org/ffdhe2048.txt).  
  Choices: `generate_by_openssl`, `download_from_mozilla`  
  Default: `generate_by_openssl`

* **`rpx_dhparam_numbits`**  
  Defines the bit length of the dhparam when generated by OpenSSL.  
  Choices: `2048`, `3072`, `4096` *(or every value possible by OpenSSL and nginx)*  
  Default: `4096`

* **`nginx_enable_hsts`**  
  Type: Boolean  
  Default: `yes`

* **`nginx_hsts_default_header`**  
  Default: `max-age=63072000`

* **`nginx_enable_ocsp_stapling`**  
  Type: Boolean  
  Default: `yes`

* **`nginx_ocsp_stapling_resolver`**  
  Default: `9.9.9.9`


### Error page configuration

* **`rpx_error_page_file`**  
  Name for the error page file with placeholder for the error number.  
  Default: `HTTP##errno##.html`

* **`rpx_error_page_file_placeholder`**  
  Placeholder of the error numbers in the file name of the error page.  
  Default: `##errno##`

* **`rpx_error_page_url_dir`**  
  URL path for internal redirection when error pages are delivered. See [error_page](http://nginx.org/en/docs/http/ngx_http_core_module.html#error_page) and [internal](http://nginx.org/en/docs/http/ngx_http_core_module.html#internal) in nginx docs.  
  Default: `/__rpx_error_page/`

* **`rpx_path_error_pages`**  
  Path where the error pages are stored.  
  Default: `/var/www/error_pages/`

* **`rpx_error_pages`**  
  Definition of error pages with error code, title and message.  
  Type: List of Dicts  
  Sub keys are: `code`, `title`, `message`  
  Default: *see vars/main.yml, variable _rpx_error_pages*  
  Pre-defined for codes: `[400, 401, 403, 404, 500, 501, 502, 503, 520, 521, 533]`

* **`rpx_error_numbers`**  
  List of all error numbers for which an error page is to be created.  
  Type: List of Integers  
  Default: *all error codes from `rpx_error_pages`*

* **`rpx_error_page_url`**  
  The generic complete URL of an error page.  
  Default: *value of `rpx_error_page_url_dir` concatenated with value of `rpx_error_page_file`*

* **`rpx_error_page_urls`**  
  Dict of all error numbers with the complete and concrete URL of the error page.  
  Type: Dict  

* **`rpx_error_page_show_message`**  
  Defines whether the error message should be integrated into the error page.  
  Type: Boolean  
  Default: `yes`


### Default Site

* **`rpx_default_site_name`**  
  Name of nginx default site.  
  Default: `000_default`

* **`rpx_default_site_function`**  
  Defines the function or behavior of the default site.  
  Choices: `error_page`, `static_content`  
  Default: `error_page`

* **`rpx_default_site_error_code`**  
  The error code if an error page is to be supplied.  
  Default: `404`

* **`rpx_default_site_content_path`**  
  The path in the file system if static content is to be delivered.  
  Default: *undefined*


### Sites of the reverse proxy

Sites are defined with **`rpx_sites`** as an List of Dicts.  
Default: `[]`  
Only sites that are defined are handled, i.e. sites that are not defined remain untouched, in particular they are not automatically deleted.

Dict configuration of a single site

* **`domain`** *(required)*  
  The main domain. Identifier for the configuration and primary domain (CN) in the certificate.

* **`additional_domains`**  
  Additional domains, e.g. for certificate requesting.  
  Type: List of Strings  
  Default: `[]`

* **`state`**  
  Indicates the desired site state.  
  Choices: `present`, `disabled`, `absent`  
  Default: `present`

  *Meaning of the state for different components:*

  |   state    |   site config   |  certificate   | log dir |
  | :--------- | :-------------: | :------------: | :-----: |
  | `present`  | created/updated | issued/enabled | created |
  | `disabled` | disabled        | disabled       | remains |
  | `absent`   | removed         | disabled       | removed |

* **`proxy_pass`** *(required)*  
  The proxy pass URL.

* **`access_log_for_proxy_pass`**  
  Defines whether the accesses should be logged on reverse proxy or not.  
  Type: Boolean  
  Default: `no`

* **`enable_hsts`**  
  Type: Boolean  
  Default: *value of `nginx_enable_hsts`*

* **`hsts_header`**  
  Default: *value of `nginx_hsts_default_header`*

* **`enable_ocsp_stapling`**  
  Type: Boolean  
  Default: *value of `nginx_enable_ocsp_stapling`*

* **`ocsp_stapling_resolver`**  
  Default: *value of `nginx_ocsp_stapling_resolver`*

* **`extra`**  
  Key-value pairs of nginx configuration parameters which are printed into the server block.  
  Type: Dict  
  Possible keys could be, for example: `client_max_body_size`, `proxy_buffering`, `proxy_read_timeout`  
  Default Key-values:
  ```yml
  extra:
    client_max_body_size: 128m
    proxy_buffering: off
    proxy_read_timeout: 3h
  ```

* **`extra_block`**  
  nginx configuration, which is copied 1:1 to the server block of the site configuration.  
  Type: String/Block  
  Default: *undefined*

* **`issue_cert`**  
  Defines whether a certificate should be requested for the site or not. The role `xolyu.acmesh` is used for certificate requesting.  
  Type: Boolean  
  Default: `yes`

Minimal `rpx_sites` configuration:
```yml
rpx_sites:
  - domain: service.example.org
    proxy_pass: http://127.0.0.1:8080
```

<!--
* **`VAR`**  
  DESC  
  Choices: `VAL`, `ANOTHER`  
  Default: `VAL`

* **`VAR`**  
  DESC  
  Type: List of Dicts  
  Default: `VAL`
-->


## Example Playbook

Minimal example for installing acme&middot;sh and nginx prepared as a reverse proxy.

```yml
- hosts: rpx_server
  roles:
    - role: xolyu.acmesh
      acmesh_ensure_requirements: yes
    - role: xolyu.rpx_nginx
```

Example with minimal site configuration

```yml
- hosts: rpx_server
  vars:
    rpx_sites:
      - domain: service.example.org
        proxy_pass: http://127.0.0.1:8080
  roles:
    - role: xolyu.acmesh
      acmesh_ensure_requirements: yes
    - role: xolyu.rpx_nginx
```

For testing purpose use test certificates from Let's Encrypt for nginx sites.  
&rArr; Define `acmesh_default_cert_test: yes`, a variable from role `xolyu.acmesh`.

```yml
- hosts: rpx_server
  vars:
    acmesh_default_cert_test: yes
    rpx_sites:
      - domain: service.example.org
        proxy_pass: http://127.0.0.1:8080
  roles:
    - role: xolyu.acmesh
      acmesh_ensure_requirements: yes
    - role: xolyu.rpx_nginx
```


## License

GNU General Public License v3.0


## Author Information

Xolyu.
