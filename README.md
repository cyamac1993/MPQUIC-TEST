THe whole Body of this Project is based on the MPQUIC-SBD: Multipath QUIC (MPQUIC) with support of Shared Bottleneck Detection (SBD) which is available here: https://github.com/thomaswpp/mpquic-sbd/tree/master

For the sake of modifying codes and topologies this repo has been created.

 


##  This repository

This repository contains the following open-source code: 
* [src/quic-go](https://github.com/thomaswpp/mpquic-sbd/tree/master/src/quic-go): the popular [MPQUIC implementation](https://multipath-quic.org/) in golang we extend to include SBD.
* [src/caddy](https://github.com/thomaswpp/mpquic-sbd/tree/master/src/caddy): the [Caddy HTTP server implementation](https://caddyserver.com/) in golang.
* [src/AStream](https://github.com/thomaswpp/mpquic-sbd/tree/master/src/AStream): a [DASH player](https://github.com/pari685/AStream) emulator in python. 
* [example](https://github.com/thomaswpp/mpquic-sbd/tree/master/example): files and scripts to process the video segments and manifest MPD files.
* [src/dash/client/proxy_module](https://github.com/thomaswpp/mpquic-sbd/tree/master/src/dash/client/proxy_module): a client-side proxy module which provides an interface through a shared object (`.so`) to enable MPQUIC implementations (in golang) underneath the AStream DASH player (in python).

All the source code files are targeted to run on Linux 64-bit hosts.

## 2. Guidelines


There are two ways to do the experiment adn run this project:

 1. You can use our VM (QEMU/KVM image) ready to run which is available here https://drive.google.com/drive/folders/15Ei5wULQDCXceoCVAE-vPrl5-oS8Ugi1?usp=share_link and refer to the original repo.

 2. If you have your own experimental environment, you can download and install the MPQUIC-SBD artefacts by yourself. 



### 3.1 Prepare the experimental environment

To reproduce the experiments and measurements in [1], you have to firstly: 

 1. Have a Linux host. For ease install, we suggest a user-friendly Linux such as Ubuntu. For instance, we installed Ubuntu 20.04.5 LTS in our host, which is consisted of a server node HP Proliant ML30 Gen9, Intel Xeon 4-Core 3GHz, 8GB RAM, 1TB hard disk.
 2. Download our prepare VM (QEMU/KVM) image from:
 ```
 https://drive.google.com/drive/folders/15Ei5wULQDCXceoCVAE-vPrl5-oS8Ugi1?usp=share_link
 ```
 3. Install the KVM package on your Linux host. Instructions are available at the [KVM webpage](https://www.linux-kvm.org).

 4. Install the Virtual Manager Machine `virt-manager` to launch the VM. Instructions are available at the [virt-manager webpage](https://virt-manager.org/).



##  Deploy MPQUIC-SBD in your experimental environment

###  Clone and build

To clone this repository and build sources:
```
$ ./build.sh
```

### 4.2 Build the applications individually


To build MPQUIC-SBD:
```
$ cd src/quic-go
$ go build ./...
```
The Go modules allow recursive build, so that this module must not necessarily be build explicitely.
The MPQUIC module can be used by other Go modules via reference in their go.mod.

To build Caddy server:
```
$ cd src/dash/caddy
$ go build
```

To build the client proxy module in a shared object (.so) to enable MPQUIC (in go) to work transparently underneath AStream DASH player (in python):
```
$ cd src/dash/client/proxy_module
$ go build -o proxy_module.so -buildmode=c-shared proxy_module.go
```

After building the proxy module, copy AStream dependencies.
(Probably also requires path change in line 5 of src/dash/client/proxy_module/conn.py)
```
$ cp src/dash/client/proxy_module/proxy_module.h src/AStream/dist/client/
$ cp src/dash/client/proxy_module/proxy_module.so src/AStream/dist/client/
$ cp src/dash/client/proxy_module/conn.py src/AStream/dist/client/
```

###  Configure and run the video server

For the DASH streaming, Caddy server needs a configuration to serve the video segments. 
To this end, a file named [*Caddyfile*](https://caddyserver.com/tutorial/caddyfile) must be configured.

Example of a Caddyfile:
```
=======
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

>>>>>>> fab09dd35611223ecf8cf6fcc93a2843b25ea438
https://localhost:4242 {
    root <URL to DASH video files>
    tls self_signed
}
<<<<<<< HEAD
```

Run the server from `src/dash/caddy`.

For a single-path server:
```
$ ./caddy -quic
```

For a multi-path server:
```
$ ./caddy -quic -mp
```
 
### 4.4 Run the DASH client

Run the AStream client from `src/AStream`.

For a single-path client:
```
$ python AStream/dist/client/dash_client.py -m <SERVER URL TO MPD> -p 'basic' -q
```

For a multi-path client:
```
$ python AStream/dist/client/dash_client.py -m <SERVER URL TO MPD> -p 'basic' -q -mp
```

The parameter `<SERVER URL TO MPD>` has to be entered in doble quotes like this: 
```
$ python AStream/dist/client/dash_client.py -m "https://localhost:4242/output_dash.mpd" -p 'basic' -q -mp
```

### 4.5 Bulk transfers

If you want to experiment MPQUIC-SBD in bulk transfers, you have to use the script `bulk_transfer.py`.

For a single-path client:
```
python AStream/dist/client/bulk_transfer.py -m <SERVER URL TO MPD> -q
```

For a multi-path client:
```
python AStream/dist/client/bulk_transfer.py -m <SERVER URL TO MPD> -q -mp
```

# Acknowledgements

This repository contains open software artefacts we have joined with other repositories. 
We are thankful to:

- **AStream: a rate adaptation model for DASH.** Available at https://github.com/pari685/AStream
- **Caddy Server.** Available at https://caddyserver.com
- **Multipath QUIC.** Available at https://multipath-quic.org
- **MAppLE: MPQUIC Application Latency Evaluation platform.** Available at https://github.com/vuva/MAppLE
- **Multipath-QUIC schedulers for video streaming applications.** Available at https://github.com/deradev/mpquicScheduler
=======
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
>>>>>>> fab09dd35611223ecf8cf6fcc93a2843b25ea438
