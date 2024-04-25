# Mac Egress CMD

Due to the internal architecture of the NFP-4000 processors, 4 bytes **are required** to be prepended on each packet exiting the NIC to pass packet modification instructions directly into the hardware. 

!!! important
    This only applies on physical egress. Which means that if you are relying on this mechanism to compute checksums it will only work for packets leaving the SmartNIC. Altering packets egressing torwards virtual interfaces, will require normal checksum update through `P4` control blocks.

---

### Usage
**If you're using the [template provided](https://github.com/RuiCunhaM/Template-Netronome-P4) this is already configured. It is recommended to leave it untouched**

1. Header declaration:
    
    ```c
    header nfp_mac_eg_cmd_t {
      bit en_l3_sum;  // Enables L3 checksum computation
      bit en_l4_sum;  // Enables L4 checksum computation
      bit en_ts_mark; // Enables Timestamp marking
      bit<29> ignore;
    }
    ```

2. Add it to headers struct (**Top position is mandatory!**)

    ```c
    struct headers_t {
      nfp_mac_eg_cmd_t nfp_mac_eg_cmd;
      // Other headers....
    }
    ```
3. Add an egress table

    ```c
    action add_empty_nfp_mac_eg_cmd() {
        hdr.nfp_mac_eg_cmd.setValid();
        hdr.nfp_mac_eg_cmd = {0, 1, 0, 0x0};
    }

    table table_add_empty_nfp_mac_eg_cmd {
        key = { 
            standard_metadata.egress_port : exact; 
        }
        
        actions = { 
            NoAction; // From core.p4
            add_empty_nfp_mac_eg_cmd;
        }

        default_action = NoAction;
        size = 2;
    }
    ```

4. Add the table configuration to the `json` file

    ```json
    "egress::table_add_empty_nfp_mac_eg_cmd": {
        "rules": [
            {
                "name": "p0",
                "match": {
                    "standard_metadata.egress_port": {
                        "value": "p0"
                    }
                },
                "action": {
                    "type": "egress::add_empty_nfp_mac_eg_cmd"
                }
            },
            {
                "name": "p2",
                "match": {
                    "standard_metadata.egress_port": {
                        "value": "p2"
                    }
                },
                "action": {
                    "type": "egress::add_empty_nfp_mac_eg_cmd"
                }
            }
        ]
    }
    ```

---

### Disabling it
In theory, it should be possible to disable the usage of this header. 

**This has not been tested yet!!**
