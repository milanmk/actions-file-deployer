# FTP/SFTP file deployer

Composite GitHub Action (Linux runner) for deploying repository content to remote host.

## Features

- Support for FTP and SFTP (SSH) protocols
- Use password or SSH private key for authentication of SFTP connection
- Defaults to delta file synchronization for faster deployment of only changed files since last commit
- Mirroring feature to copy entire file and folder structure of repository content
- Optimized for faster file transfers via parallel connections
- Allows connecting via [SOCKS proxy](https://en.wikipedia.org/wiki/SOCKS) to bypass firewall / NAT / IP whitelist / VPC for FTP connection (via [SSH tunneling](https://www.ssh.com/academy/ssh/tunneling))
- Uses [composite action](https://docs.github.com/en/actions/creating-actions/about-actions#types-of-actions) without Docker container for faster deployments and shorter run time
- Allows passing additional command arguments to SSH and FTP client for custom configurations
- Step runs messages categorized nicely in log groups

![Workflow screenshot](./screenshot.png)

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

- Character support for `remote-user` and `remote-password` is limited and should not contain shell special characters
- Delta file synchronization is only supported for `push`, `pull_request` and `workflow_dispatch` [events](https://docs.github.com/en/actions/reference/events-that-trigger-workflows) and requires `fetch-depth: 0` in [checkout action](https://github.com/actions/checkout)
- For `ftp-options` and `ftp-mirror-options` please refer to [LFTP manual](https://lftp.yar.ru/lftp-man.html)

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

on: [push, pull_request, workflow_dispatch]

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
