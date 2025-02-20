# nRF5x Firmware disassembly tools - Proof of Concept #

This repository contains a collection of tools to reverse NRF5x firmwares.

## How to install dependencies ##

```
pip install -r requirements.txt 
```

## NRF parsing and database population ##

The `nrfparse.py` python script must be run once to generate and populate the `nRF.db` SQLite3 database.
It extracts, from each Development Kit archive in the `developer.nordicsemi/nRF5_SDK` directory to disk in the `SDKs` directory, the different SoftDevices (s110, s210, ...etc) and their associated headers and linkers files and the firmware in its IntelHex format if contained in the archive.

The parsing then extracts for each SoftDevice version:
- Associated SDK version (14.0.0, ...)
- Associated NRF (nrf51422, nrf51822, ...)
- Associated RAM and FLASH addresses and sections length for binary mapping 
- Associated SVC_BASE and SVC_LAST ranges
- Associated SVCALLS' structures
- Associtade SVCALLS prototypes that will help nrfreverse using IDA python to define types of SVCALLS and rename functions and parameters

The data is then commited to the `nRF.db`.

## NRF identification given a .hex or .bin version of the firmware ##

The `nrfidentify.py` python script is run with two arguments: `bin` or `hex` and NRF firmware in its .bin or .hex format.
```
usage: nrfident.py [-h] {bin,hex} FILE

positional arguments:
  {bin,hex}   the object format bfdname
  FILE        input file to identify

optional arguments:
  -h, --help  show this help message and exit
```

The script computes the SD firmware's signature, then looks for it in the `nRF.db` database.

If not found, the strings in the binary can be used to try an identification, an approximate 
signature is then generated.

The following information are then extracted from the `nRF.db`:
- Associated SDK version
- Associated NRF
- Associated RAM and FLASH addresses and sections length for binary mapping in IDA pro

A file named `nRF_version` is then generated containing the firmware's signature.

This file is used with the `nRF.db` by `nrfreverse.py` that mainly uses IDApython to redefine types and rename SVCALL functions.
The `nRF.db` and `nRF_version` can be be copied to the  `%PROGRAMFILES%\IDA\python` folder.

```

python3 nrfident.py bin firmwares/s132.bin
############################ nRF5-tool ############################ 
#####                                                         ##### 
#####                Identifying nRF5x firmwares              ##### 
#####                                                         ##### 
################################################################### 


Binary file provided firmwares/s132.bin

Computing signature from binary
100%|██████████████████████████████████████████████████████████████████████| 
Signature:  d082a85351ee18ecfdc9dcb01352f5df3d938a2270bcadec2ec083e9ceeb3b1e
Searching for signature in nRF.db
100%|██████████████████████████████████████████████████████████████████████| 
=========================
SDK version:  14.0.0
SoftDevice version: s132
NRF: nrf52832
=========================
SDK version:  14.1.0
SoftDevice version: s132
NRF: nrf52832
                               ==================
nRF5x signature written to file nRF_ver in current directory
nRF_ver path must be provided when running nrfreverse.py from IDA

                                     *****
                                  Binary mapping
                                     *****

SoftDevice  :  s132
Card version :  xxaa
           *****
RAM address  :  0x20001368
RAM length   :  0xec98
ROM address  :  0x23000
ROM length   :  0x5d000
```


## NRF firmware reversing in IDA pro ##

The NRF firmware is mapped in IDA pro using associated FLASH and RAM addresses and lengths.

The `nrfreverse.py` python script must be in the same directory as the `nRF.db` database and the `nRF_ver` 
file that was generated by `nrfident.py`.

It parses the assembly code of the given SoftDevice firmware to find *SVC* opcodes.

It selects the appropriate functions' prototypes from the `nRF.db` and outputs them in the python output window of IDA pro.

### Further improvements ###

1. Automatically mapping the binary according to the RAM and FLASH addresses and length in IDA pro.

2. Recover classic libc functions based on their corresponding signatures

3. Moar testing !
