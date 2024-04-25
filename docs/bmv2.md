# Migrating from bmv2

This page highlights some differences from the behavioral model (bmv2).

## Functions/Actions
- `mark_to_drop()` does **not** receive any argument

## Field Sizes
- `egress_spec` is 16 bits long

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
