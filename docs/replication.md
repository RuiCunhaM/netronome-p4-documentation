# Replication

This page presents some useful information about packet replication.

---

## General problems in Netronome

!!! warning
    Keep in mind since the `egress_spec` is used to keep track of the replication operations, only one of these operations can be used per Ingress|Egress iteration.

---

## Multicast

Creating multicast groups is very simple, just needing to add 
```
"<GROUP_NAME>": {
    "group_id": <GROUP_ID>,
    "ports": [
        "v0.0",
        "v0.1",
        ...
    ]
}
```
to the `multicast` section of the `p4cfg` file.

!!! warning
    The multicast `GROUP_NAME` must always start with `mg`. While the `GROUP_ID` must be an integer in between 0 and the maximum number of multicast groups. In my personal suggestion, make the `GROUP_NAME` be `mg<GROUP_ID>` since this name is merely decorative. 
