# Tips and Tricks

This page presents some useful tips and tricks. It also compiles some `scripts` to help debugging P4 programs.

## No action usage
---

Do not use `NoAction` outside of tables. In Netronome SmartNICs this will cause an error while loading: `design_data.c:262 table ingress::tbl_NoAction has no allowed actions`.

---

## Casting variable to a lower number of bits
---

When casting a variable to a lower number of bits ensure that it won't cause an overflow. This will result in Undefined Behaviour, leading to the machine crashing and possibly the SmartNIC to overheat.

---

## Maximum register size
---
The maximum number of values in a register is `2,147,483,647` a.k.a. maximum signed integer value.

---

## Validating headers

Even though Netronome's Documentation specifies that the validity of a header can be checked in a table using the match type `header_valid`, this does not appear to work properly. The solution is to use `if` conditions instead. 

---

## Using variables/metadata as keys in tables.

When converted to `micro_c`, P4 variables and metadata are stored inside the scalars structure. When referencing these properties in the `p4cfg` file, they must be addressed in the following manner: 

- `scalars.<VariableName>`
- `scalars.metadata@<MetadataField>`

!!! warning
    Sometimes the compiler inserts `Ingress::` or `Egress::` before the `scalars.<VariableName>` depending if it is used in the Ingress or in the Egress.
    To check if this prefix is needed you can check the `pifs/pif_debug.json` and search for `Ingress::scalars`, if it is defined, then it is the correct one to use.
    
---

## Cloning and Multicasting simultaneously

In case you need to multicast a packet and also send it to another interface not on the multicast group, you can not do both simultaneously.
As a workaround, first, you need to clone the packet to the Ingress and forward the original one to the inerface outside the multicast group.
Then, in the Ingress, check for cloned packets, if true, multicast it.

---

## Debugging Locks

This `script` is meant to be used before compiling a program. It will create a new set of registers that reflect the existent locks in the original program. Those registers can later be inspected to help identify acquisitions and releases of locks.

```python
import json
import sys
import os
import re

if len(sys.argv) !=6:
    print("usage locksRegisterCompiler <.c input file> <.p4cfg input file> <.c output file> <.p4cfg output file> <.p4 file>")

with open(sys.argv[5]) as f: p4FileData = f.read()
with open(sys.argv[1]) as f: cFileData = f.read()
with open(sys.argv[2]) as f: p4cfgData = json.load(f)

for lock,numberOfLocks in re.findall(r"__declspec\(emem export aligned\(64\)\) int ([^\[;\s]+)(?:\[(.+)\])?\s*;",cFileData):
    if numberOfLocks=='':
        numberOfLocks=1
    else:
        if not numberOfLocks.isnumeric():
            r = re.findall(r"#define\s+"+numberOfLocks+r"\s+(\d+)",cFileData)[0]
            numberOfLocks=r
        numberOfLocks=int(numberOfLocks)
    p4cfgData["registers"]["configs"].append(
        {
            "count": numberOfLocks,
            "index": 0,
            "register": lock,
            "name": lock,
            "value": "0"
        }
    )
    release=lambda x:x.group(0)+(f"pif_register_{lock}[0].value=0;" if x.group(1)==None else f"pif_register_{lock}[{x.group(1)}].value=0;")
    acquire=lambda x:x.group(0)+(f"pif_register_{lock}[0].value=1;" if x.group(1)==None else f"pif_register_{lock}[{x.group(1)}].value=1;")

    if not re.search(r"release\(\s*\&"+lock+r"(?:\[(.+)\])?\s*\)\s*;"+f"pif_register_{lock}"):
        cFileData = re.sub(
            r"release\(\s*\&"+lock+r"(?:\[(.+)\])?\s*\)\s*;",
            release,
            cFileData)
    if not re.search(r"acquire\(\s*\&"+lock+r"(?:\[(.+)\])?\s*\)\s*;"+f"pif_register_{lock}"):
        cFileData = re.sub(
            r"acquire\(\s*\&"+lock+r"(?:\[(.+)\])?\s*\)\s*;",
            acquire,
            cFileData)
    if not re.search(f"register<bit<1>>({numberOfLocks}) {lock};"):
        p4FileData,_ = re.subn(
            "#include <v1model.p4>",
            f"#include <v1model.p4>\nregister<bit<1>>({numberOfLocks}) {lock};",
            p4FileData,1
        )

with open(sys.argv[3],"w") as f: f.write(cFileData)
with open(sys.argv[4],"w") as f: json.dump(p4cfgData,f,indent=4)
with open(sys.argv[5],"w") as f: f.write(p4FileData)
```
