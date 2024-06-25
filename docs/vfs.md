# Virtual Interfaces (VFs)

When loading a `P4` program, multiple Virtual Interfaces/Functions (VFs) are created (See [Changing the number of VFs](#changing-the-number-of-vfs)). These are usually called `vf0_<n>`, where `n` is the index. VFs work just like normal interfaces, and it is possible to assign them IP addresses.


In the configuration files, these interfaces are addressed as `v0.<n>`, while physical interfaces are named `p<n>`. See [config.p4cfg](https://github.com/RuiCunhaM/Template-Netronome-P4/blob/master/configs/config.p4cfg) for an example of basic forwarding between virtual and physical interfaces, allowing for external communication.

In case of needing to route a packet to a virtual interface without getting its value from the `p4cfg` file, these values are static and `v0.0` corresponds to `0x300`,`v0.1` to `0x301` and so on.

The same philosophy applies to physical interfaces. `p0` corresponds to `0x0`, `p1` to `0x1` and so on. 

!!! warning
    When sending network traffic between two or more Netronome SmartNICs keep in mind that VFs with the same name but in different hosts still **have the same MAC address!** Therefore, you should avoid exchanging traffic between VFs with the same name to avoid routing conflicts.

---

## Using the SmartNIC as a router/switch

Sometimes it can be useful to send network traffic between local VFs, effectively using the SmartNIC as a router/switch. To do this, you need to utilize Linux namespaces, otherwise traffic is sent through the `loopback` interface without never touching the NIC. Each namespace will have their own isolated routing tables, this way, by binding one or more VFs to a namespace they become "isolated" from the host. To use those interfaces you then need to run the intended programs also from within the namespace.

### Adding a VF to a namespace

1. Create a namespace

    ```bash
    ip netns add <name space>
    ```

2. Bind a VF to a namespace
    
    ```bash
    ip link set <interface> netns <name space>
    ```

3. Assign an IP to the interface and set the interface up

    ```bash
    ip netns exec <name space> ip addr add dev <interface> <ip address> 
    ip netns exec <name space> ip link set dev <interface> up
    ```

4. Disable checksum offload for the interface

    ```bash
    ip netns exec <name space> ethtool --offload <interface> rx off tx off
    ```

5. Run a program within a name space

    ```bash
    ip netns exec <name space> <program> <program args>
    ```

### Example usage

The following example demonstrates a simple client <-> server scenario.

!!! note
    This example assumes you have configured traffic forwarding/routing between `vf0_0` and `vf0_1`. 

1. Create the namespaces

    ```bash
    ip netns add ns_server
    ip netns add ns_client
    ```

2. Bind an interface to each namespace

    ```bash
    ip link set vf0_0 netns ns_server
    ip link set vf0_1 netns ns_client
    ```

3. Assign IPs and disable checksums

    ```bash
    ip netns exec ns_server ip addr add dev vf0_0 10.0.0.1/24
    ip netns exec ns_server ip link set dev vf0_0 up
    ip netns exec ns_server ethtool --offload vf0_0 rx off tx off

    ip netns exec ns_client ip addr add dev vf0_1 10.0.0.2/24
    ip netns exec ns_client ip link set dev vf0_1 up
    ip netns exec ns_client ethtool --offload vf0_1 rx off tx off
    ```

4. Run `iperf3` between client and server

    ```bash
    ip netns exec ns_server iperf3 -s
    ip netns exec ns_client iperf3 -c 10.0.0.1
    ```

!!! important
    Depending on your test configuration, type of traffic, number of VFs in each namespace, etc... You may be required to configure routing/forwarding rules inside each namespace using `ip netns exec <namespace>`.

---

## Changing the number of VFs

By default, `4` VFs are created when a `P4` program is loaded. A single NIC supports up to `64` different VFs. To change the number of VFs, edit the value of `NUM_VFS` at `/usr/lib/systemd/system/nfp-sdk6-rte.service`:

Example:
```toml
Environment=NUM_VFS=10
```

After applying the changes restart the `RTE` service.

