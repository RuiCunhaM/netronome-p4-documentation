# Tips and Tricks

This page presents some useful tips and tricks. It also compiles some `scripts` to help debugging P4 programs.

---

## Debugging Locks (by [Luís Pereira](https://github.com/lumafepe))

This `script` is meant to be used after compiling a program. It will create a new set of registers that reflect the existent locks in the original program. Those register can later be inspected to help identify acquisitions and releases of locks.

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
