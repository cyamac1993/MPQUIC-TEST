# MPQUIC
MPQUIC and QUIC test on a bottlenecked network 
Clone this repository and build sources:

$ git clone https://github.com/thomaswpp/mpquic-sbd.git
$ cd mpquic-sbd
$ ./build.sh
4.2 Build the applications individually
If you built all from ./build.sh, then it is done.



The Go modules are implemented from golang version 1.12. Here, the used modules are built outside of GOPATH. The local setup redirects the modular dependencies to the local implementations.

To build MPQUIC-SBD:

$ cd src/quic-go
$ go build ./...
The Go modules allow recursive build, so that this module must not necessarily be build explicitely. The MPQUIC module can be used by other Go modules via reference in their go.mod.

To build Caddy server:

$ cd src/dash/caddy
$ go build
To build the client proxy module in a shared object (.so) to enable MPQUIC (in go) to work transparently underneath AStream DASH player (in python):

$ cd src/dash/client/proxy_module
$ go build -o proxy_module.so -buildmode=c-shared proxy_module.go
After building the proxy module, copy AStream dependencies. (Probably also requires path change in line 5 of src/dash/client/proxy_module/conn.py)

$ cp src/dash/client/proxy_module/proxy_module.h src/AStream/dist/client/
$ cp src/dash/client/proxy_module/proxy_module.so src/AStream/dist/client/
$ cp src/dash/client/proxy_module/conn.py src/AStream/dist/client/
4.3 Configure and run the video server
For the DASH streaming, Caddy server needs a configuration to serve the video segments. To this end, a file named Caddyfile must be configured.

Example of a Caddyfile:

https://localhost:4242 {
    root <URL to DASH video files>
    tls self_signed
}
Run the server from src/dash/caddy.

For a single-path server:

$ ./caddy -quic
For a multi-path server:

$ ./caddy -quic -mp
4.4 Run the DASH client
Run the AStream client from src/AStream.

For a single-path client:

$ python AStream/dist/client/dash_client.py -m <SERVER URL TO MPD> -p 'basic' -q
For a multi-path client:

$ python AStream/dist/client/dash_client.py -m <SERVER URL TO MPD> -p 'basic' -q -mp
The parameter <SERVER URL TO MPD> has to be entered in doble quotes like this:

$ python AStream/dist/client/dash_client.py -m "https://localhost:4242/output_dash.mpd" -p 'basic' -q -mp

Acknowledgements
This repository contains open software artefacts we have joined with other repositories. We are thankful to:

AStream: a rate adaptation model for DASH. Available at https://github.com/pari685/AStream
Caddy Server. Available at https://caddyserver.com
Multipath QUIC. Available at https://multipath-quic.org
MAppLE: MPQUIC Application Latency Evaluation platform. Available at https://github.com/vuva/MAppLE
Multipath-QUIC schedulers for video streaming applications. Available at https://github.com/deradev/mpquicScheduler
MPQUIC-SBD: Multipath QUIC (MPQUIC) with support of Shared Bottleneck Detection (SBD) Available at [https://github.com/deradev/mpquicScheduler](https://github.com/thomaswpp/mpquic-sbd?tab=readme-ov-file)
