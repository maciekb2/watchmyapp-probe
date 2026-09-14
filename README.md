# WatchMyApp Probe

Public downloads and installation documentation for the WatchMyApp monitoring probe.
This repository contains distribution artifacts and documentation, not application source.

**Preview:** private-probe support is awaiting the WatchMyApp production rollout.
A downloadable probe does not yet make workspace registration available.

Download the Linux amd64 or arm64 archive and `SHA256SUMS` from
[Releases](https://github.com/maciekb2/watchmyapp-probe/releases).
Follow [the Linux installation guide](INSTALL.md) to verify and install it.
For containers, use the [standalone Docker image](DOCKER.md).

The probe uses outbound HTTPS, an individual workspace credential and a persistent
local buffer. It does not need inbound ports or a VPN connection to WatchMyApp.

For support, contact support@watchmyapp.io. Never post tokens, buffers or private
monitor targets in public issues.

Website: https://watchmyapp.io
