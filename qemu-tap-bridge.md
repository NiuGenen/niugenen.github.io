# QEMU net with tap

## what we need

- qemu & disk image
- kernel module tun
- kernel configure with BRIDGE support
- tools
    - `sudo apt install bridge-utils` 
    - `sudo apt install uml-utilities` 

## what to do

- enable tun
  
    - `sudo modprobe tun` 
    
- create bridge
    - `sudo brctl addbr virbr0 `
    - `sudo brctl stp virbr0 on `
    
- create tap

    - `sudo ip tuntap   add  dev tap0 mode tap `
    - `sudo ip link set tap0 up` 

- link tap with bridge

    - `sudo brctl addif virbr0 tap0` 

- configure

    - `sudo ifconfig virbr0 192.168.122.1` 
    - `sudo ifconfig tap0 192.168.122.100 `

- result

    - `brctl show `

    ```c
    $ btctl show
    bridge name	bridge id		STP enabled	interfaces
    virbr0		8000.52540011a6ac	yes		tap0
    ```

## run QEMU with tap

- `-net nic,model=rtl8139` 
- `-net tap,ifname=tap0,script=no ` 

- shutdown the firewall

## theory

```c
               +---------+---------+
               |   Physical Eth    |                           real hardware
               +---------+---------+
                        |                               ---------------------------
                 +------+------+
                 |  Net Driver |
                 +------+------+
                        |
                 +------+------+
                 | Host System |
                 +------+------+
                        |
                  +-----+-----+                                  host linux
                  |  Virtual  |
                  |  Bridge   |
                  +--+-----+--+
                     |     |
           +---------+     +---------+
           |                         |
+----------+----------+   +----------+----------+
|  Virtual Eth (tap0) |   |  Virtual Eth (tap1) |
+----------+----------+   +----------+----------+
           |                         |                   ---------------------------
+----------+----------+   +----------+----------+
|   Simluated Eth0    |   |   Simulated Eth1    |             simulated hardware
+----------+----------+   +----------+----------+              by QEMU
           |                         |                   ----------------------------
   +-------+-------+         +-------+-------+
   |   Net Driver  |         |   Net Driver  |
   +-------+-------+         +-------+-------+
           |                         |                           guest linux
   +-------+-------+         +-------+-------+
   | Guest System1 |         | Guest System2 |
   +---------------+         +---------------+
```

