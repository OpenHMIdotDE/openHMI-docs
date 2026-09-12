# OpenHMI Remote Access Setup

## Purpose

This document defines the standard provisioning procedure for administrative remote access to an OpenHMI Linux node.

The OpenHMI remote-access model is:

```text
Primary administrative path:   Tailscale + OpenSSH
Secondary recovery path:        Raspberry Pi Connect (where available/configured)
Local fallback:                 LAN / physical console
Public inbound SSH:             not required
```

OpenHMI uses **standard OpenSSH over the Tailscale network**. Tailscale SSH is intentionally not used.

The procedure separates two independent security layers:

1. **Network access** — Tailscale
2. **Host authentication** — OpenSSH client-specific keys

A Tailscale identity does not replace an SSH identity.

---

## 1. SSH identity model

Every administrative client has its own SSH key pair.

Example:

```text
MacBook
└── dedicated OpenHMI ED25519 key

Debian development laptop
└── dedicated OpenHMI ED25519 key

iPhone / SSH client
└── dedicated OpenHMI ED25519 key

Offline recovery media
└── dedicated OpenHMI recovery ED25519 key
```

The public keys of authorized clients are installed on each OpenHMI node in:

```text
/home/hmi/.ssh/authorized_keys
```

### Rules

- Never copy a normal private SSH key from one client device to another.
- Every client gets a separate identity.
- Use ED25519 for newly created OpenHMI identities.
- Normal administrative private keys should be protected with a passphrase.
- Private keys, passphrases, Tailscale auth keys and other secrets must never be committed to a repository.
- The recovery private key is kept offline and is not configured as a normal daily-use identity.
- Keep old working access until a replacement identity has been tested successfully.

---

## 2. Initial access to a new node

A new node requires one trusted bootstrap path before SSH keys can be provisioned.

Possible bootstrap paths are:

- local keyboard/console
- existing LAN/password access during commissioning
- Raspberry Pi Connect
- the offline recovery identity on an already provisioned node

For Raspberry Pi based OpenHMI systems, Raspberry Pi Connect can provide a useful independent bootstrap/recovery path.

Check the target identity and addresses:

```bash
whoami
hostname
hostname -I
```

The standard OpenHMI service/admin user is currently:

```text
hmi
```

---

## 3. Create a client-specific SSH identity

Create an explicitly named OpenHMI key instead of relying on a generic `id_ed25519`:

```bash
ssh-keygen -t ed25519 -a 100 \
  -f ~/.ssh/openhmi_client_ed25519 \
  -C "OpenHMI client"
```

Set a passphrase when prompted.

Files created:

```text
~/.ssh/openhmi_client_ed25519       PRIVATE — never copy or publish
~/.ssh/openhmi_client_ed25519.pub   PUBLIC
```

Verify the public-key fingerprint:

```bash
ssh-keygen -lf ~/.ssh/openhmi_client_ed25519.pub
```

Display only the public key when it needs to be transferred:

```bash
cat ~/.ssh/openhmi_client_ed25519.pub
```

The important rule is that every administrative client uses its own private key.

---

## 4. Provision a public key on the OpenHMI node

On the target node as user `hmi`:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

Add the complete public key as one line, for example:

```text
ssh-ed25519 AAAA... OpenHMI client
```

Then:

```bash
chmod 600 ~/.ssh/authorized_keys
```

Only public keys belong in `authorized_keys`.

---

## 5. Verify OpenSSH

Check that the SSH service is enabled and running:

```bash
systemctl is-enabled ssh
systemctl is-active ssh
```

If required:

```bash
sudo systemctl enable --now ssh
```

Test the new identity explicitly before removing any previous access:

```bash
ssh -o IdentitiesOnly=yes \
  -i ~/.ssh/openhmi_client_ed25519 \
  hmi@<NODE-IP>
```

A successful login with the explicitly selected key is required before obsolete keys are removed.

If a rebuilt device legitimately has a new SSH host key and reuses an old address/name, verify that the target really is the replaced device before removing the old known-host entry:

```bash
ssh-keygen -R <HOSTNAME-OR-IP>
```

Never ignore a host-key warning without first establishing why the key changed.

---

## 6. Configure SSH client aliases

Client configuration belongs in:

```text
~/.ssh/config
```

Example:

```sshconfig
Host openhmi-node
    HostName <TAILSCALE-IP-OR-MAGICDNS-NAME>
    User hmi
    IdentityFile ~/.ssh/openhmi_client_ed25519
    IdentitiesOnly yes
```

Protect the configuration:

```bash
chmod 600 ~/.ssh/config
```

Then normal access becomes:

```bash
ssh openhmi-node
```

If an explicit LAN fallback alias is desired, keep it separate from the Tailscale alias.

---

## 7. Install Tailscale on an OpenHMI node

The exact package instructions depend on the installed Debian/Raspberry Pi OS release. The supported generic Linux installer currently used by Tailscale is:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

For controlled production images, distribution-specific repository installation may be preferred so the package source is explicit and reproducible.

After installation verify the daemon:

```bash
systemctl is-enabled tailscaled
systemctl is-active tailscaled
```

If necessary:

```bash
sudo systemctl enable --now tailscaled
```

---

## 8. Join the OpenHMI tailnet

For manual commissioning:

```bash
sudo tailscale up
```

Tailscale prints an authentication URL. Open that URL on an authorized administrator device and approve the new node in the intended tailnet.

Then verify:

```bash
tailscale status
tailscale ip -4
```

Where MagicDNS is configured, its Tailscale hostname may be used in `~/.ssh/config` instead of a numeric Tailscale address.

### Important: do not enable Tailscale SSH

OpenHMI currently uses:

```text
Tailscale network
      ↓
standard OpenSSH
      ↓
client-specific SSH public-key authentication
```

Do **not** enable Tailscale SSH (`--ssh`) as part of standard OpenHMI provisioning.

This keeps network authorization and host SSH authentication separate and preserves the established OpenHMI key/recovery model.

---

## 9. Automated / fleet provisioning

For larger deployments, Tailscale auth keys can be used to register nodes without interactive browser login.

Conceptually:

```bash
sudo tailscale up --auth-key=<AUTH-KEY>
```

Do not put an auth key into source code, Git, provisioning documentation, shell scripts committed to the repository, or other persistent plaintext locations.

Prefer short-lived/one-off credentials where practical. For reusable automation credentials, use an appropriate secret-management mechanism.

Fleet provisioning, tags, ACL/grants and organisation-level policy should be documented separately when the OpenHMI fleet-management model is finalized.

---

## 10. Recovery identity

The recovery identity is a separate offline SSH key pair.

Principle:

```text
Normal operation:

client private keys ─────> OpenHMI authorized_keys

Emergency recovery:

offline recovery private key
        ↓ temporary use only
OpenHMI node
        ↓
provision/recover a normal client-specific identity
        ↓
remove recovery private key from the temporary client
```

The recovery **public** key may be preinstalled on OpenHMI nodes. The recovery **private** key remains offline.

Do not use the recovery key as the normal administrative identity.

---

## 11. Raspberry Pi Connect as secondary access

Where configured, Raspberry Pi Connect is an independent secondary administrative/recovery path.

It is useful when, for example:

- the normal SSH client identity is unavailable,
- Tailscale configuration requires repair,
- a new device needs initial SSH-key provisioning.

It is **not** the primary OpenHMI administrative network and does not replace Tailscale + OpenSSH.

For headless production controllers, remote shell access is generally sufficient; screen sharing should only be enabled where it provides an operational benefit.

Whether Raspberry Pi Connect is enabled on a production system may also depend on customer security policy and deployment requirements.

---

## 12. New-device commissioning checklist

For every new OpenHMI Linux node:

```text
[ ] Set a unique hostname
[ ] Confirm `hmi` user / intended admin account
[ ] Establish trusted bootstrap access
[ ] Enable and verify OpenSSH
[ ] Install required client public SSH keys
[ ] Install recovery public key
[ ] Verify ~/.ssh permissions
[ ] Test each required SSH identity explicitly
[ ] Install Tailscale
[ ] Join the correct tailnet
[ ] Verify tailscaled starts automatically
[ ] Verify Tailscale address / MagicDNS identity
[ ] Test standard OpenSSH over Tailscale
[ ] Configure client ~/.ssh/config aliases
[ ] Verify access after reboot
[ ] Verify access from an external/non-site network where appropriate
[ ] Remove obsolete bootstrap credentials/keys only after successful verification
[ ] Confirm no private keys or auth tokens were stored on the node unnecessarily
```

---

## 13. Verification after reboot

Reboot the node:

```bash
sudo reboot
```

After it returns, verify remotely:

```bash
ssh <OPENHMI-SSH-ALIAS>
```

On the node:

```bash
systemctl is-active ssh
systemctl is-active tailscaled
tailscale status
```

For a production-ready remote-access setup, the node must recover SSH and Tailscale connectivity automatically after reboot without requiring local intervention.

---

## 14. Security baseline

- No public inbound SSH port is required.
- Tailscale is the primary remote network path.
- Standard OpenSSH provides host authentication.
- Separate client identities are mandatory for administrative clients.
- Use passphrase-protected private keys for normal workstation identities.
- Keep the recovery private key offline.
- Require 2FA on administrative service accounts where supported.
- Remove obsolete/decommissioned devices and credentials.
- Apply least privilege to remote access.
- Do not commit secrets, private keys or authentication tokens.
- Preserve at least one verified recovery path before changing remote-access configuration.

---

## Related architecture decisions

- **ADR-007 — Remote Access Strategy**
- **ADR-015 — Remote Update and Recovery Architecture**

Architecture decisions are maintained internally during active development and may be published separately when suitable for stable public documentation.
