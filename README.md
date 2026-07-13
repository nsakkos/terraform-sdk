# terraform-cli Workshop SDK

A **thin** [Workshop](https://ubuntu.com/workshop/docs/) SDK that ships the
HashiCorp Terraform CLI as a prebuilt binary. Optional capabilities are 
exposed as plugs the consuming workshop wires as needed.

## What it ships

- The `terraform` CLI (from the official `releases.hashicorp.com` archive),
  installed at `$SDK/bin/terraform` and added to the system `PATH`.
- `TF_PLUGIN_CACHE_DIR` pointed at `/home/workshop/.cache/terraform`.

## Plugs

| Plug | Interface | Auto-connect | Purpose |
|------|-----------|--------------|---------|
| `ssh-agent` | `ssh-agent` | No (opt-in) | Fetch private git modules over SSH. Connect with `workshop connect <workshop>/terraform-cli:ssh-agent`. |

Both extras are opt-in — nothing crosses the host boundary unless you wire it.

## Slots

This SDK declares no slots — it consumes capabilities, it does not provide them.

## Persistent plugin cache (optional)

`setup-base` sets `TF_PLUGIN_CACHE_DIR=/home/workshop/.cache/terraform` and
`setup-project` creates that directory, so terraform always has a working
provider cache. By default it is a plain **local** directory: no host mount, no
cross-boundary connection. It does **not** survive a rebuilding workshop refresh.

The SDK deliberately does **not** declare a mount plug for it. A `mount` plug
*always* auto-connects to `system:mount` (there is no per-plug opt-out), so
shipping one would bind a host directory into every workshop by default. 
Keeping the cache local by default preserves isolation; a consumer who wants it 
to persist across refreshes grafts a mount at that path in their own workshop 
definition (extending the SDK's interfaces without the publisher's involvement — see
[Plugs, slots, connections](https://ubuntu.com/workshop/docs/explanation/workshops/concepts/#exp-workshop-definition-connections)).

## Build and try locally

```bash
sdkcraft try                         # packs terraform-cli_<arch>_<base>.sdk into the try area
# add `- name: try-terraform-cli` under sdks: in a scratch workshop, then:
workshop launch --verbose --wait-on-error
workshop exec -- terraform version
```

Iterate with `sdkcraft clean && sdkcraft try` then `workshop refresh`.

## Test

```bash
sdkcraft test        # runs the spread tests under tests/ against a clean container
```

## Versioning

The `VERSION` file is the single source of truth for the pinned upstream
Terraform release; `sdkcraft.yaml` adopts it at build time (`adopt-info` +
`craftctl set version`). Renovate (see `renovate.json`) opens PRs to bump
`VERSION` as new Terraform releases ship — no separate literal to keep in sync.


## License and copyright

Copyright 2026 Canonical Ltd.

This program is free software: you can redistribute it and/or modify it under
the terms of the
[GNU Lesser General Public License version 2.1 (LGPLv2.1)](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html)
as published by the Free Software Foundation.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU Lesser General Public License for more details.

The packaged binary is Terraform, licensed under the Business Source License
(BUSL-1.1) for releases >= 1.6. This SDK only downloads and repackages the
official archive; the `license:` field reflects the bundled binary's terms.
