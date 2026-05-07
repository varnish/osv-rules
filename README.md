# OSV Rules for Varnish Artifact Firewall

[![CI Pipeline](https://github.com/varnish/osv-rules/actions/workflows/ci.yaml/badge.svg)](https://github.com/varnish/osv-rules/actions/workflows/ci.yaml)
[![Last updated](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2Fvarnish%2Fosv-rules%2Fmain%2F.github%2Fbadges%2Flast-updated.json)](https://github.com/varnish/osv-rules/commits/main)

This repository contains auto-generated Varnish Artifact Firewall rulesets derived from [<img src="https://raw.githubusercontent.com/google/osv.dev/master/docs/images/osv_logo_light-full.svg" alt="OSV" height="20">](https://osv.dev) vulnerability data.

Rulesets are regenerated automatically every hour using [osv-rulegen](https://hub.docker.com/r/varnish/osv-rulegen) and committed directly to this repository.

## Rulesets

| Ecosystem | File | Rules |
|-----------|------|-------|
| npm       | [rulesets/npm/all.yaml](rulesets/npm/all.yaml)     | ![npm rules](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2Fvarnish%2Fosv-rules%2Fmain%2F.github%2Fbadges%2Fnpm.json) |
| pypi      | [rulesets/pypi/all.yaml](rulesets/pypi/all.yaml)   | ![pypi rules](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2Fvarnish%2Fosv-rules%2Fmain%2F.github%2Fbadges%2Fpypi.json) |
| nuget     | [rulesets/nuget/all.yaml](rulesets/nuget/all.yaml) | ![nuget rules](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2Fvarnish%2Fosv-rules%2Fmain%2F.github%2Fbadges%2Fnuget.json) |

## Usage

Add one or more `git` rulesets to the firewall configuration:

```yaml
firewall:
  rulesets:
    - git:
        name: pypi-osv-rules
        url: https://github.com/varnish/osv-rules
        ref: main
        sub_path: rulesets/pypi/all.yaml
        interval: 1h
    - git:
        name: npm-osv-rules
        url: https://github.com/varnish/osv-rules
        ref: main
        sub_path: rulesets/npm/all.yaml
        interval: 1h
    - git:
        name: nuget-osv-rules
        url: https://github.com/varnish/osv-rules
        ref: main
        sub_path: rulesets/nuget/all.yaml
        interval: 1h
```

## Generate your own rules

The rulesets in this repo are produced by [osv-rulegen](https://hub.docker.com/r/varnish/osv-rulegen). You can run it yourself to generate rules into your own repo, tweak the default action, or work from a local OSV dump.

```sh
docker run --rm varnish/osv-rulegen -ecosystem pypi > pypi-osv.yaml
```

### Flags

| Flag         | Default | Description |
|--------------|---------|-------------|
| `-ecosystem` | _(required)_ | Ecosystem to convert |
| `-action`    | `deny`  | Rule action when no severity score can be determined (`deny` or `hide`) |
| `-input`     |         | Path to a local `.zip` to convert instead of downloading |
| `-output`    |         | Download the zip and save it to this path, then exit |
| `-verbose`   | `false` | Log each deduplicated and ecosystem-skipped record |

### Examples

Use `hide` instead of `deny` when severity is unknown:

```sh
docker run --rm varnish/osv-rulegen -ecosystem npm -action hide > npm-osv.yaml
```

Convert a previously downloaded OSV zip:

```sh
docker run --rm -v "$PWD:/data" varnish/osv-rulegen -ecosystem npm -input /data/npm-all.zip > npm-osv.yaml
```

Each OSV vulnerability becomes one rule. Duplicates that share aliases (e.g. a CVE and its corresponding GHSA entry) are deduplicated. Withdrawn vulnerabilities are skipped.

## Severity mapping

Rules are assigned a numeric severity score (0–10) when one can be derived:

| Source | Logic |
|--------|-------|
| `MAL-` prefix | Always 10.0 (confirmed malicious package) |
| CVSS v4 vector | Base score from the vector (preferred over v3) |
| CVSS v3 vector | Base score from the vector |
| Qualitative label | `CRITICAL`→9.5, `HIGH`→8.0, `MODERATE`/`MEDIUM`→5.5, `LOW`→2.0, `NONE`→0.0 |

When no severity information is available, `deny` is used.
