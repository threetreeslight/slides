## `systemd-nspawn` <br> は便利らしい

@threetreeslight

on Repro Tech Meetup #1

---

## whoami

![](https://avatars3.githubusercontent.com/u/1057490?s=200&v=4)

@threetreeslight / VPoE at Repro

shinjuku.rb, shinjuku-aar, shinjuku-mokumoku, Hacking HR! などのコミュニティオーガナイザーしています


---

## とあるもくもく会にて・・・

「 `systemd-nspawn` が便利すぎてdockerのcommand 忘れた話を聞いたので使ってみた 😇 」という話を聞く 

---

# そんなものが世の中にあるなんて!!!

---

## `systemd-nspawn` って何？

> systemd-nspawn is like the chroot command, but it is **a chroot on @color[red](steroids)**.
>
> -- [archlinux wiki - systemd-nspawn](https://wiki.archlinux.org/index.php/Systemd-nspawn)

---?image=repro-tech-meetup/1/meme-yatta.png?size=auto 100%

# @color[red](強くてヤバイ `chroot`)

---

# 全くわからん

![](repro-tech-meetup/1/meme-oh.png)

---

## もうすこし読み解くと

systemd-nspawnは軽量な名前空間コンテナ技術。具体的には以下を仮想化する

- ファイルシステム階層
- プロセスツリー
- さまざまなIPCサブシステム
- ホストとドメイン名

---

# なるほど？

---

# 以下、間違ってたらごめんなさい 🙇

---

## Dockerの構成技術

1. namespaces <- 🤔
1. Control groups <- 🤔
1. Union file systems
1. Container format

From [dockr overview - the-underlying-technology](https://docs.docker.com/engine/docker-overview/#the-underlying-technology)

---

## `linux namespace`

階層構造に名前空間を切り分け、そのアクセスを制限するやつ。

container内のPIDが1から始まるのはこの子のおかげ。

-- [wiki - Linux namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)

---

## `Control groups(cgroup)`

processをgroupingし、CPU, memory, disk I/O, network, etc.のresouceの利用を監視、制限するやつ。

このおかげで特定プロセスがhostのリソースを専有してしまう事象を防ぐ。

-- [wiki - cgroups](https://en.wikipedia.org/wiki/Cgroups)

---

## で、`chroot` は？

ルートディレクトリを変更して、その外のファイルやコマンドへのアクセスを制限するやつ。

```sh
docker run --rm -it debian:latest sh
apt-get update
apt-get install -y binutils debootstrap
mkdir -p /srv/chroot/wheezy
debootstrap --arch i386 wheezy /srv/chroot/wheezy http://httpredir.debian.org/debian

chroot /srv/chroot/wheezy /bin/bash
I have no name!@1af6c5c55369:/$
# いやっふぅ
```

https://wiki.debian.org/chroot

---

## `systemd-nspawn` の雰囲気

linux namespace + chroot = <br> 
@color[orange](**a chroot on steroids**)

---

## やってみる<br>`debian/jessie64`

```sh
# create a new container using debootstrap
$ CDIR=/var/lib/machines/freedombox
$ sudo debootstrap sid $CDIR
$ sudo systemd-nspawn -D $CDIR --machine FreedomBox

root@FreedomBox:~$ apt-get install -y freedombox-setup

# set root password and stop the container
root@FreedomBox:~$ passwd root
root@FreedomBox:~$ ^D
```

---

## いくぜ！

```sh
# start the container and its services
$ sudo systemd-nspawn -D $CDIR --machine FreedomBox -b
Spawning container FreedomBox on /var/lib/machines/freedombox.
Press ^] three times within 1s to kill container.
/etc/localtime is not a symlink, not updating container timezone.
systemd 239 running in system mode. (+PAM +AUDIT +SELINUX +IMA +APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD -IDN2 +IDN -PCRE2 default-hierarchy=hybrid)
Detected virtualization systemd-nspawn.
Detected architecture x86-64.

Welcome to Debian GNU/Linux buster/sid!

Set hostname to <jessie>.
File /lib/systemd/system/systemd-journald.service:36 configures an IP firewall (IPAddressDeny=any), but the local system does not support BPF/cgroup based firewalling.
Proceeding WITHOUT firewalling in effect! (This warning is only shown for the first loaded unit using IP firewalling.)
[  OK  ] Reached target Swap.
[  OK  ] Reached target System Time Synchronized.
[  OK  ] Listening on Syslog Socket.
[  OK  ] Started Dispatch Password Requests to Console Directory Watch.
[  OK  ] Listening on Journal Socket.
system.slice: Failed to reset devices.list: Operation not permitted
dev-hugepages.mount: Failed to reset devices.list: Operation not permitted
         Mounting Huge Pages File System...
systemd-remount-fs.service: Failed to reset devices.list: Operation not permitted
         Starting Remount Root and Kernel File Systems...
[  OK  ] Reached target Remote File Systems.
dev-mqueue.mount: Failed to reset devices.list: Operation not permitted
         Mounting POSIX Message Queue File System...
[  OK  ] Started Forward Password Requests to Wall Directory Watch.
[  OK  ] Reached target Paths.
system-getty.slice: Failed to reset devices.list: Operation not permitted
[  OK  ] Created slice system-getty.slice.
[  OK  ] Created slice User and Session Slice.
resolvconf.service: Failed to reset devices.list: Operation not permitted
         Starting Nameserver information manager...
[  OK  ] Reached target Slices.
[  OK  ] Reached target Local Encrypted Volumes.
[  OK  ] Listening on initctl Compatibility Named Pipe.
ifupdown-pre.service: Failed to reset devices.list: Operation not permitted
         Starting Helper to synchronize boot up for ifupdown...
[  OK  ] Listening on Journal Socket (/dev/log).
systemd-journald.service: Failed to reset devices.list: Operation not permitted
         Starting Journal Service...
[  OK  ] Mounted Huge Pages File System.
[  OK  ] Mounted POSIX Message Queue File System.
[  OK  ] Started Remount Root and Kernel File Systems.
systemd-sysusers.service: Failed to reset devices.list: Operation not permitted
         Starting Create System Users...
[  OK  ] Started Helper to synchronize boot up for ifupdown.
[  OK  ] Started Nameserver information manager.
[  OK  ] Started Create System Users.
[  OK  ] Reached target Local File Systems (Pre).
[  OK  ] Reached target Local File Systems.
[  OK  ] Started Journal Service.
         Starting Flush Journal to Persistent Storage...
[  OK  ] Started Flush Journal to Persistent Storage.
         Starting Create Volatile Files and Directories...
[  OK  ] Started Create Volatile Files and Directories.
         Starting Update UTMP about System Boot/Shutdown...
[  OK  ] Started Update UTMP about System Boot/Shutdown.
[  OK  ] Reached target System Initialization.
[  OK  ] Started Daily apt download activities.
[  OK  ] Listening on D-Bus System Message Bus Socket.
[  OK  ] Started Daily apt upgrade and clean activities.
[  OK  ] Started Daily Cleanup of Temporary Directories.
[  OK  ] Listening on Avahi mDNS/DNS-SD Stack Activation Socket.
[  OK  ] Reached target Sockets.
[  OK  ] Reached target Basic System.
         Starting Modem Manager...
[  OK  ] Started D-Bus System Message Bus.
         Starting firewalld - dynamic firewall daemon...
         Starting WPA supplicant...
[  OK  ] Started handle automounting.
         Starting Restore /etc/resolv.conf if the system crashed before the ppp link was shut down...
         Starting System Logging Service...
         Starting Avahi mDNS/DNS-SD Stack...
[  OK  ] Started Run certbot twice daily.
         Starting Name Service Cache Daemon...
[  OK  ] Started Clean PHP session files every 30 mins.
[  OK  ] Reached target Timers.
         Starting Login Service...
[  OK  ] Started Restore /etc/resolv.conf if the system crashed before the ppp link was shut down.
[  OK  ] Started Name Service Cache Daemon.
[  OK  ] Started Login Service.
[  OK  ] Started System Logging Service.
[  OK  ] Started Avahi mDNS/DNS-SD Stack.
[  OK  ] Started WPA supplicant.
         Starting Authorization Manager...
[  OK  ] Started Authorization Manager.
[  OK  ] Started Modem Manager.
[  OK  ] Started firewalld - dynamic firewall daemon.
[  OK  ] Reached target Network (Pre).
         Starting Raise network interfaces...
         Starting Network Manager...
[  OK  ] Started Raise network interfaces.
[  OK  ] Started Network Manager.
         Starting Network Manager Wait Online...
[  OK  ] Reached target Network.
         Starting Fail2Ban Service...
         Starting OpenBSD Secure Shell server...
         Starting Network Time Service...
         Starting Permit User Sessions...
[  OK  ] Started Plinth Web Interface.
[  OK  ] Started Unattended Upgrades Shutdown.
[  OK  ] Started Fail2Ban Service.
[FAILED] Failed to start OpenBSD Secure Shell server.
See 'systemctl status ssh.service' for details.
[  OK  ] Started Permit User Sessions.
[  OK  ] Started Console Getty.
[  OK  ] Reached target Login Prompts.
         Starting Hostname Service...
[  OK  ] Started Network Time Service.
[  OK  ] Started Hostname Service.
         Starting Network Manager Script Dispatcher Service...
[  OK  ] Started Network Manager Script Dispatcher Service.
[  OK  ] Started Network Manager Wait Online.
[  OK  ] Reached target Network is Online.
         Starting LSB: OpenLDAP standalone server (Lightweight Directory Access Protocol)...
[  OK  ] Started LSB: OpenLDAP standalone server (Lightweight Directory Access Protocol).
         Starting LSB: LDAP connection daemon...
[  OK  ] Started LSB: LDAP connection daemon.
         Starting The Apache HTTP Server...
[  OK  ] Started Regular background program processing daemon.
[  OK  ] Started The Apache HTTP Server.
[  OK  ] Reached target Multi-User System.
[  OK  ] Reached target Graphical Interface.
         Starting Update UTMP about System Runlevel Changes...
[  OK  ] Started Update UTMP about System Runlevel Changes.

Debian GNU/Linux buster/sid jessie console

jessie login: root
Password:
Linux jessie 3.16.0-6-amd64 #1 SMP Debian 3.16.56-1+deb8u1 (2018-05-08) x86_64

                         .--._    _.--.
                        (     \  /     )
                         \     /\     /
                          \_   \/   _/
                           /        \
                          (    /\    )
                           `--'  `--'

                           FreedomBox

FreedomBox is a pure blend of Debian GNU/Linux.  FreedomBox manual is
available in /usr/share/doc/plinth.

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@jessie:~#
root@jessie:~#
```

---

## やったね！

普通のdocker imageをtarで固めたのをnspawnで立ち上げても動く（らしい)

---

## 何が嬉しいか？(捻り出す)

- dockerに関わるecosystemが不要なので、純粋に隔離環境つくるときはありそう？
- DockerのようにLayerで管理しないのでimageが軽い。
- systemdと組み合わせて起動時にいい感じに立てとくとかもできる（と思う）
- ただのfileなので、気合入れればgit管理できる(gitignoreはいっぱい書く)

---

## だが `systemd`が動くところしか動かない

ちょっと環境汚さず動かすのなら、<br>chrootとdebootstrapで良い気もする 🤔

---

## コンテナ技術便利 ❤️

---

# references

- [chroot, cgroups and namespaces — An overview](https://itnext.io/chroot-cgroups-and-namespaces-an-overview-37124d995e3d)
- [archlinux wiki - systemd-nspawn](https://wiki.archlinux.org/index.php/Systemd-nspawn)
- [debian - nspawn](https://wiki.debian.org/nspawn)
- [wiki - Linux namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)
- [wiki - cgroups](https://en.wikipedia.org/wiki/Cgroups)
