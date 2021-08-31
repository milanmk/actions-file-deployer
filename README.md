# FTP/SFTP file deployer

Fast and customizable deployment with parallel connections and proxy support. Deploy only changed files or do full sync.

This is a composite GitHub Action (Linux runner) for deploying repository content to remote server.

## Features

- Support for FTP and SFTP (SSH) protocols
- Use password or SSH private key for authentication of SFTP connection
- Delta file synchronization for faster deployment of only changed files since last revision
- Mirroring feature to copy entire file and folder structure of repository content
- Optimized for faster file transfers via parallel connections
- Connect to remote server via [SOCKS proxy](https://en.wikipedia.org/wiki/SOCKS) using [SSH tunneling](https://www.ssh.com/academy/ssh/tunneling) to bypass firewall / NAT / IP whitelist / VPC
- Uses [composite action](https://docs.github.com/en/actions/creating-actions/about-actions#types-of-actions) without Docker container for faster deployments and shorter run time
- Pass additional command arguments to SSH and FTP client for custom configurations and settings
- Step runs messages categorized nicely in log groups

![Workflow screenshot](./screenshot.png)

## Usage

```yml
- name: Checkout
  uses: actions/checkout@v2
  with:
    fetch-depth: 0
- name: Deploy
  uses: milanmk/actions-file-deployer@master
  with:
    remote-protocol: "sftp"
    remote-host: ${{ secrets.DEPLOY_PROD_HOST }}
    remote-user: ${{ secrets.DEPLOY_PROD_USER }}
    ssh-private-key: ${{ secrets.DEPLOY_PROD_PRIVATE_KEY }}
    remote-path: "/var/www/example.com"
```

Workflow example `.github/workflows/main.yml`.

```yml
name: Deploy Files

on:
  push:
    branches:
      - master
  # Enables manually triggering of Workflow with file synchronization option
  workflow_dispatch:
    inputs:
      sync:
        description: "File synchronization"
        required: true
        default: "delta"

jobs:
  master:
    name: master
    if: ${{ github.ref == 'refs/heads/master' }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - name: Deploy
        uses: milanmk/actions-file-deployer@master
        with:
          remote-protocol: "sftp"
          remote-host: ${{ secrets.DEPLOY_PROD_HOST }}
          remote-port: 22
          remote-user: ${{ secrets.DEPLOY_PROD_USER }}
          ssh-private-key: ${{ secrets.DEPLOY_PROD_PRIVATE_KEY }}
          remote-path: "/var/www/example.com"
```

## Inputs

| Name                  | Required             | Default | Description                                   |
|-----------------------|----------------------|---------|-----------------------------------------------|
| remote-protocol       | yes                  | sftp    | Remote file transfer protocol (ftp, sftp)     |
| remote-host           | yes                  |         | Remote host                                   |
| remote-port           | yes                  | 22      | Remote port                                   |
| remote-user           | yes                  |         | FTP/SSH username                              |
| remote-password       | no                   |         | FTP/SSH password                              |
| ssh-private-key       | no                   |         | SSH private key of user                       |
| proxy                 | yes                  | false   | Enable proxy for FTP connection (true, false) |
| proxy-host            | yes (if proxy: true) |         | Proxy host                                    |
| proxy-port            | yes (if proxy: true) | 22      | Proxy port                                    |
| proxy-forwarding-port | yes (if proxy: true) | 1080    | Proxy forwarding port                         |
| proxy-user            | yes (if proxy: true) |         | Proxy username                                |
| proxy-private-key     | yes (if proxy: true) |         | Proxy SSH private key of user                 |
| local-path            | yes                  | .       | Local path to repository                      |
| remote-path           | yes                  | .       | Remote path on host                           |
| sync                  | yes                  | delta   | File synchronization (delta, full)            |
| ssh-options           | no                   |         | Additional arguments for SSH client           |
| ftp-options           | no                   |         | Additional arguments for FTP client           |
| ftp-mirror-options    | no                   |         | Additional arguments for mirroring            |
| debug                 | no                   | false   | Enable debug information (true, false)        |

### Notes

- Character support for `remote-user` and `remote-password` is limited due to its usage in [.netrc file](https://www.gnu.org/software/inetutils/manual/html_node/The-_002enetrc-file.html)
  - It should not contain shell/URL special characters
- File synchronization options
  - `delta`: Transfer only changed files (upload and delete) since last revision
    - Only supported for `push`, `pull_request` and `workflow_dispatch` [events](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)
    - Requires `fetch-depth: 0` option in [checkout action](https://github.com/actions/checkout)
    - It is recommended to initially do a full synchronization and then switch to delta
  - `full`: Transfer all files (upload)
    - Does not delete files on remote host
    - Default glob exclude pattern is `.git*/`
- For `ftp-options` and `ftp-mirror-options` command arguments please refer to [LFTP manual](https://lftp.yar.ru/lftp-man.html)
- Enabling `debug` option will output useful context, inputs, configuration file contents and transfer logs to help debug each step
