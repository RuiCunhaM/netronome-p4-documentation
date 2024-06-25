# Migrating from bmv2

This page highlights some differences from the behavioral model (bmv2).

## Functions/Actions
- `mark_to_drop()` does **not** receive any argument

## Field Sizes
- `egress_spec` is 16 bits long 

## Field Values
- `standard_metadata.instance_type`

| Value | Meaning     |
|-------|-------------|
| 0x0   | Normal      |
| 0x1   | Clone I2I   |
| 0x2   | Clone E2I   |
| 0x3   | Recirculate |
| 0x4   | Resubmit    |
| 0x8   | Clone I2E   |
| 0x9   | Clone E2E   |
| 0xa   | Multicast   |

## Timestamps
- Time stamps are 64 bits long
- Contrary to Bmv2, the `ingress_global_timestamp` is not available through `standard_metadata`. If you need to use this field you need to:

<break>

1. Declare an intrinsic metadata header:
    
    ```c
    header intrinsic_metadata_t { bit<64> ingress_global_timestamp; }
    ```
  
2. Add it to `metadata_t`:
    
    ```c
    struct metadata_t {
        intrinsic_metadata_t intrinsic_metadata;
    }
    ```
## Known Issues

- Some issues were encountered with `digest messages`. In some cases, the message format was not allowed on Netronome, requiring the `struct` to be changed.
