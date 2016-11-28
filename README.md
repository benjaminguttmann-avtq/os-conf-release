## BOSH Operating System Configuration Release

BOSH Release that enables configuration of the Operating System

- customize login banner text (job: `login_banner`)
- change TCP keepalive kernel args (job: `tcp_keepalive`)
- configure resolv.conf search domain (job: `search_domain`)
- add UNIX users to VM (job: `user_add`)
- customize `limits.conf` (job: `limits`)

## Upload the Release to the BOSH Director

```
bosh upload-release https://bosh.io/d/github.com/cloudfoundry/os-conf-release
```

## BOSH Manifests / Runtime Config

Include the release:

```yaml
releases:
  name: os-conf
  version: latest
```

In this example, we use BOSH's [Runtime Config](https://bosh.io/docs/runtime-config.html)
to customize login banner and create two users: first, an _operator_ user with an encrypted password; second, a _backup_ user with an ssh-key:

```yaml
addons:
  - name: os-configuration
    jobs:
    - name: login_banner
      release: os-conf
      properties:
        login_banner:
          text: |
            Authorized Use Only.
            Unauthorized use will be prosecuted to the fullest extent of the law.
    - name: user_add
      release: os-conf
      properties:
        users:
        - name: backup
          public_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDbss5XtLYRYDeV8AmouVYOHmYPxPsN4F59fZnY4kJnimM3sk5TbP0ow19GMDppQOPzAQ1TcYH4sYhpnxwq5f32XYtw12rFnO8BatHISWIdjoEjHfdA1qLIMGouWZPbGIQ1qURbfJdR9e2shS7U/WSXD+AJ9Zy0ZKTsIvlukWSX8Nsxvfn7VaAFvhgI3YPmhjV3TCEVMDsWGbBXlMq+qiJt22JEOw+3dnrvfGzRUULGznO/8y4NvVQsQc5KGnJkeQWkmlOIrhUGYwd/hMn6zQEIxkR4elmwp+pjyLR0qYLUFjpMn2GJMG7lvTzF8SzQLhzTVrjW1E3nve2eCuJ5bB6/"
```

In this example, we configure our BOSH deployment manifest to configure the DNS
search domain to `pivotal.io` and the TCP keepalive kernel settings:

```yaml
instance_groups:
- name: network-infrastructure
  jobs:
  - name: tcp_keepalive
    release: os-conf
    properties:
      tcp_keepalive:
        time:     120
        interval:  30
        probes:     8
  - name: search_domain
    release: os-conf
    properties:
      search_domain: pivotal.io
```

##  Examples

See `manifests/` for examples.
