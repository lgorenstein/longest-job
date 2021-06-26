# longest-job

A tool to show when busy cluster nodes could become free (i.e. when the last
running Slurm job on each node exits).

By default, reports such high-water mark jobs on every cluster node that
has at least one active job on it (calls `squeue --states=R` under the hood).
A handful of familiar squeue options can be used to further limit
reporting to certain nodes (by partition, by account, by QoS, etc).
Alternatively, explicit Slurm-style nodelists can be passed as arguments.

## Usage:
```
    longest-job [-h|--help] [OPTIONS] [nodelist(s)]
```

**General options:**
```
  -t, --time     Sort output by job end time (default is to sort by node name).
      --quiet    Be really quiet (suppress most of non-essential output).
  -V, --version  Print program version and exit.
  -h, --help     Display this help message and exit.
```

**Recognized `squeue`-style filtering options** (passed to the underlying squeue
call verbatim, see `man squeue` for further details):
```
  -A, --account=<account_list>
  -j, --jobs=<job_id_list>
  -L, --licenses=<license_list>
  -M, --clusters=<clusters_list>
  -n, --name=<name_list>
  -p, --partition=<part_list>
  -q, --qos=<qos_list>
  -R, --reservation=<reservation_name>
  -u, --user=<user_list>
  -w, --nodelist=<hostlist>
```

In true `squeue` fashion, multiple filters are AND-ed.  If both `-w hostlist`
flag and explicit node names (`$1`...`$n`) are given, explicit ones win.

As a special case, nodelists in `$1`...`$n` can be absolute paths.
Cue this excerpt from `man scontrol`:
   > `scontrol show hostlist` can also take the absolute pathname of a file
   > (beginning with the character '/') containing a list of hostnames.


## Example
By node in a given partition:
```
$ longest-job --partition bell-b | head -3
NODE          JobID         END_TIME
bell-b000     3640405       2021-06-26T11:52:00
bell-b001     3606031       2021-06-25T21:48:04
bell-b002     3598003       2021-06-26T06:34:30
```
Or by earliest time:
```
$ longest-job --partition bell-b -t | head -3
NODE          JobID         END_TIME
bell-b001     3606031       2021-06-25T21:48:04
bell-b004     3606035       2021-06-25T21:52:04
bell-b002     3598003       2021-06-26T06:34:30
```
