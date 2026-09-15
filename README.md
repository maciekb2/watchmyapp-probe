# WatchMyApp Probe

Public downloads and installation documentation for the WatchMyApp monitoring probe.
Private probes monitor services reachable from your own network. A verified workspace
administrator can register a probe in **Probes & locations** at
[app.watchmyapp.io](https://app.watchmyapp.io). Pro includes one private probe; Team
includes three.

Download the Linux amd64 or arm64 archive and `SHA256SUMS` from
[Releases](https://github.com/maciekb2/watchmyapp-probe/releases).
Follow [the Linux installation guide](INSTALL.md) to verify and install it.
For containers, use the [standalone Docker image](DOCKER.md).

The probe uses outbound HTTPS, an individual workspace credential and a persistent
local buffer. It does not need inbound ports or a VPN connection to WatchMyApp.

For support, contact support@watchmyapp.io. Never post tokens, buffers or private
monitor targets in public issues.

Website: https://watchmyapp.io
