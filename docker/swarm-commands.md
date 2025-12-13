# Docker Swarm Cheat Sheet

This cheat sheet provides a quick reference to common Docker Swarm commands.

## Swarm Management

### `docker swarm init`
Initialize a swarm.

```
Usage:  docker swarm init [OPTIONS]

Options:
      --advertise-addr string                  Advertised address (format: "<ip|interface>[:port]")
      --autolock                               Enable manager autolocking (requiring an unlock key to start a stopped manager)
      --availability string                    Availability of the node ("active", "pause", "drain") (default "active")
      --cert-expiry duration                   Validity period for node certificates (ns|us|ms|s|m|h) (default 2160h0m0s)
      --data-path-addr string                  Address or interface to use for data path traffic (format: "<ip|interface>")
      --data-path-port uint32                  Port number to use for data path traffic (1024 - 49151). If no value is set or is set to 0, the default port (4789) is used.
      --default-addr-pool ipNetSlice           default address pool in CIDR format (default [])
      --default-addr-pool-mask-length uint32   default address pool subnet mask length (default 24)
      --dispatcher-heartbeat duration          Dispatcher heartbeat period (ns|us|ms|s|m|h) (default 5s)
      --external-ca external-ca                Specifications of one or more certificate signing endpoints
      --force-new-cluster                      Force create a new cluster from current state
      --listen-addr node-addr                  Listen address (format: "<ip|interface>[:port]") (default 0.0.0.0:2377)
      --max-snapshots uint                     Number of additional Raft snapshots to retain
      --snapshot-interval uint                 Number of log entries between Raft snapshots (default 10000)
      --task-history-limit int                 Task history retention limit (default 5)
```

### `docker swarm join`
Join a swarm as a node and/or manager.

```
Usage:  docker swarm join [OPTIONS] HOST:PORT

Options:
      --advertise-addr string   Advertised address (format: "<ip|interface>[:port]")
      --availability string     Availability of the node ("active", "pause", "drain") (default "active")
      --data-path-addr string   Address or interface to use for data path traffic (format: "<ip|interface>")
      --listen-addr node-addr   Listen address (format: "<ip|interface>[:port]") (default 0.0.0.0:2377)
      --token string            Token for entry into the swarm
```

### `docker swarm join-token`
Manage join tokens.

```
Usage:  docker swarm join-token [OPTIONS] (worker|manager)

Options:
  -q, --quiet    Only display token
      --rotate   Rotate join token
```

### `docker swarm leave`
Leave the swarm.

```
Usage:  docker swarm leave [OPTIONS]

Options:
  -f, --force   Force this node to leave the swarm, ignoring warnings
```

### `docker swarm update`
Update the swarm.

```
Usage:  docker swarm update [OPTIONS]

Options:
      --autolock                        Change manager autolocking setting (true|false)
      --cert-expiry duration            Validity period for node certificates (ns|us|ms|s|m|h) (default 2160h0m0s)
      --dispatcher-heartbeat duration   Dispatcher heartbeat period (ns|us|ms|s|m|h) (default 5s)
      --external-ca external-ca         Specifications of one or more certificate signing endpoints
      --max-snapshots uint              Number of additional Raft snapshots to retain
      --snapshot-interval uint          Number of log entries between Raft snapshots (default 10000)
      --task-history-limit int          Task history retention limit (default 5)
```

### `docker swarm ca`
Display and rotate the root CA.

```
Usage:  docker swarm ca [OPTIONS]

Options:
      --ca-cert pem-file          Path to the PEM-formatted root CA certificate to use for the new cluster
      --ca-key pem-file           Path to the PEM-formatted root CA key to use for the new cluster
      --cert-expiry duration      Validity period for node certificates (ns|us|ms|s|m|h) (default 2160h0m0s)
  -d, --detach                    Exit immediately instead of waiting for the root rotation to converge
      --external-ca external-ca   Specifications of one or more certificate signing endpoints
  -q, --quiet                     Suppress progress output
      --rotate                    Rotate the swarm CA - if no certificate or key are provided, new ones will be generated
```

### `docker swarm unlock`
Unlock swarm.

```
Usage:  docker swarm unlock
```

### `docker swarm unlock-key`
Manage the unlock key.

```
Usage:  docker swarm unlock-key [OPTIONS]

Options:
  -q, --quiet    Only display token
      --rotate   Rotate unlock key
```

## Node Management

### `docker node demote`
Demote one or more nodes from manager in the swarm.

```
Usage:  docker node demote NODE [NODE...]
```

### `docker node inspect`
Display detailed information on one or more nodes.

```
Usage:  docker node inspect [OPTIONS] self|NODE [NODE...]

Options:
  -f, --format string   Format output using a custom template:
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
      --pretty          Print the information in a human friendly format
```

### `docker node ls`
List nodes in the swarm.

```
Usage:  docker node ls [OPTIONS]

Aliases:
  docker node ls, docker node list

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Format output using a custom template:
                        'table':            Print output in table format with column headers (default)
                        'table TEMPLATE':   Print output in table format using the given Go template
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
  -q, --quiet           Only display IDs
```

### `docker node promote`
Promote one or more nodes to manager in the swarm.

```
Usage:  docker node promote NODE [NODE...]
```

### `docker node ps`
List tasks running on one or more nodes, defaults to current node.

```
Usage:  docker node ps [OPTIONS] [NODE...]

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print tasks using a Go template
      --no-resolve      Do not map IDs to Names
      --no-trunc        Do not truncate output
  -q, --quiet           Only display task IDs
```

### `docker node rm`
Remove one or more nodes from the swarm.

```
Usage:  docker node rm [OPTIONS] NODE [NODE...]

Aliases:
  docker node rm, docker node remove

Options:
  -f, --force   Force remove a node from the swarm
```

### `docker node update`
Update a node.

```
Usage:  docker node update [OPTIONS] NODE

Options:
      --availability string   Availability of the node ("active", "pause", "drain")
      --label-add list        Add or update a node label ("key=value")
      --label-rm list         Remove a node label if exists
      --role string           Role of the node ("worker", "manager")
```

## Service Management

### `docker service create`
Create a new service.

```
Usage:  docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]

Options:
      --cap-add list                       Add Linux capabilities
      --cap-drop list                      Drop Linux capabilities
      --config config                      Specify configurations to expose to the service
      --constraint list                    Placement constraints
      --container-label list               Container labels
      --credential-spec credential-spec    Credential spec for managed service account (Windows only)
  -d, --detach                             Exit immediately instead of waiting for the service to converge
      --dns list                           Set custom DNS servers
      --dns-option list                    Set DNS options
      --dns-search list                    Set custom DNS search domains
      --endpoint-mode string               Endpoint mode (vip or dnsrr) (default "vip")
      --entrypoint command                 Overwrite the default ENTRYPOINT of the image
  -e, --env list                           Set environment variables
      --env-file list                      Read in a file of environment variables
      --generic-resource list              User defined resources
      --group list                         Set one or more supplementary user groups for the container
      --health-cmd string                  Command to run to check health
      --health-interval duration           Time between running the check (ms|s|m|h)
      --health-retries int                 Consecutive failures needed to report unhealthy
      --health-start-interval duration     Time between running the check during the start period (ms|s|m|h)
      --health-start-period duration       Start period for the container to initialize before counting retries towards unstable (ms|s|m|h)
      --health-timeout duration            Maximum time to allow one check to run (ms|s|m|h)
      --host list                          Set one or more custom host-to-IP mappings (host:ip)
      --hostname string                    Container hostname
      --init                               Use an init inside each service container to forward signals and reap processes
      --isolation string                   Service container isolation mode
  -l, --label list                         Service labels
      --limit-cpu decimal                  Limit CPUs
      --limit-memory bytes                 Limit Memory
      --limit-pids int                     Limit maximum number of processes (default 0 = unlimited)
      --log-driver string                  Logging driver for service
      --log-opt list                       Logging driver options
      --max-concurrent uint                Number of job tasks to run concurrently (default equal to --replicas)
      --memory-swap bytes                  Swap Bytes (-1 for unlimited)
      --memory-swappiness int              Tune memory swappiness (0-100), -1 to reset to default (default -1)
      --mode string                        Service mode ("replicated", "global", "replicated-job", "global-job") (default "replicated")
      --mount mount                        Attach a filesystem mount to the service
      --name string                        Service name
      --network network                    Network attachments
      --no-healthcheck                     Disable any container-specified HEALTHCHECK
      --no-resolve-image                   Do not query the registry to resolve image digest and supported platforms
      --oom-score-adj int                  Tune host's OOM preferences (-1000 to 1000)
      --placement-pref pref                Add a placement preference
  -p, --publish port                       Publish a port as a node port
  -q, --quiet                              Suppress progress output
      --read-only                          Mount the container's root filesystem as read only
      --replicas uint                      Number of tasks
      --replicas-max-per-node uint         Maximum number of tasks per node (default 0 = unlimited)
      --reserve-cpu decimal                Reserve CPUs
      --reserve-memory bytes               Reserve Memory
      --restart-condition string           Restart when condition is met ("none", "on-failure", "any") (default "any")
      --restart-delay duration             Delay between restart attempts (ns|us|ms|s|m|h) (default 5s)
      --restart-max-attempts uint          Maximum number of restarts before giving up
      --restart-window duration            Window used to evaluate the restart policy (ns|us|ms|s|m|h)
      --rollback-delay duration            Delay between task rollbacks (ns|us|ms|s|m|h) (default 0s)
      --rollback-failure-action string     Action on rollback failure ("pause", "continue") (default "pause")
      --rollback-max-failure-ratio float   Failure rate to tolerate during a rollback (default 0)
      --rollback-monitor duration          Duration after each task rollback to monitor for failure (ns|us|ms|s|m|h) (default 5s)
      --rollback-order string              Rollback order ("start-first", "stop-first") (default "stop-first")
      --rollback-parallelism uint          Maximum number of tasks rolled back simultaneously (0 to roll back all at once) (default 1)
      --secret secret                      Specify secrets to expose to the service
      --stop-grace-period duration         Time to wait before force killing a container (ns|us|ms|s|m|h) (default 10s)
      --stop-signal string                 Signal to stop the container
      --sysctl list                        Sysctl options
  -t, --tty                                Allocate a pseudo-TTY
      --ulimit ulimit                      Ulimit options (default [])
      --update-delay duration              Delay between updates (ns|us|ms|s|m|h) (default 0s)
      --update-failure-action string       Action on update failure ("pause", "continue", "rollback") (default "pause")
      --update-max-failure-ratio float     Failure rate to tolerate during an update (default 0)
      --update-monitor duration            Duration after each task update to monitor for failure (ns|us|ms|s|m|h) (default 5s)
      --update-order string                Update order ("start-first", "stop-first") (default "stop-first")
      --update-parallelism uint            Maximum number of tasks updated simultaneously (0 to update all at once) (default 1)
  -u, --user string                        Username or UID (format: <name|uid>[:<group|gid>])
      --with-registry-auth                 Send registry authentication details to swarm agents
  -w, --workdir string                     Working directory inside the container
```

### `docker service inspect`
Display detailed information on one or more services.

```
Usage:  docker service inspect [OPTIONS] SERVICE [SERVICE...]

Options:
  -f, --format string   Format output using a custom template:
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
      --pretty          Print the information in a human friendly format
```

### `docker service logs`
Fetch the logs of a service or task.

```
Usage:  docker service logs [OPTIONS] SERVICE|TASK

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --no-resolve     Do not map IDs to Names in output
      --no-task-ids    Do not include task IDs in output
      --no-trunc       Do not truncate output
      --raw            Do not neatly format logs
      --since string   Show logs since timestamp (e.g. "2013-01-02T13:23:37Z") or relative (e.g. "42m" for 42 minutes)
  -n, --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
```

### `docker service ls`
List services.

```
Usage:  docker service ls [OPTIONS]

Aliases:
  docker service ls, docker service list

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Format output using a custom template:
                        'table':            Print output in table format with column headers (default)
                        'table TEMPLATE':   Print output in table format using the given Go template
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
  -q, --quiet           Only display IDs
```

### `docker service ps`
List the tasks of one or more services.

```
Usage:  docker service ps [OPTIONS] SERVICE [SERVICE...]

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print tasks using a Go template
      --no-resolve      Do not map IDs to Names
      --no-trunc        Do not truncate output
  -q, --quiet           Only display task IDs
```

### `docker service rm`
Remove one or more services.

```
Usage:  docker service rm SERVICE [SERVICE...]

Aliases:
  docker service rm, docker service remove
```

### `docker service rollback`
Revert changes to a service's configuration.

```
Usage:  docker service rollback [OPTIONS] SERVICE

Options:
  -d, --detach   Exit immediately instead of waiting for the service to converge
  -q, --quiet    Suppress progress output
```

### `docker service scale`
Scale one or multiple replicated services.

```
Usage:  docker service scale SERVICE=REPLICAS [SERVICE=REPLICAS...]

Options:
  -d, --detach   Exit immediately instead of waiting for the service to converge
```

### `docker service update`
Update a service.

```
Usage:  docker service update [OPTIONS] SERVICE

Options:
      --args command                       Service command args
      --cap-add list                       Add Linux capabilities
      --cap-drop list                      Drop Linux capabilities
      --config-add config                  Add or update a config file on a service
      --config-rm list                     Remove a configuration file
      --constraint-add list                Add or update a placement constraint
      --constraint-rm list                 Remove a constraint
      --container-label-add list           Add or update a container label
      --container-label-rm list            Remove a container label by its key
      --credential-spec credential-spec    Credential spec for managed service account (Windows only)
  -d, --detach                             Exit immediately instead of waiting for the service to converge
      --dns-add list                       Add or update a custom DNS server
      --dns-option-add list                Add or update a DNS option
      --dns-option-rm list                 Remove a DNS option
      --dns-rm list                        Remove a custom DNS server
      --dns-search-add list                Add or update a custom DNS search domain
      --dns-search-rm list                 Remove a DNS search domain
      --endpoint-mode string               Endpoint mode (vip or dnsrr)
      --entrypoint command                 Overwrite the default ENTRYPOINT of the image
      --env-add list                       Add or update an environment variable
      --env-rm list                        Remove an environment variable
      --force                              Force update even if no changes require it
      --generic-resource-add list          Add a Generic resource
      --generic-resource-rm list           Remove a Generic resource
      --group-add list                     Add an additional supplementary user group to the container
      --group-rm list                      Remove a previously added supplementary user group from the container
      --health-cmd string                  Command to run to check health
      --health-interval duration           Time between running the check (ms|s|m|h)
      --health-retries int                 Consecutive failures needed to report unhealthy
      --health-start-interval duration     Time between running the check during the start period (ms|s|m|h)
      --health-start-period duration       Start period for the container to initialize before counting retries towards unstable (ms|s|m|h)
      --health-timeout duration            Maximum time to allow one check to run (ms|s|m|h)
      --host-add list                      Add a custom host-to-IP mapping ("host:ip")
      --host-rm list                       Remove a custom host-to-IP mapping ("host:ip")
      --hostname string                    Container hostname
      --image string                       Service image tag
      --init                               Use an init inside each service container to forward signals and reap processes
      --isolation string                   Service container isolation mode
      --label-add list                     Add or update a service label
      --label-rm list                      Remove a label by its key
      --limit-cpu decimal                  Limit CPUs
      --limit-memory bytes                 Limit Memory
      --limit-pids int                     Limit maximum number of processes (default 0 = unlimited)
      --log-driver string                  Logging driver for service
      --log-opt list                       Logging driver options
      --max-concurrent uint                Number of job tasks to run concurrently (default equal to --replicas)
      --memory-swap bytes                  Swap Bytes (-1 for unlimited)
      --memory-swappiness int              Tune memory swappiness (0-100), -1 to reset to default (default -1)
      --mount-add mount                    Add or update a mount on a service
      --mount-rm list                      Remove a mount by its target path
      --network-add network                Add a network
      --network-rm list                    Remove a network
      --no-healthcheck                     Disable any container-specified HEALTHCHECK
      --no-resolve-image                   Do not query the registry to resolve image digest and supported platforms
      --oom-score-adj int                  Tune host's OOM preferences (-1000 to 1000)
      --placement-pref-add pref            Add a placement preference
      --placement-pref-rm pref             Remove a placement preference
      --publish-add port                   Add or update a published port
      --publish-rm port                    Remove a published port by its target port
  -q, --quiet                              Suppress progress output
      --read-only                          Mount the container's root filesystem as read only
      --replicas uint                      Number of tasks
      --replicas-max-per-node uint         Maximum number of tasks per node (default 0 = unlimited)
      --reserve-cpu decimal                Reserve CPUs
      --reserve-memory bytes               Reserve Memory
      --restart-condition string           Restart when condition is met ("none", "on-failure", "any")
      --restart-delay duration             Delay between restart attempts (ns|us|ms|s|m|h)
      --restart-max-attempts uint          Maximum number of restarts before giving up
      --restart-window duration            Window used to evaluate the restart policy (ns|us|ms|s|m|h)
      --rollback                           Rollback to previous specification
      --rollback-delay duration            Delay between task rollbacks (ns|us|ms|s|m|h)
      --rollback-failure-action string     Action on rollback failure ("pause", "continue")
      --rollback-max-failure-ratio float   Failure rate to tolerate during a rollback
      --rollback-monitor duration          Duration after each task rollback to monitor for failure (ns|us|ms|s|m|h)
      --rollback-order string              Rollback order ("start-first", "stop-first")
      --rollback-parallelism uint          Maximum number of tasks rolled back simultaneously (0 to roll back all at once)
      --secret-add secret                  Add or update a secret on a service
      --secret-rm list                     Remove a secret
      --stop-grace-period duration         Time to wait before force killing a container (ns|us|ms|s|m|h)
      --stop-signal string                 Signal to stop the container
      --sysctl-add list                    Add or update a Sysctl option
      --sysctl-rm list                     Remove a Sysctl option
  -t, --tty                                Allocate a pseudo-TTY
      --ulimit-add ulimit                  Add or update a ulimit option (default [])
      --ulimit-rm list                     Remove a ulimit option
      --update-delay duration              Delay between updates (ns|us|ms|s|m|h)
      --update-failure-action string       Action on update failure ("pause", "continue", "rollback")
      --update-max-failure-ratio float     Failure rate to tolerate during an update
      --update-monitor duration            Duration after each task update to monitor for failure (ns|us|ms|s|m|h)
      --update-order string                Update order ("start-first", "stop-first")
      --update-parallelism uint            Maximum number of tasks updated simultaneously (0 to update all at once)
  -u, --user string                        Username or UID (format: <name|uid>[:<group|gid>])
      --with-registry-auth                 Send registry authentication details to swarm agents
  -w, --workdir string                     Working directory inside the container
```

## Stack Management

### `docker stack config`
Outputs the final config file, after doing merges and interpolations.

```
Usage:  docker stack config [OPTIONS]

Options:
  -c, --compose-file strings   Path to a Compose file, or "-" to read from stdin
      --skip-interpolation     Skip interpolation and output only merged config
```

### `docker stack deploy`
Deploy a new stack or update an existing stack.

```
Usage:  docker stack deploy [OPTIONS] STACK

Aliases:
  docker stack deploy, docker stack up

Options:
  -c, --compose-file strings   Path to a Compose file, or "-" to read from stdin
  -d, --detach                 Exit immediately instead of waiting for the stack services to converge (default true)
      --prune                  Prune services that are no longer referenced
  -q, --quiet                  Suppress progress output
      --resolve-image string   Query the registry to resolve image digest and supported platforms ("always", "changed", "never") (default "always")
      --with-registry-auth     Send registry authentication details to Swarm agents
```

### `docker stack ls`
List stacks.

```
Usage:  docker stack ls [OPTIONS]

Aliases:
  docker stack ls, docker stack list

Options:
      --format string   Format output using a custom template:
                        'table':            Print output in table format with column headers (default)
                        'table TEMPLATE':   Print output in table format using the given Go template
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
```

### `docker stack ps`
List the tasks in the stack.

```
Usage:  docker stack ps [OPTIONS] STACK

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Format output using a custom template:
                        'table':            Print output in table format with column headers (default)
                        'table TEMPLATE':   Print output in table format using the given Go template
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
      --no-resolve      Do not map IDs to Names
      --no-trunc        Do not truncate output
  -q, --quiet           Only display task IDs
```

### `docker stack rm`
Remove one or more stacks.

```
Usage:  docker stack rm [OPTIONS] STACK [STACK...]

Aliases:
  docker stack rm, docker stack remove, docker stack down

Options:
  -d, --detach   Do not wait for stack removal (default true)
```

### `docker stack services`
List the services in the stack.

```
Usage:  docker stack services [OPTIONS] STACK

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Format output using a custom template:
                        'table':            Print output in table format with column headers (default)
                        'table TEMPLATE':   Print output in table format using the given Go template
                        'json':             Print in JSON format
                        'TEMPLATE':         Print output using the given Go template.
                        Refer to https://docs.docker.com/go/formatting/ for more information about formatting output with templates
  -q, --quiet           Only display IDs
```
