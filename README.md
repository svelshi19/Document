# Persistency Configuration and Verification

## CONTENTS OF THIS FILE

 * Introduction
 * Requirements
 * Configuration
 * Installation
 * Verification
 * Issues encountered
 * Reference
 
## Introduction

Persistence database is used to store data to non-volatile storage. This database is important as the data needs to be available even when the power of the device is switched off. This document describes the steps that are necessary for installation and how to validate the correctness of the storage.
Persistent database systems cache frequently requested data in memory for faster access, but write database updates, insertions and deletes through the cache to be stored to disk.
Database will be no-sql key value pair. The schema will be defined in "Types/VConfig".

## Requirements
Below list of libraries are required on the system for the configuration:

	1. Boost
	2. Sqlite3
	3. aracom_helper

## Configuration
Configuration need to be defined in "Types/VConfig" and values can be added/deleted in ConfigKey.json.
Build the repository again from root when json is modified.

```bash
# regenerate from root folder
./build.sh
```

List of new files/ changes to be configured:
Changes required to read value from Persistency by key:
1. Integrate Persistency
	1. persistency_proto/CMakeLists.txt. Can be used as a template.  
2. Configure Persistency
    1. model/PersistencyManager.ecuconfig. Can be used as a template.
    2. model/PersistencyManager.paramdef.arxml. Can be used as a template.
3. Configure key:value
   1. model/service_pm.arxml.
   key - ConfigKey
   value - bzghTFGgbvkdD. Value will be generated when ./gen_value.sh is executed. 
   protoserial.py is called from gen_value.sh to generate serialized structure(value) described into struct "Types/VConfig".
    ```xml
        <KEY-VALUE-PAIRS>
            <PERSISTENCY-KEY-VALUE-PAIR>
                <SHORT-NAME>ConfigKey</SHORT-NAME>
                <INIT-VALUE>
                    <NUMERICAL-VALUE-SPECIFICATION>
                    <SHORT-LABEL>bzghTFGgbvkdD</SHORT-LABEL>
                    </NUMERICAL-VALUE-SPECIFICATION>
                </INIT-VALUE>
                <UPDATE-STRATEGY>KEEP-EXISTING</UPDATE-STRATEGY>
                <VALUE-DATA-TYPE-REF DEST="STD-CPP-IMPLEMENTATION-DATA-TYPE">/ara/impltypes/String</VALUE-DATA-TYPE-REF>
            </PERSISTENCY-KEY-VALUE-PAIR>
        </KEY-VALUE-PAIRS>
	```
   2. model/vndls_service_pm_db.arxml. Can be used as a template.
   The contents of this file are populated when script ./gen_value.sh is executed.
4. Define interfaces and method to read from persistency:
	 persistency_proto/inc/ServiceConfig.hpp
	 persistency_proto/src/ServiceConfig.cpp
5. project_config.json
   Necessary modification: Update destination location to copy configuration files required by method: KeyValueStorage::OpenKeyValueStorage()


## Installation
Build.sh executes below steps:
1. Set env. This commands add 'protoc' to be used for compilation:
    ```bash
    	cd script/
    	source ~/ara/<..........>/environment-setup-linux
    ```
2. Generate Persistency INIT-VALUE for key ConfigKey. Output: service_pm_db.arxml
   ```bash
	PM_ARXML="$PWD/<...>/service_pm_db.arxml"
	KEY_VALUE_PAIR_SERVICE_CONFIG="/EB/PM/PersistencyDeployment/kvs_demo_database,$PWD/service/persistency_proto/ConfigKey.json"
	OUT_PM_ARXML=$PM_ARXML
	./gen_value.sh $FRANCA $PROTOPY $PM_ARXML $KEY_VALUE_PAIR_SERVICE_CONFIG $OUT_PM_ARXML
   ```
3. Generate pb.cc/pb.h files from 'protopy' proto/com/ford/fnv4/vsp/os/data/Types/*.proto.
   Output directory: vndlsservice-tcp/vndls_service/persistency_proto/generated
    ```bash
	PROTO="$PWD/build/proto/com/<........>/Types"
	IMPORT_PROTO="$PWD/build/proto"
	OUT_PROTOCPP="$PWD/service/persistency_proto/generated"
	mkdir -p $OUT_PROTOCPP
	./gen_pb.sh $FRANCA $PROTO $IMPORT_PROTO $OUT_PROTOCPP
    ```
4. Build the system again.

## Verification
1. Install and run log-viewer.
2. Configure the unit with configuration that includes: Hostname: 192.168.7.797, Port: 3422
3. Run the service in container
	```bash
		start_container.sh 1
		./deploy.sh
		start_service.sh
	```
4. Search for config info in the logs.

## Issues encountered
1. Version update: Invalid executable version configuration. Expected pattern: MajorVersion.MinorVersion.PatchVersion.BuildVersion with numeric version entries
   Solution: app.arxml. Update the version to:
   ```xml
	<VERSION>1.0.0.0</VERSION>
   ```

## Reference:
What is Persistency?
[Persistency](https://www.mcobject.com/what-is-a-persistent-database-and-how-is-it-different/)

How to configure/modeling Persistency?
Page 633. A.6 Definition of Persistent Data
[Specification of Manifest](https://www.autosar.org/fileadmin/user_upload/standards/adaptive/19-11/AUTOSAR_TPS_ManifestSpecification.pdf)



