# UzCloud CLI

[🇬🇧 English](README.md) | [🇷🇺 Русский](README.ru.md) | [🇺🇿 O'zbekcha](README.uz.md)

`uzc` is the command-line interface for [UzCloud](https://uzcloud.uz/). It manages virtual machines, volumes, networks, Kubernetes clusters, load balancers, DNS and billing through the same API (and the same API client) as the [Terraform provider](https://registry.terraform.io/providers/telecom-cloud-uzcloud/uzcloud/latest).

## Install

Download the archive for your platform from [Releases](https://github.com/Telecom-Cloud-UzCloud/uzcloud-cli/releases) and put `uzc` on your `PATH`.

macOS and Linux:

```bash
VERSION=1.0.3
OS=darwin      # or linux
ARCH=arm64     # or amd64
curl -LO https://github.com/Telecom-Cloud-UzCloud/uzcloud-cli/releases/download/v$VERSION/uzcloud_cli_${VERSION}_${OS}_${ARCH}.tar.gz
tar -xzf uzcloud_cli_${VERSION}_${OS}_${ARCH}.tar.gz uzc
sudo mv uzc /usr/local/bin/
uzc version
```

Windows: download `uzcloud_cli_<version>_windows_amd64.zip`, unpack it and add the folder with `uzc.exe` to `PATH`.

Debian/Ubuntu, RHEL/Fedora and Alpine packages (`.deb`, `.rpm`, `.apk`) are attached to every release too.

Verify the download against `uzcloud_cli_<version>_SHA256SUMS` from the same release:

```bash
shasum -a 256 -c uzcloud_cli_${VERSION}_SHA256SUMS --ignore-missing
```

On macOS the binary is not notarized yet. If Gatekeeper blocks it, run `xattr -d com.apple.quarantine /usr/local/bin/uzc`.

## Quick start

Issue an API token in the UzCloud portal (**Profile → API**), then:

```bash
uzc init                       # saves the token plus default project/region/provider/billing cycle
uzc region list
uzc plan list                  # VM plans in the default region
uzc template list
uzc ssh-key create laptop      # uploads ~/.ssh/id_ed25519.pub
uzc vm create web-1 --template ubuntu-22-04 --plan box2cm2 --ssh-key laptop --wait
uzc vm list
```

## Configuration

Settings are resolved in this order: **flag → environment variable → profile**.

| Setting        | Flag               | Environment          |
|----------------|--------------------|----------------------|
| API token      | `--token`          | `UZC_API_TOKEN`      |
| API endpoint   | `--endpoint`       | `UZC_API_ENDPOINT`   |
| Project        | `--project`        | `UZC_PROJECT`        |
| Region         | `--region`         | `UZC_REGION`         |
| Cloud provider | `--cloud-provider` | `UZC_CLOUD_PROVIDER` |
| Billing cycle  | `--billing-cycle`  | `UZC_BILLING_CYCLE`  |
| Profile        | `--profile`, `-p`  | `UZC_PROFILE`        |
| Config file    | `--config`         | `UZC_CONFIG`         |

The variable names match the Terraform provider's, so both tools can share one environment. Profiles live in `<user config dir>/uzcloud/config.yaml` (`~/.config/uzcloud` on Linux, `~/Library/Application Support/uzcloud` on macOS), written with `0600` permissions.

```bash
uzc config set region tashkent
uzc config view
uzc init --profile staging --endpoint https://staging.example/api
uzc config use staging
uzc -p prod vm list
```

## Output

Every command supports `-o table` (default), `-o json` and `-o yaml`; JSON/YAML print the full API object. Progress messages go to stderr, so stdout stays machine-readable:

```bash
uzc vm list -o json | jq -r '.[] | select(.state=="Running") | .slug'
uzc vm list -q | xargs uzc vm stop   # -q: only slugs
uzc vm get web-1 -o yaml
```

## Commands

| Group | Commands |
|---|---|
| `vm` | `list`, `get`, `create`, `start`, `stop`, `reboot`, `reset`, `delete`, `rename`, `resize`, `reinstall`, `set-password`, `set-user-data`, `add-network`, `tag`, `logs` |
| `snapshot` | `list`, `get`, `create`, `revert`, `delete`, `backup …` |
| `volume` | `list`, `get`, `create`, `attach`, `detach`, `delete`, `snapshot …`, `backup …` |
| `network` | `list`, `get`, `create`, `update`, `delete`, `egress …`, `router …`, `vpn-user …` |
| `vpc` | `list`, `get`, `create`, `rename`, `restart`, `delete`, `acl …` |
| `ip` | `list`, `get`, `create`, `delete`, `static-nat`, `firewall …`, `forward …`, `vpn …` |
| `dns` | `list`, `get`, `create`, `delete`, `add-record` |
| `k8s` | `list`, `get`, `create`, `start`, `stop`, `resize`, `delete` |
| `lb` | `list`, `get`, `create`, `add-rule`, `attach`, `delete` |
| `autoscale` | `list`, `get`, `enable`, `disable`, `resize`, `set-template`, `delete` |
| `ssh-key`, `affinity-group`, `iso` | `list`, `get`, `create`, `delete` |
| `project` | `list`, `get`, `create`, `update`, `use`, `delete` |
| `region`, `plan`, `template`, `cloud-provider` | catalog listings |
| `billing` | `balance`, `invoices`, `costs`, `cycles list` |
| `api` | raw authenticated request to any endpoint |

Run `uzc <command> --help` for flags and examples. Async operations accept `--wait` (with `--wait-timeout`); destructive ones ask for confirmation unless `--yes` is passed.

### Raw API access

Endpoints without a dedicated command are still reachable, with the same auth, retries and error handling:

```bash
uzc api GET /account/balance
uzc api GET /virtual-machines -q include=region,offering
uzc api POST /account/coupons -F coupon_code=FREE100
uzc api POST /autoscale -d @autoscale.json
```

### Shell completion

```bash
uzc completion zsh > "${fpath[1]}/_uzc"      # zsh
uzc completion bash > /etc/bash_completion.d/uzc
uzc completion fish > ~/.config/fish/completions/uzc.fish
```

Resource slugs complete too: `uzc vm stop <TAB>`.
