![Ansible Molecule Pipeline](https://github.com/inmotionhosting/ansible-role-nginx_proxy/actions/workflows/main.yml/badge.svg) [![GPL-3.0 License](https://img.shields.io/github/license/inmotionhosting/ansible-role-nginx_proxy.svg?color=blue)](https://github.com/inmotionhosting/ansible-role-nginx_proxy/blob/master/LICENSE) [![GitHub stars](https://img.shields.io/github/stars/inmotionhosting/ansible-role-nginx_proxy.svg)](https://github.com/inmotionhosting/ansible-role-nginx_proxy/stargazers)

![InMotion Hosting Ultrastack](https://www.inmotionhosting.com/wp-content/uploads/2024/01/ultrastack-logo-black-vertical.png)

# Ansible Role: Nginx Proxy

Modular Ansible Role for deploying and configuring Nginx as a reverse-proxy

## Requirements
This Ansible role supports the two latest stable releases of specific
server-focused Linux distributions and aims to follow their deprecation
policies. Additionally we will focus on supporting the latest two stable
releases of each, which at the time of writing are as follows:

* CentOS 7.x
* Debian 11 or later
* Ubuntu 20.04 LTS or later
* AlmaLinux 8.x or later
* RockyLinux 8.x or later

## Dependencies

* community.general
* ansible.posix

## Role Variables

Available variables are listed below with their default values (you can also see `defaults/main.yml`)

| Variable | Description |
| -------- | ----------- |
| nginx_daemon | Default: `nginx`
| nginx_group | Default: `nobody`
| nginx_name | Default: `nginx`
| nginx_user | Default: `nginx`
| nginx_packages | Default: `[nginx]`
| nginx_pid | Default: `/var/run/nginx.pid`
| nginx_mime_includes | Default: `/etc/nginx/mime.types`
| nginx_module_includes | Default: `/usr/share/nginx/modules/*.conf`
| nginx_proxy_includes | Default: `/etc/nginx/proxy.conf`
| nginx_site_includes | Default: `/etc/nginx/conf.d/*.conf`
| nginx_trusted_proxies_includes | Default: `/etc/nginx/trusted_proxies.conf`

### Core
| Variable | Description |
| -------- | ----------- |
| nginx_client_body_buffer_size | Default `1m`
| nginx_client_header_buffer_size | Default `2k`
| nginx_client_max_body_size | Default `512m`

### Caching
| Variable | Description |
| -------- | ----------- |
| nginx_cache_convert_head: | Default: `true`
| nginx_cache_honor_cc: | Default: `false`
| nginx_cache_honor_cookies: | Default: `true`
| nginx_cache_honor_expires: | Default: `false`
| nginx_cache_inactive | Default: `1h`
| nginx_cache_name | Default: `sitecache`
| nginx_cache_time_404 | Default: `10`
| nginx_cache_time_default | Default: `5`
| nginx_etag | Default: `true`
| nginx_open_file_cache_errors | Default: `false`
| nginx_open_file_cache_inactive | Default: `8m`
| nginx_open_file_cache_max | Default: `16536`
| nginx_open_file_cache_min_uses | Default: `1`
| nginx_open_file_cache_valid | Default: `5m`
| nginx_ssi | Default: `false`

### Compression
| Variable | Description |
| -------- | ----------- |
| nginx_gzip_enabled | Default: `true`
| nginx_gzip_comp_level | Default: `9`
| nginx_gzip_min_length | Default: `256`

### Connection
| Variable | Description |
| -------- | ----------- |
| nginx_hsts_enable | Default: `false`
| nginx_http2_enable | Default: `true`
| nginx_keepalive_requests | Default: `100`
| nginx_keepalive_timeout | Default: `30`
| nginx_multi_accept | Default: `true`
| nginx_reset_timedout_connection | Default: `true`
| nginx_sendfile | Default: `true`
| nginx_tcp_nodelay | Default: `false`
| nginx_tcp_nopush | Default: `true`

### Logging
| Variable | Description |
| -------- | ----------- |
| nginx_access_log | Default: `/var/log/nginx/access.log`
| nginx_error_log | Default: `/var/log/nginx/error.log`
| nginx_log_format_main | Default: `$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"`

### Proxy
| Variable | Description |
| -------- | ----------- |
| nginx_proxy_buffers | Default: `[4, 32k]`
| nginx_proxy_buffer_size | Default: `32k`
| nginx_proxy_busy_buffers_size | Default: `64k`
| nginx_proxy_cache_key | Default: `"$scheme$request_method$host$request_uri"`
| nginx_proxy_connect_timeout | Default: `90`
| nginx_proxy_hide_header | Default: `["Upgrade"]`
| nginx_proxy_read_timeout | Default: `90`
| nginx_proxy_redirect | Default: `false`
| nginx_proxy_send_timeout | Default: `90`

### Ratelimit
| Variable | Description |
| -------- | ----------- |
| nginx_ratelimit | Default: `8`
| nginx_ratelimit_burst | Default: `8`
| nginx_ratelimit_nodelay | Default: `true`
| nginx_ratelimit_zone | Default: `rlzone`
| nginx_ratelimit_paths | Default: `[".*login\\.php", ".*xmlrpc\\.php", ".*wp-cron\\.php"]`

### SSL
| Variable | Description |
| -------- | ----------- |
| nginx_ssl_enable | Default: `true`
| nginx_ssl_ciphers | Default: `["EECDH+AESGCM", "EDH+AESGCM", "AES256+EECDH", "AES256+EDH", "ECDHE-RSA-AES128-GCM-SHA256", "ECDHE-ECDSA-AES128-GCM-SHA256"]`
| nginx_ssl_protocols | Default: `["TLSv1.2", "TLSv1.3"]`
| nginx_ssl_session_cache | Default: `"shared:SSL:32m"`

### Static
| Variable | Description |
| -------- | ----------- |
| nginx_static_content_accel | Default: `true`
| nginx_static_content_paths | Default: `[]`

### Workers
| Variable | Description |
| -------- | ----------- |
| nginx_worker_connections | Default: `4096`
| nginx_worker_processes | Default: `auto`
| nginx_worker_rlimit_nofile | Default: `8192`
| nginx_worker_shutdown_timeout | Default: `4`

### SELinux
| Variable | Description |
| -------- | ----------- |
| selinux_enabled | Default: `false`

## Example Playbook

```yaml
- hosts: www
  roles:
     - role: inmotionhosting.nginx_proxy
```

## License

GPLv3

## Author Information

[InMotion Hosting](https://inmotionhosting.com)
