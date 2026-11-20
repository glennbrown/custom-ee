# custom-ee

A basic Ansible execution environment built on `ubi10/ubi-minimal` with
[ansible-builder](https://ansible.readthedocs.io/projects/builder/). Images are built by GitHub
Actions and published to the GitHub Container Registry.

## Contents

Everything is defined in a single file, `execution-environment.yml`, which is the ansible-builder
definition (base image, ansible-core, runner, collections, Python packages, system packages, and
extra build steps).

### Ansible collections

`ansible.posix`, `ansible.mariadb`, `ansible.mysql`, `ansible.utils`, `ansible.windows`,
`community.crypto`, `community.dns`, `community.docker`, `community.library_inventory_filtering_v1`,
`community.general`, `community.hashi_vault`, `community.libvirt`, `community.postgresql`,
`community.proxmox`, `containers.podman`

### Python packages

`ansible-core` (>=2.19,<2.22), `ansible-runner`, `ansible-pylibssh`, `ansible-lint`, `boto3`,
`cryptography`, `dnspython`, `docker`, `hvac`, `fqdn`, `jxmlease`, `jmespath`, `paramiko`,
`passlib`, `petname`, `pexpect`, `proxmoxer`, `pytz`, `pywinrm`, `netaddr`, `requests`, `toml`,
`xkcdpass`

The Python interpreter is 3.14.

### System packages

`apr-util`, `bsdtar`, `bind-utils`, `bzip2`, `git-core`, `less`, `jq`, `openssh-clients`, `rsync`,
`tar`, `gzip`, `shadow-utils`, `unzip`, `zip`

## Bundled CLIs

The image also includes `sshpass` (built from source), the 1Password CLI (`op`), and the Bitwarden
CLI (`bw`). The Bitwarden CLI is downloaded from the latest `cli-v*` release of `bitwarden/clients`
at build time, so it is not version pinned. The download is verified against the SHA256 digest that
GitHub publishes for the release asset.

## Build locally

```sh
pip install ansible-builder
ansible-builder build --tag ghcr.io/<owner>/custom-ee:dev --container-runtime podman
```

## Run

```sh
podman run --rm ghcr.io/<owner>/custom-ee:latest ansible --version
```

Use it with ansible-navigator:

```sh
ansible-navigator run site.yml --eei ghcr.io/<owner>/custom-ee:latest --mode stdout
```

## CI

The workflow in `.github/workflows/build.yml` runs on pushes to every branch, all pull requests, a
weekly schedule (Mondays at 06:00 UTC, to pick up base image updates), and manual dispatch. Version
tags do not trigger builds. It builds with Buildah (`redhat-actions/buildah-build`) and pushes with
`redhat-actions/push-to-registry`. Pull requests build the image without pushing.

Other events push multi-arch (amd64 and arm64) images to `ghcr.io/<owner>/<repo>`:

| Built from | Tags |
| --- | --- |
| `main` | `latest`, short SHA |
| any other branch | branch name (with `/` and other unsafe characters replaced by `-`), short SHA |

Scheduled runs only execute on the default branch, so only `latest` is refreshed weekly. Branch
images are rebuilt when the branch is pushed to.

After the first push, the package is private by default. Change its visibility under the package
settings on GitHub if you want it to be public.
