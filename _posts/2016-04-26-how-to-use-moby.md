# How to use Moby

Get Moby

```bash
$ git clone git@github.com:linuxkit/linuxkit.git
```

Start with your docker deamon
On Mac
start docker on Mac

```bash
$ cd linuxkit
$ make
$ export PATH=$PATH:$(pwd)/bin
```
The build will generate linuxkit and moby

Add linuxkit.yml

```bash
$ cat linuxkit.yml
kernel:
  image: "mobylinux/kernel:4.9.x"
  cmdline: "console=ttyS0 console=tty0 page_poison=1"
init:
  - linuxkit/init:42fe8cb1508b3afed39eb89821906e3cc7a70551
  - mobylinux/runc:b0fb122e10dbb7e4e45115177a61a3f8d68c19a9
  - linuxkit/containerd:60e2486a74c665ba4df57e561729aec20758daed
  - mobylinux/ca-certificates:eabc5a6e59f05aa91529d80e9a595b85b046f935
onboot:
  - name: sysctl
    image: "mobylinux/sysctl:2cf2f9d5b4d314ba1bfc22b2fe931924af666d8c"
    net: host
    pid: host
    ipc: host
    capabilities:
     - CAP_SYS_ADMIN
    readonly: true
  - name: binfmt
    image: "linuxkit/binfmt:8881283ac627be1542811bd25c85e7782aebc692"
    binds:
     - /proc/sys/fs/binfmt_misc:/binfmt_misc
    readonly: true
  - name: dhcpcd
    image: "linuxkit/dhcpcd:48e249ebef6a521eed886b3bce032db69fbb4afa"
    binds:
     - /var:/var
     - /tmp/etc:/etc
    capabilities:
     - CAP_NET_ADMIN
     - CAP_NET_BIND_SERVICE
     - CAP_NET_RAW
    net: host
    command: ["/sbin/dhcpcd", "--nobackground", "-f", "/dhcpcd.conf", "-1"]
services:
  - name: rngd
    image: "mobylinux/rngd:3dad6dd43270fa632ac031e99d1947f20b22eec9"
    capabilities:
     - CAP_SYS_ADMIN
    oomScoreAdj: -800
    readonly: true
  - name: nginx
    image: "nginx:alpine"
    capabilities:
     - CAP_NET_BIND_SERVICE
     - CAP_CHOWN
     - CAP_SETUID
     - CAP_SETGID
     - CAP_DAC_OVERRIDE
    net: host
files:
  - path: etc/docker/daemon.json
    contents: '{"debug": true}'
trust:
  image:
    - mobylinux/kernel
outputs:
  - format: kernel+initrd
  - format: iso-bios
  - format: iso-efi
  ```

  ```bash
  $ moby build linuxkit.yml
Extract kernel image: mobylinux/kernel:4.9.x
Add init containers:
Process init image: linuxkit/init:42fe8cb1508b3afed39eb89821906e3cc7a70551
Process init image: mobylinux/runc:b0fb122e10dbb7e4e45115177a61a3f8d68c19a9
Process init image: linuxkit/containerd:60e2486a74c665ba4df57e561729aec20758daed
Process init image: mobylinux/ca-certificates:eabc5a6e59f05aa91529d80e9a595b85b046f935
Add onboot containers:
  Create OCI config for mobylinux/sysctl:2cf2f9d5b4d314ba1bfc22b2fe931924af666d8c
  Create OCI config for linuxkit/binfmt:8881283ac627be1542811bd25c85e7782aebc692
  Create OCI config for linuxkit/dhcpcd:48e249ebef6a521eed886b3bce032db69fbb4afa
Add service containers:
  Create OCI config for mobylinux/rngd:3dad6dd43270fa632ac031e99d1947f20b22eec9
  Create OCI config for nginx:alpine
Add files:
  etc/docker/daemon.json
Create outputs:
  linuxkit-bzImage linuxkit-initrd.img linuxkit-cmdline
  linuxkit.iso
  linuxkit-efi.iso
```

Run the moby image, make sure your docker deamon is running
```bash
$ linuxkit run moby

Welcome to LinuxKit

                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/


/ # [    3.208263] IPVS: Creating netns size=2104 id=1
[    3.208808] IPVS: ftp: loaded support on port[0] = 21
[    3.597846] clocksource: Switched to clocksource tsc
[    3.644015] IPVS: Creating netns size=2104 id=2
[    3.644447] IPVS: ftp: loaded support on port[0] = 21
```

```bash
/ #
/ # runc list
ID          PID         STATUS      BUNDLE                        CREATED                          OWNER
nginx       557         running     /run/containerd/linux/nginx   2017-04-26T13:43:38.4053303Z     root
rngd        604         running     /run/containerd/linux/rngd    2017-04-26T13:43:38.578112356Z   root
/ # halt
Terminated
/ #
/ #
/ #
/ #
/ #
/ #
/ # [   77.040416] reboot: System halted
```

the system needs the xhyve to run 
