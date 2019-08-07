# Intro
Creates NGINX docker container with `host` networking. Default config shows NGINX welcome page on web site root.
Designed to act as a reverse proxy for intalled services. 
Additional servers/locations can be added by placing include configs into `{{ reverse_proxy_nginx_data_dir }}/config.d` folder.
To add location place
```
location <path> {
  <config>
}
```
as `{{ reverse_proxy_nginx_data_dir }}/conf.d/<location_name>.http.location` file to add http location
or
`{{ reverse_proxy_nginx_data_dir }}/conf.d/<location_name>.https.location`for https.

HTTPS server will try to get certificate from FreeIPA and if failed will generate self-signed certificate. Parameters for self-signed certificate can be configured through <global_*> vars defined, preferably, as a group variables.

## Default NGINX configuration
```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
      worker_connections  1024;
}


http {
  include       /etc/nginx/mime.types;
  default_type  application/octet-stream;

  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

  access_log  /var/log/nginx/access.log  main;

  sendfile        on;
  #tcp_nopush     on;

  keepalive_timeout  65;

  #gzip  on;

  include /etc/nginx/conf.d/*.conf;
}
```

## Variables
### Role
`reverse_proxy_nginx_container_name: reverse_proxy_nginx` Name for new container.
`reverse_proxy_nginx_label: "role=reverse_proxy"` Label for new container. Used to filter out containers when removing old container.
`reverse_proxy_nginx_data_dir: "{{ global_ansible_dir }}/{{ reverse_proxy_nginx_container_name }}"` Path where fiels will be placed.
`reverse_proxy_docker_remove_container: false`  will destroy old container when set to true.
`reverse_proxy_nginx_new_cert: false` will archive old certificate to `old_cert\_<epoch>.tar.gz` and get new certificates when set to true.
`reverse_proxy_nginx_new_config: false` will archive old config to `old_config_<epoch>.tar.gx` and roll out new config when set to true.
`reverse_proxy_nginx_http: false` Will include HTTP server configuration if set to true
`reverse_proxy_nginx_https: true` Will include HTTPS server configuration when set to true
`reverse_proxy_nginx_cert_ext: crt` Extension for the certificate. Mostly for compatibility as getting cert from IPA will produce `.chain` file that should be used as a certificate.
### Global
`global_ansible_dir` directory on the host for any ansible related files.
`global_certificate_validity` validity for self-signed certificate.
`global_certificate_country` country for self-signed certificate.
`global_certificate_state` state for self-signed certificate.
`global_certificate_company` company for self-signed certificate.
### Other
ipa-getcert.yml requires variables to query IPA server.
`pki_dir` directory to store certificates nad keys.
`ipa_host` FQDN of IPA server.
`ipa_admin_user` username for a user allowed to get new certificates.
`ipa_admin_pass` password for the user above.
`use_custom_root` set to true if root CA is different from IPA.
`cert_chain_checksum` checksum of the ca certificate in format `<hash_algorithm>:<sum>.


