# Namespace plugin

The namespace plugin is an auxiliary plugin used mainly by other Linux plugins to handle namespaces and microservices.

* [Namespaces](#ns)
* [Microservices](#ms)
* [Model](#model)

### <a name="ns">Namespaces</a>

The agent has full support for Linux network namespaces. It is possible to attach Linux interface/ARP/route into a new, existing or even yet-to-be-created network namespace via the `Namespace` configuration section inside data model.

Namespace can be referenced in multiple ways. The most low-level link to a namespace is a file descriptor associated with the symbolic link automatically created in the `proc` filesystem, pointing to the definition of the namespace used by a given process (`/proc/<PID>/ns/net`) or by a task of a given process (`/proc/<PID>/task/<TID>/ns/net`). A more common approach to reference namespace is to use just the PID of the process whose namespace we want to attach to, or to create a bind-mount of the symbolic link into `/var/run/netns` directory and use the filename of that mount. The latter is called `named` namespace and it is created and managed for example by the `ip netns` command line tool from the `iproute2` package. The advantage of `named` namespace is that it can outlive the process it was originally created by.

A `namespace` configuration section should be seen as a union of values. First, set the type and then store the reference into the appropriate field (`pid` vs. `name` vs `microservice`). The agent supports both PID-based references as well as `named` namespaces.

### <a name="ms">Microservices</a>

Additionally, we provide a non-standard namespace reference, denoted as `MICROSERVICE_REF_NS`, which is specific to ecosystems with microservices. It is possible to attach interface/ARP/route into the namespace of a container that runs microservice with a given label. To make it even simpler, it is not required to start the microservice before the configured item is pushed. The agent will postpone interface (re)configuration until the referenced microservice gets launched. Behind the scenes, the agent communicates with the docker daemon to construct and maintain an up-to-date map of microservice labels to PIDs and IDs of their corresponding containers. Whenever a new microservice is detected, all pending interfaces are moved to its namespace.

### <a name="model">Model</a>

The namespace [model](https://github.com/ligato/vpp-agent/blob/master/api/models/linux/namespace/namespace.proto) contains two fields - the type, and the reference.
The namespace can be of type named, defined by a process ID, referenced by a file handle or as a docker container running some microservice. The reference is a namespace identifier (namespace ID, specific PID, file path or a microservice label).

The namespace is imported in the [Linux interface plugin](Linux-Interface-plugin), by itself does not define any keys (except notifications). 
