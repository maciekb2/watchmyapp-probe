# WatchMyApp Probe with Docker

Run a private probe on a host that can reach your internal services. A verified
workspace administrator can register one in **Probes & locations** at
[app.watchmyapp.io](https://app.watchmyapp.io). Pro includes one private probe; Team
includes three.

Image: `ghcr.io/maciekb2/watchmyapp-probe:0.1.0-preview.a579ced`.
Linux amd64 and arm64 are available. Use the immutable digest below.

Create a private probe in your workspace and save its one-time token to
`/etc/watchmyapp/probe-token`. The container runs as UID/GID 10001. Ensure only
that runtime identity can read the token (for rootful Docker: owner 10001:10001,
mode 0400, protected parent directory). Adapt ownership for rootless UID mappings.
Never put the token in an image, command-line argument or public issue.

```sh
docker run -d --name watchmyapp-probe --restart unless-stopped \
  --read-only --cap-drop ALL --security-opt no-new-privileges \
  --mount type=bind,src=/etc/watchmyapp/probe-token,dst=/run/secrets/probe-token,readonly \
  --mount type=volume,src=watchmyapp-probe-buffer,dst=/data \
  --env CORE_URL=https://api.watchmyapp.io \
  --env PROBE_TOKEN_FILE=/run/secrets/probe-token \
  --env PRIVATE_TARGET_CIDRS=10.20.0.0/24 \
  ghcr.io/maciekb2/watchmyapp-probe@sha256:5b6fd39226b3dd3f95e123f7ab92a12ce8c1ac33c3746388baec872a0e46eb63
```

Replace the example CIDR with only your intended internal networks; empty means
public targets only. No inbound ports, privileged mode or host networking are
required. ICMP depends on host unprivileged ping socket policy.

A fresh named volume inherits the image's `/data` ownership. An existing buffer
volume must be writable by UID 10001. Verify last contact and fresh results in the
workspace; a running container may be buffering offline.

For updates, stop the old container, keep its image digest and a consistent
protected backup of the stopped buffer, then recreate only the container with
the new digest and the same token and volume. Never run two processes against one
buffer. Check buffer compatibility before rollback. Replacing a token requires a
restart. Revocation also requires stopping the local container: cached assignments
can continue until their lease expires (up to 24 hours).

Removing the container preserves the named volume. Retain or dispose of the token
and buffer deliberately. Support: support@watchmyapp.io.
