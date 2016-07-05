# Ansible Role: HAProxy
[![Build Status](https://travis-ci.org/robskie/ansible-haproxy.svg?branch=master)][1]
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-haproxy-blue.svg)][2]

This role installs and configures HAProxy versions `1.5` and `1.6`. Tested on
Debian and Ubuntu Linux distributions.

[1]: https://travis-ci.org/robskie/ansible-haproxy
[2]: https://galaxy.ansible.com/robskie/haproxy/

## Requirements

None.

## Role Variables

These are the available variables with their default values.

    haproxy_version: 1.6

The HAProxy version to install. Supported versions are `1.5` and `1.6`. This
will install the latest release for the given version.

    haproxy_config_file: haproxy-default.cfg

The local path where the configuration file resides. This file must contain the
default and global parameters. See `files/haproxy-default.cfg` for the default
configuration file.

    haproxy_clear_config: no

This role creates a configuration directory located at `/etc/haproxy/conf.d`.
This directory will contain the default/global configuration file, and a file
for each frontend and backend section. To create the final configuration file,
all the files inside this directory will be concatenated and stored in
`/etc/haproxy/haproxy.cfg`.

By default, old files are not deleted before the final configuration file is
assembled. This means that new frontend and backend settings will be stacked on
top of previous settings. To clear all previous configurations and disable
stacking, set `haproxy_clear_config` to `yes`.

    haproxy_separate_logs: no

If this is set to `yes`, then all log files will be saved in `/var/log/haproxy`
directory. In addition, logs with severity greater than `err` will be logged in
`error.log`, logs with severity `warning`, `notice`, or `debug` will be saved in
`haproxy.log`, and access logs will be separated by each backend and will be
stored in `[backend_name].log`.

    haproxy_frontends:
      - name: www-http
        bind: '*:80'
        default_backend: no-match
        acls:
          - name: host-example
            condition: hdr(host) -i example.com
        backends:
          - name: example.com
            condition: host-example
          - name: another.example.com
            domain: another.example.com
        extra_params:
          - 'capture request header User-Agent len 200'

This is an example that uses all the variables available in the frontend
section.

    haproxy_backends:
      - name: no-match
        extra_params:
          - 'errorfile 504 /etc/haproxy/errors/504.http'
      - name: example.com
        balance: roundrobin
        servers:
          - name: svr1
            address: 192.168.10.42:8080
            check: yes
            extra_params:
              - fall 5
          - name: svr2
            address: 192.168.10.43:8080
            check: yes
        extra_params:
          - 'option httpchk GET /check'
          - 'http-check expect status 200'
      - name: another.example.com
        servers:
          - name: svr1
            address: 192.168.10.44:8080

An example showing all available variables for the backend section.

## Tags

  - haproxy
  - install
  - configure
  - haproxy:install
  - haproxy:configure

## Dependencies

None.

## Example Playbook

    ---
    hosts: balancers
    become: yes
    roles:
      - role: robskie.haproxy
        haproxy_config_file: haproxy-custom.cfg
        haproxy_separate_logs: yes
        haproxy_frontends:
          - name: www-http
            bind: '*:80'
            backends:
              - name: my.domain.com
                domain: my.domain.com
        haproxy_backends:
          - name: my.domain.com
            balance: roundrobin
            servers:
              - name: svr1
                address: 192.168.10.42:8080
                check: yes
              - name: svr2
                address: 192.168.10.43:8080
                check: yes

## License

MIT
