# ansible-role-nftables

Ansible role to manage an `nftables` ruleset from simple template files and a
directory-based layout.

The role:

- Installs the `nftables` package (Ubuntu via `apt`)
- Ensures the `nftables` service is enabled and started
- Deploys `/etc/nftables.conf` from a Jinja2 template
- Includes rule fragments from `/etc/nftables.d/common`, `/etc/nftables.d/site`,
  and `/etc/nftables.d/host`
- Validates the configuration with `nft -c -f /etc/nftables.conf` before
  reloading the service

---

## Rule layout and logic

At runtime, the role expects rules to be present under the role’s `files/`
directory:

1. **Common rules**: applied to **all** routers/hosts
   - **Path**: `files/common/*.nft`
   - **Deployed to**: `/etc/nftables.d/common/`

2. **Site rules**: applied to all routers/hosts that belong to a given site
   - **Path**: `files/sites/<site_name>/*.nft`
   - **Deployed to**: `/etc/nftables.d/site/`
   - The `<site_name>` is the value of the host variable
     `nftables_rules_site`.

3. **Host rules**: applied only to a specific router/host
   - **Path**: `files/hosts/<inventory_hostname>/*.nft`
   - **Deployed to**: `/etc/nftables.d/host/`
   - `<inventory_hostname>` is the Ansible inventory hostname for that target.

The generated `/etc/nftables.conf` simply includes these directories:

- `include "/etc/nftables.d/common/*.nft"`
- `include "/etc/nftables.d/site/*.nft"`
- `include "/etc/nftables.d/host/*.nft"`

---

## Example rules

This repository ships with example rule sets under `example_rules/` to help you
get started:

- `example_rules/common/*.nft`
- `example_rules/sites/example-site01/*.nft`
- `example_rules/hosts/example-router01/*.nft`

To build your own rules:

1. Copy the relevant example directories into `files/`:
   - Copy `example_rules/common/*` to `files/common/`
   - Copy `example_rules/sites/<your_site>/*` to
     `files/sites/<your_site>/`
   - Copy `example_rules/hosts/<your_host>/*` to
     `files/hosts/<your_host>/`
2. Adjust the `.nft` files to match your environment (interfaces, addresses,
   sets, etc.).

The examples are **not** deployed automatically; only files under `files/`
are used by the role.

---

## Required variables

Per host, you must define:

- **`nftables_rules_site`**: the site identifier used to select which
  `files/sites/<site_name>/` directory to deploy.

The role also relies on Ansible’s built-in `inventory_hostname` to select
host-specific rules under `files/hosts/<inventory_hostname>/`.

If `nftables_rules_site` is empty or undefined, the role will fail early with
an assertion error.

### Example inventory snippet

```ini
[routers]
router01 nftables_rules_site=example-site01
router02 nftables_rules_site=example-site01
router03 nftables_rules_site=another-site...
```

## Example playbook

```yaml
- name: Configure nftables on routers
  hosts: routers
  become: true

  roles:
    - role: nftablesMake sure that:
```

- You have created the appropriate rule files under `files/common/`,
  `files/sites/<site_name>/`, and/or `files/hosts/<inventory_hostname>/`.
- Each host defines `nftables_rules_site` in inventory or host vars.

---

## Testing and safety

Some suggestions for testing this role safely:

1. **Dry-run / check mode**
   - Run the playbook in check mode first:
    `ansible-playbook -i inventory site.yml --check --diff`
   - This shows what would change without touching the firewall.

2. **Apply to a test host first**
   - Start with a non-critical host or a lab VM.
   - After a run, inspect the rules:
    `sudo nft list ruleset`

3. **Manual validation**
   - The role already validates the configuration via:
    `nft -c -f /etc/nftables.conf`
   - You can run that manually as an extra safety check before applying any
    additional changes to your rule files.

4. **Have out-of-band access**
   - When testing new firewall rules, ensure you have console or out-of-band
     access so you can recover if you accidentally lock yourself out.

These steps do not require any extra configuration in the role and are a good
baseline until you add more advanced testing (e.g. Molecule) later.
