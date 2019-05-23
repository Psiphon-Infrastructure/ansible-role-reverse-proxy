# ansible-role-reverse-proxy
Creates reverse proxy with HTTPS only. New self-signed cert and key will be generated and placed in `{{ nginx_data_dir }}/server.[crt,key]` Target URL can be specified by setting `nginx_proxy_pass` variable.

Defaults to IP whitelisting. All hosts in *prometheus-servers* will be allowed automatically. Any other hosts can be specified by setting `{{ allowedHosts }}` varibale. Ex. `allowedHosts: ['127.0.0.1','172.16.0.0/12','all']`. 

Complete list of variables can be found in *./defaults/main.yaml*. To pass custom values define them in parent playbook like so:
```
- include_role:
    name: ansible-role-reverse-proxy
  vars:
    nginx_proxy_pass: http://targeserver.bogus.local:80
    allowedHosts: ['all']
    nginx_container_port: 8443:443 
```
