# Custom Bluefin DX Image

My custom bootc image

This repository builds a custom bootc image on GitHub Actions.

| Setting | Value |
|---------|-------|
| Repository | `slewfoot66/my-bluefin-dx` |
| Base Image | `Bluefin DX` |
| Base Image URI | `ghcr.io/ublue-os/bluefin-dx:stable` |
| Published Image | `ghcr.io/slewfoot66/my-bluefin-dx:latest` |
| Build Method | `Containerfile` |

## Managed By Atomic Image Builder

This repo is managed by `atomic-image-builder`. `.atomic-image-builder.json` is the saved settings file and source of truth for future updates.

If you hand-edit this repo after `atomic-image-builder` creates or manages it, stop using `atomic-image-builder` for this repo.

Later tool-driven updates rewrite managed files and can overwrite manual changes, especially `README.md` and `build_files/build.sh`.

## Requested Packages

These are the package names requested by this repo's generated build script.
Selected packages are what this repo will attempt to add, even if some are already present in the chosen base image.

- None selected yet.

## COPR Repositories

- None.

## Enabled Services

- `sshd.service`

## Removed Base Packages

- None.

## Before The First Switch

The first build publishes

    ghcr.io/slewfoot66/my-bluefin-dx:latest

as a **private** package. That is GitHub's default for a newly published package,
and it does not change with the repository's own visibility, so `bootc switch` on a
machine with no registry credentials cannot read it yet.

Make it readable once, from the package's own page:

1. Open <https://github.com/slewfoot66/my-bluefin-dx/pkgs/container/my-bluefin-dx>
2. **Package settings** -> **Change visibility** -> **Public**

Keeping it private is fine too, but then the machine needs GHCR pull credentials for
root before the switch below will work. See <https://bootc.dev/bootc/registries-and-offline.html>.

## Trusting The Signing Key

This repository signs every published image. Before switching, clone this repo
and install its committed public key:

```bash
sudo install -Dm0644 cosign.pub /etc/pki/containers/slewfoot66-my-bluefin-dx.pub
```

The filename carries the owner because `/etc/pki/containers` is shared by every
image this machine trusts.

Then add this repository to `/etc/containers/policy.json`. This command adds one
entry, keeps every other entry, creates `transports` -> `docker` if the file has
none -- the stock Fedora policy does not -- and always writes valid JSON:

```bash
sudo python3 - <<'EOF'
import json, pathlib

path = pathlib.Path("/etc/containers/policy.json")
policy = json.loads(path.read_text())
docker = policy.setdefault("transports", {}).setdefault("docker", {})
docker["ghcr.io/slewfoot66/my-bluefin-dx"] = [
    {
        "type": "sigstoreSigned",
        "keyPath": "/etc/pki/containers/slewfoot66-my-bluefin-dx.pub",
        "signedIdentity": {"type": "matchRepository"},
    }
]
path.write_text(json.dumps(policy, indent=4) + "\n")
EOF
```

It reads and parses the file before writing anything, so an already-broken
`policy.json` stops it rather than being overwritten. To edit the file by hand
instead, add this member to the `docker` object, comma-separated from whatever
is already there:

```json
"ghcr.io/slewfoot66/my-bluefin-dx": [
  {
    "type": "sigstoreSigned",
    "keyPath": "/etc/pki/containers/slewfoot66-my-bluefin-dx.pub",
    "signedIdentity": { "type": "matchRepository" }
  }
]
```

Cosign stores these signatures as registry attachments, whose lookup is disabled
by default. As root, create the repository-specific discovery file
`/etc/containers/registries.d/slewfoot66-my-bluefin-dx.yaml` with:

```yaml
docker:
  ghcr.io/slewfoot66/my-bluefin-dx:
    use-sigstore-attachments: true
```

Reinstall `cosign.pub` after rotating the repository's signing key.

## Using The Image

After the first successful GitHub Actions build finishes, switch to it with:

```bash
sudo bootc switch --enforce-container-sigpolicy ghcr.io/slewfoot66/my-bluefin-dx:latest
systemctl reboot
```

`--enforce-container-sigpolicy` verifies this switch against the policy above
and keeps signature verification enabled for later `bootc upgrade` operations.
Dropping the flag disables verification for both the switch and those upgrades.
