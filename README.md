## roachprod

⚠️ roachprod is an **internal** tool for testing CockroachDB clusters. Use at
your own risk! ⚠️

## Setup

Make sure you have [gcloud installed] and configured (`gcloud auth list` to
check, `gcloud auth login` to authenticate). You may want to update old
installations (`gcloud components update`).

To build and install into `$GOPATH/bin`:

```
$ go get -u github.com/cockroachlabs/roachprod
```

## Summary

* Clusters are created under the [cockroach-ephemeral] GCE project.
* Anyone can connect to any port on VMs in `cockroach-ephemeral`.
  **DO NOT STORE SENSITIVE DATA**.
* Cluster names are prefixed with the user creating them. For example,
  `roachprod create test` creates the `marc-test` cluster.
* VMs have a default lifetime of 12 hours (changeable with the `-l` flag) and
  are deleted 6 hours after expiration.
* Default settings create 4 VMs (`-n 4`) with 4 CPUs, 15GB memory
  (`--machine-type=n1-standard-4`), and local SSDs (`--local-ssd`).

## Cluster quick-start using roachprod/roachperf

```bash
# Create a cluster with 4 nodes and local SSD.
# The last node is used as load generator by roachperf.
export NAME="test"
export FULLNAME="${USER}-${NAME}"
roachprod create ${NAME} -n 4 --local-ssd

# Add gcloud SSH key.
ssh-add ~/.ssh/google_compute_engine

# Stage scripts and binaries using crl-prod...
crl-stage-binaries ${FULLNAME} all scripts
crl-stage-binaries ${FULLNAME} all cockroach

# ...or using roachperf (e.g., for your locally-built binary).
roachperf ${FULLNAME} put cockroach

# Start a cluster using roachperf.
roachperf ${FULLNAME} start

# Check the admin UI.
# http://35.196.94.196:8080

# Open a SQL connection to the first node.
cockroach sql --insecure --host=35.196.94.196

# Extend lifetime by another 6 hours.
roachprod extend ${NAME} --lifetime=6h

# Destroy the cluster.
roachprod destroy ${NAME}
```

## Command reference

Warning: this reference is incomplete. Be prepared to refer to the CLI help text
and the source code.

### Create a cluster
```
$ roachprod create foo
Creating cluster marc-foo with 3 nodes
OK
marc-foo: 23h59m42s remaining
  marc-foo-0000   [marc-foo-0000.us-east1-b.cockroach-ephemeral]
  marc-foo-0001   [marc-foo-0001.us-east1-b.cockroach-ephemeral]
  marc-foo-0002   [marc-foo-0002.us-east1-b.cockroach-ephemeral]
Syncing...
```

### Interact using crl-prod tools
`roachprod` populates hosts files in `~/.roachprod/hosts`. These are used by
`crl-prod` tools to map clusters to node addresses.

```
$ crl-ssh marc-foo all df -h /
1: marc-foo-0000.us-east1-b.cockroach-ephemeral
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        49G  1.2G   48G   3% /

2: marc-foo-0001.us-east1-b.cockroach-ephemeral
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        49G  1.2G   48G   3% /

3: marc-foo-0002.us-east1-b.cockroach-ephemeral
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        49G  1.2G   48G   3% /
```

### Interact using `roachperf`
[roachperf] also consumes `~/.roachprod/hosts`.

```
# Add ssh-key
$ ssh-add ~/.ssh/google_compute_engine

$ roachperf marc-foo status
marc-foo: status 3/3
   1: not running
   2: not running
   3: not running
```

### SSH into hosts
`roachprod` uses `gcloud` to sync the list of hostnames to `~/.ssh/config` and
set up keys.

```
$ ssh marc-foo-0000.us-east1-b.cockroach-ephemeral
```

### List clusters
```
$ roachprod list
marc-foo: 23h58m27s remaining
  marc-foo-0000
  marc-foo-0001
  marc-foo-0002
Syncing...
```

### Destroy cluster
```
$ roachprod destroy foo
Destroying cluster marc-foo with 3 nodes
OK
```

See `roachprod help <command>` for further details.


# Future improvements

* Bigger loadgen VM (last instance)

[cockroach-ephemeral]: https://console.cloud.google.com/home/dashboard?project=cockroach-ephemeral
[gcloud installed]: https://cloud.google.com/sdk/downloads
[roachperf]: https://github.com/cockroachdb/roachperf
