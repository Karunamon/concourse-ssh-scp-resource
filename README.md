![](https://img.shields.io/github/license/Karunamon/concourse-ssh-scp-resource) ![](https://img.shields.io/badge/resource%20url-ghcr.io%2Fkarunamon%2Fssh--scp--resource-blue?logo=Concourse)
# ssh-scp-resource
A Concourse resource to run commands on hosts or copy files to them using SSH or SCP.

## Resource Configuration
All items are required, and go under the `source` key:

* `user`: SSH username
* `host`: Host to log into
* `private_key`:  Private key for `user`

## Behavior
* `check`: Not implemented
* `get`: Not implemented
* `put`: Run a command or copy a file to the configured `user@host`

To copy files, add the `files` key to `params`, and under it, give the source as a key and destination as value for each file or directory. It is recommended to quote both the source and destination to avoid any YAML parsing surprises. The source directory should be an `output` name from a previous build step.

```yaml
 params:
   files:
     "outfiles/index.html": "/var/www/public_html/index.html"
     "outfiles/page.html": "/var/www/public_html/page.html"
 ```
 
To run commands, add the `commands` key to `params`, and under that, give the commands to run as list items (prefixed with a `-`). Again, quotes are recommended to avoid YAML surprises. Note that all commands run in a `&&` chain from top to bottom, so failure or a nonzero exit of any command will result in all later commands on this step not executing.

```yaml
params:
  commands:
    - "ls /var/www/public_html"
    - "free -m"
    - "/bin/false"
    - "/bin/true"  # Will not run
```

You may copy files and run commands as part of the same step. **Note that file copies will always be run first**.

```yaml
params:
  files:
     "outfiles/main.py": "/var/www/public_html/main.py"
  commands:
    - 'ls /var/www/public_html`
    - `md5sum /var/www/public_html/*`
    - `systemctl restart nginx`
```

## Installation
Add a new resource type to your pipeline:
```yaml
resource_types:
- name: ssh-scp
  type: registry-image
  source: { repository: ghcr.io/karunamon/ssh-scp-resource }
```

Then, define a resource targeting the system you want to run commands on or copy files to:
```yaml
resources:
- name: website-html-scp
  type: ssh-scp
  icon: web
  source:
    user: someuser
    host: mywebserver.com
    private_key: |
      -----BEGIN OPENSSH PRIVATE KEY-----
      ...
      -----END OPENSSH PRIVATE KEY-----
```

## Thanks

[Dan Spencer (@danrspencer)](https://github.com/danrspencer) for writing this. All I've done is minor bug fixing and documentation.
