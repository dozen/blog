title: systemd-modules-loadがコケる
date: 2017-05-06 00:30:00
author: me
preview: 
tags:
   - linux
   - Raspberry Pi2

---

`bcm2708-rng`はラズパイのハードウェア乱数生成器を使用するためのカーネルモジュールですが、それが読み込めて無くて `systemd-modules-load.service`がコケていました。

```
● systemd-modules-load.service - Load Kernel Modules
   Loaded: loaded (/usr/lib/systemd/system/systemd-modules-load.service; static; vendor preset: disabled)
   Active: failed (Result: exit-code) since Wed 2017-02-01 10:11:51 JST; 20s ago
     Docs: man:systemd-modules-load.service(8)
           man:modules-load.d(5)
  Process: 850 ExecStart=/usr/lib/systemd/systemd-modules-load (code=exited, status=1/FAILURE)
 Main PID: 850 (code=exited, status=1/FAILURE)

 2月 01 10:11:51 pi systemd[1]: Starting Load Kernel Modules...
 2月 01 10:11:51 pi systemd-modules-load[850]: Failed to find module 'bcm2708-rng'
 2月 01 10:11:51 pi systemd[1]: systemd-modules-load.service: Main process exited, code=exited, status=1/FAILURE
 2月 01 10:11:51 pi systemd[1]: Failed to start Load Kernel Modules.
 2月 01 10:11:51 pi systemd[1]: systemd-modules-load.service: Unit entered failed state.
 2月 01 10:11:51 pi systemd[1]: systemd-modules-load.service: Failed with result 'exit-code'.
```

いつの間にかサポートされなくなってたみたいです。

<a href="https://archlinuxarm.org/forum/viewtopic.php?f=64&t=10153" target="_blank">Arch Linux ARM • View topic - Solved: RPi 2: bcm2708-rng module missing after update</a>

`/etc/modules-load.d/raspberrypi.conf`に読み込むカーネルモジュールを列挙していましたが、`bcm2708-rng`の代わりに`bcm2835-rng`を使うよう変更しました。

```
● systemd-modules-load.service - Load Kernel Modules
   Loaded: loaded (/usr/lib/systemd/system/systemd-modules-load.service; static; vendor preset: disabled)
   Active: active (exited) since Sat 2017-05-06 00:31:18 JST; 2s ago
     Docs: man:systemd-modules-load.service(8)
           man:modules-load.d(5)
  Process: 2385 ExecStart=/usr/lib/systemd/systemd-modules-load (code=exited, status=0/SUCCESS)
 Main PID: 2385 (code=exited, status=0/SUCCESS)

 5月 06 00:31:18 pi systemd[1]: Starting Load Kernel Modules...
 5月 06 00:31:18 pi systemd[1]: Started Load Kernel Modules.
```

無事systemd-modules-loadが起動できました🐘
