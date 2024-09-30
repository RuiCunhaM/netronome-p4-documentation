# Assets

## Development Tools
To develop and load programs into the SmartNICs you require at least one of the following tool packages. The **Hosted Tool Chain** is a set of command line tools to compile and load programs. The **SDK** is Graphic IDE with multiple debug capabilities (e.g. Breakpoints). 

### Hosted Tool Chain (Linux)
- [Download Link (.deb package)](https://cloud63/assets/netronome/nfp-sdk.deb)


**Non Debian systems:**

1. Extract the package contents: 
        
    ```
    mkdir tmp
    dpkg-deb -R nfp-sdk.deb tmp
    ```

2. Copy the contents to your `opt`
    
    ```
    cp -r tmp/opt/netronome /opt/
    ```

3. Add `/opt/netronome/p4/bin` to your PATH

4. Test by running:

    ```
    nfp4build
    ```

### SDK 
This is a **Windows** program. Either run Windows from a VM or use a compatibility tool (e.g. [Bottles](https://usebottles.com/)).

- [SDK Setup (.exe)](https://cloud63/assets/netronome/sdk.exe)

---

## Documentation
After installing at least one of the development tools packages you should have access to all the documentation available: 

- If you installed the Hosted Tool Chain look into `/opt/netronome/doc`.
- If you're using the SDK, the documentation is available from within the IDE.

[Alternative download (.zip)](https://cloud63/assets/netronome/docs.zip)
