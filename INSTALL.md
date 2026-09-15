# WatchMyApp Probe — Linux installation

Install a private probe on a Linux host that can reach the services you want to
monitor. You need administrator access to a WatchMyApp workspace on Pro or Team.

The probe runs outbound-only and checks the targets assigned by your workspace.
No inbound ports or VPN connection to WatchMyApp are required.

## Register a probe

A verified workspace administrator opens **Probes & locations** at
[app.watchmyapp.io](https://app.watchmyapp.io) and registers a private probe. Save its one-time token in a protected
file. Use a separate registration and buffer for every deployed instance.
Private probes require an eligible plan (Pro: one, Team: three).

Set `PRIVATE_TARGET_CIDRS` in the supplied configuration to only the internal
networks you intend to monitor. Empty means public destinations only. Your workspace settings cannot
expand this local allowlist. Private monitor targets must also be reachable from
this host. Keep TLS verification enabled.

## Download and install

Choose the archive for your Linux architecture from the WatchMyApp probe release.
Obtain `SHA256SUMS` from the same trusted release channel. A checksum detects
corruption; it is not a publisher signature.

On a Linux host with systemd supporting `LoadCredential` and the `%d` credential
directory specifier, download the archive matching the host architecture and its
trusted checksum manifest. Verify before extracting:

```sh
sha256sum --ignore-missing -c SHA256SUMS
# Substitute the exact downloaded filename (amd64 or arm64).
tar -xzf watchmyapp-probe-VERSION-linux-amd64.tar.gz
cd watchmyapp-probe-VERSION-linux-amd64
./watchmyapp-probe --version
```

The host needs trusted CA certificates, DNS resolution and outbound HTTPS access to
WatchMyApp at `api.watchmyapp.io` plus the configured monitoring destinations. Install as the host administrator:

```sh
sudo install -d -m 0700 /etc/watchmyapp
sudo install -m 0755 watchmyapp-probe /usr/local/bin/watchmyapp-probe
sudo install -m 0644 watchmyapp-probe.service /etc/systemd/system/watchmyapp-probe.service
# Only on first installation: preserve an existing operator configuration.
sudo install -m 0600 probe.env.example /etc/watchmyapp/probe.env
# TOKEN_FILE is the protected file saved from the one-time registration screen.
sudo install -m 0600 TOKEN_FILE /etc/watchmyapp/probe-token
sudoedit /etc/watchmyapp/probe.env
sudo systemd-analyze verify /etc/systemd/system/watchmyapp-probe.service
sudo systemctl daemon-reload
sudo systemctl enable --now watchmyapp-probe.service
sudo systemctl status watchmyapp-probe.service
```

The unit runs as a dynamic, unprivileged identity. systemd owns the persistent
`/var/lib/watchmyapp-probe` state directory and copies the root-only token into the
service credential directory at startup. It exposes no listening port. It has no
Linux capabilities and cannot write the general filesystem. Do not replace this
with `PrivateNetwork=yes`, which would prevent useful outbound checks. ICMP still
requires an appropriate unprivileged ping socket policy on the customer's host;
the package does not change host sysctls or grant raw-socket privileges.

Verify **last seen**, location and results in the workspace after startup. A running
process with an offline buffer does not prove a successful Core connection. Read
service logs with `sudo journalctl -u watchmyapp-probe.service`; tokens are not
logged. For offline buffer status, stop the service first and run:

```sh
sudo systemctl stop watchmyapp-probe.service
sudo env BUFFER_PATH=/var/lib/watchmyapp-probe/probe.db /usr/local/bin/watchmyapp-probe status
sudo systemctl start watchmyapp-probe.service
```

For an update, verify the new archive, stop the service, retain a protected copy of
the old binary and a consistent stopped buffer backup, replace only the binary,
and start the service. Do not overwrite `probe.env`, the token or the state directory.
Check the release's buffer compatibility before rolling back a binary against an
updated database. Token replacement requires a restart to reload credentials.
Never run two processes against the same buffer or copy a running buffer file.

To uninstall, first revoke the registration in the workspace, then:

```sh
sudo systemctl disable --now watchmyapp-probe.service
sudo rm /etc/systemd/system/watchmyapp-probe.service /usr/local/bin/watchmyapp-probe
sudo systemctl daemon-reload
```

This intentionally preserves `/etc/watchmyapp` and the persistent state directory,
including the token and any buffered results, for deliberate retention or disposal.
Do not use `systemctl clean --what=state` during ordinary updates or uninstall.

## Credential replacement and revocation

To replace a token, use the workspace action, replace the local protected token
file and restart the service. Revocation prevents new WatchMyApp API operations, but a
disconnected worker can execute previously cached assignments until its lease
expires (up to 24 hours). Stop the local process as well when revoking it.

## Support

Contact support@watchmyapp.io. Include the probe version and a description of the
problem. Never include tokens, credential files or private target details in a
public issue. Do not upload the result buffer to a public repository.
