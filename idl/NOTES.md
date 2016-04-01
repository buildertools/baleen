# Baleen IDL Notes

The Baleen IDL is used by image authors to define interfaces exposed by running containers created from implementing images.

## The Property Set

Container integration points are tightly coupled to the namespaces that protect them.

* *MNT* - The mount namespace is peirced by shared volumes.
* *NET* - A single network namespace can be shared by multiple containers. In such cases the processes in each container can communicate privately on the same loopback interface and serve endpoints on the same veth device.
* *IPC* - This namespace provides shared memory tools like POSIX queues, semaphores, and SysV shared memory blocks. Each of these tools are "named." As long as all participating processes agree on the type and name of the shared memory IPC tool (and have the same namespace) they can communicate.
* *USR* - Sharing a user namespace can be useful. Consider a database that requires a specific user. Rather than creating the user itself it can declare a dependency on a container with configured users. That dependency might define several such users and include other user-related configuration like key material, shell configurations, or other metadata.
* *UTS* - Can't think of a dependency or composition need. Not included
* *PID* - Please help me think of a relevant use-case.

### What could the IDL look like? 

A challenge for this project is to define interfaces where none have previously existed. The most interesting interface is for the file system and composing volumes. A mount point has a few interesting properties. A composer or consumer needs to know where the volume is mounted, the file permissions at that point (RWX), the owner and group ID, and an abstract idea about what is contained there.

The path hints a bit at the contents of the volume, but there are sure to be collisions. Adding an additional type field should be enough to namespace/qualify the purpose of the volume. That type field can be freeform.

OID/GID are important to navigate file permissions across user namespaced containers.

File permissions are obviously important. A consumer with a dependency on a file or mount point typically knows if it needs to be able to read/write/execute the wired file. 

#### JSON?

    {
        "schema":"baleen://_____",
        "name":"SomeInterface",
        "namespace":"buildertools",
        "properties": {
            "mnt":[
                { 
                    "path":"/var/secrets", 
                    "type":"pollendina-store",
                    "oid":1000,
                    "gid":200,
                    "flags":"600"
                },
                { 
                    "path":"/data",
                    "type":"mysql-data-dir",
                    "oid":1000,
                    "gid":200,
                    "flags":"600"
                },
                { 
                    "path":"/etc/nginx.conf", 
                    "type":"nginx-configuration",
                    "oid":1000,
                    "gid":200,
                    "flags":"644"
                },
                { 
                    "path":"/usr/bin/someTool", 
                    "type":"binary",
                    "oid":1,
                    "gid":200,
                    "flags":"755"
                }
            ],
            "net":[
                "tcp://localhost:3306",
                "udp://localhost:55556"
            ],
            "ipc":[
                "pq://queue_name",
                "sem://semaphore_name",
                "blk://shm_block_name"
            ]
        }
    }

