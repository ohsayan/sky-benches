# Skytable v/s the rest

> More comprehensive and thorough benchmarks will be published soon. Please wait till then! **[Read the benchmark notes](#notes)**!

## Benchmark summary

This is **an early benchmark** of [Skytable](https://github.com/skytable/skytable), [Redis](https://github.com/redis/redis) and [KeyDB](https://github.com/EQ-Alpha/KeyDB). It might have some minimal _glitches_. The intention here is to see how fast each database is, out-of-the-box without any added customizations. Skytable uses multi-threaded I/O and so do Redis and KeyDB.

To summarize, this is what things look like:

<img src="get-benches.svg" height="auto" width="100%" alt="Benchmark">

<img src="set-benches.svg" height="auto" width="100%" alt="Benchmark">

(click to see an enlarged image)

Reading the graph:

- The x-axis shows the number of threads
- The y-axis shows the throughput (operations/sec)

> ## Notes
>
> 1: **Seeing the _weird throughput bump_?**  
> Yes, I'm aware of the issue, and I will try to fix the fluctuation issue in an upcoming release.
>
> 2: **The KeyDB benchmark is still experimental**
>
> 3: **No pipelining was used in any benchmark**
>
> 4: **Key/value sizes are different!**
>
> - **Redis uses 3 byte** k/v sizes
> - **Skytable uses 3 byte** k/v sizes
> - **KeyDB** uses an unspecified k/v size (memtier doesn't say much about it; I couldn't find any appropriate documentation)
>
> 5: **Average values were taken**
>
> 6: **Skytable does an extra check:** Skytable's `SET` is not like Redis's/KeyDB's `SET`. It does
> an additional check to see if the key is new (much like `SETNX`)

## Procedure

For running each benchmark:

1. The source code was downloaded and compiled (if required)
2. The machine was allowed to _cool down_ for exactly 1 hour after a reboot to try and reduce nondeterministic cases
3. The benchmark was run

Jump to:

- [Machine specs](#machine)
- [Redis benchmark procedure](#benchmarking-redis)
- [Skytable benchmark procedure](#benchmarking-skytable)
- [KeyDB benchmark procedure](#benchmarking-keydb)

## Machine

Output of `lscpu` (trimmed):

```
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   46 bits physical, 48 bits virtual
CPU(s):                          16
On-line CPU(s) list:             0-15
Thread(s) per core:              2
Core(s) per socket:              8
Socket(s):                       1
NUMA node(s):                    1
Vendor ID:                       GenuineIntel
CPU family:                      6
Model:                           85
Model name:                      Intel(R) Xeon(R) Platinum 8259CL CPU @ 2.50GHz
Stepping:                        7
CPU MHz:                         2499.992
BogoMIPS:                        4999.98
Hypervisor vendor:               KVM
Virtualization type:             full
L1d cache:                       256 KiB
L1i cache:                       256 KiB
L2 cache:                        8 MiB
L3 cache:                        35.8 MiB
NUMA node0 CPU(s):               0-15
```

Output of `uname -a`:

```
Linux ip-172-31-26-159 5.13.0-1022-aws #24~20.04.1-Ubuntu SMP Thu Apr 7 22:10:15 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

Output of `cat /proc/meminfo | grep MemTotal`:

```
MemTotal:       65036640 kB
```

Output of `lsb_release -a`:

```
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.4 LTS
Release:	20.04
Codename:	focal
```

**Summary**
So we have a 16-Core VM running Ubuntu 20.04.4 with 64GB RAM.

## Benchmarking Redis

### Installing Redis

```sh
wget https://download.redis.io/releases/redis-6.2.6.tar.gz
tar -xf redis-6.2.6.tar.gz
cd redis-6.2.6 && sudo make install
```

Now that Redis is installed, let's benchmark `GET` and `SET` in Redis. First start the server (simply `redis-server --io-threads 16`). We'll pass `--csv` to avoid too much output and only show the
required section of output.

### Benchmarking Redis

**Bench 1: 1 clients, 1 thread**

```sh
redis-benchmark -t get,set -n 100000 -c 1 --threads 1 --csv
```

Output:

```
"SET","35549.23","0.021","0.016","0.023","0.031","0.031","0.207"
"GET","35663.34","0.021","0.016","0.023","0.031","0.031","0.247"
```

**Bench 2: 2 clients, 2 threads**

```sh
redis-benchmark -t get,set -n 100000 -c 2 --threads 2 --csv
```

Output:

```
"SET","79936.05","0.020","0.008","0.023","0.023","0.031","0.319"
"GET","79936.05","0.020","0.008","0.023","0.023","0.031","0.287"
```

**Bench 3: 5 clients, 5 threads**

```sh
redis-benchmark -t get,set -n 100000 -c 5 --threads 5 --csv
```

Output:

```
"SET","99800.40","0.043","0.016","0.047","0.055","0.063","0.407"
"GET","99900.09","0.042","0.008","0.039","0.055","0.063","0.479"
```

**Bench 4: 10 clients, 10 threads**

```sh
redis-benchmark -t get,set -n 100000 -c 10 --threads 10 --csv
```

Output:

```
"SET","100000.00","0.090","0.016","0.087","0.111","0.127","0.359"
"GET","99900.09","0.091","0.016","0.087","0.119","0.127","0.295"
```

**Bench 5: 20 clients, 20 threads**

```sh
redis-benchmark -t get,set -n 100000 -c 20 --threads 20 --csv
```

Output:

```
"SET","79808.46","0.194","0.024","0.191","0.239","0.303","14.087"
"GET","99900.09","0.182","0.032","0.175","0.223","0.271","15.927"
```

**Bench 6: 30 clients, 30 threads**

```sh
redis-benchmark -t get,set -n 100000 -c 30 --threads 30 --csv
```

Output:

```
"SET","79872.20","0.297","0.024","0.287","0.351","0.559","19.823"
"GET","99900.09","0.286","0.032","0.279","0.343","0.543","17.839"
```

**Bench 6: 40 clients, 40 threads**

```sh
redis-benchmark -t get,set -n 100000 -c 40 --threads 40 --csv
```

Output:

```
"SET","94966.77","0.284","0.024","0.255","0.375","0.463","108.799"
"GET","98814.23","0.277","0.032","0.239","0.335","0.495","95.871"
```

## Benchmarking Skytable

We'll use the latest stable Skytable release (0.7.5).

### Installing Skytable

Skytable comes with pre-built binaries by default, unlike Redis that requires you to manually build the binaries.

```sh
wget https://github.com/skytable/skytable/releases/download/v0.7.5/sky-bundle-v0.7.5-x86_64-linux-gnu.zip
unzip sky-bundle-v0.7.5-x86_64-linux-gnu.zip
```

### Benchmarking Skytable

To start Skytable, we'll simply run `./skyd`. Like the other benchmarks, we'll trim output to only keep what's necessary. Also the number of threads for the benchmark in Skytable is determined by the number of clients, so we don't have to explicitly specify the number of threads.

**Bench 1: 1 client, 1 thread**

```sh
sky-bench --json -q100000 -c1
```

Output:

```json
[
  { "name": "GET", "stat": 45080.70651766973 },
  { "name": "SET", "stat": 44619.17770341946 },
  { "name": "UPDATE", "stat": 44637.814355164934 }
]
```

**Bench 2: 2 clients, 2 threads**

```sh
sky-bench --json -q100000 -c2
```

Output:

```json
[
  { "name": "GET", "stat": 85939.41975369977 },
  { "name": "SET", "stat": 84749.19672776942 },
  { "name": "UPDATE", "stat": 85019.81860688081 }
]
```

**Bench 3: 5 clients, 5 threads**

```sh
sky-bench --json -q100000 -c5
```

Output:

```json
[
  { "name": "GET", "stat": 176005.44450454682 },
  { "name": "SET", "stat": 175169.52245994573 },
  { "name": "UPDATE", "stat": 174387.18917416828 }
]
```

**Bench 4: 10 clients, 10 threads**

```sh
sky-bench --json -q100000 -c10
```

Output:

```json
[
  { "name": "GET", "stat": 276042.02079694957 },
  { "name": "SET", "stat": 273855.8675870863 },
  { "name": "UPDATE", "stat": 273408.49434915493 }
]
```

**Bench 5: 20 clients, 20 threads**

```sh
sky-bench --json -q100000 -c20
```

Output:

```json
[
  { "name": "GET", "stat": 363006.23495993915 },
  { "name": "SET", "stat": 362381.6830633375 },
  { "name": "UPDATE", "stat": 261012.99510384072 }
]
```

**Bench 6: 30 clients, 30 threads**

```sh
sky-bench --json -q100000 -c30
```

Output:

```json
[
  { "name": "GET", "stat": 606951.4517511007 },
  { "name": "SET", "stat": 610312.8640304531 },
  { "name": "UPDATE", "stat": 676091.4010804704 }
]
```

**Bench 6: 40 clients, 40 threads**

```sh
sky-bench --json -q100000 -c40
```

Output:

```json
[
  { "name": "GET", "stat": 712626.3673524659 },
  { "name": "SET", "stat": 697300.2152009319 },
  { "name": "UPDATE", "stat": 680826.7853727927 }
]
```

## Benchmarking KeyDB

Please note that the KeyDB benchmark is **still experimental**.

### Installing KeyDB

We'll download and install the latest KeyDB release at this time from [this link](https://github.com/EQ-Alpha/KeyDB/releases/tag/v6.2.0).

```sh
wget https://github.com/EQ-Alpha/KeyDB/archive/refs/tags/v6.2.2.zip
unzip v6.2.2.zip
cd KeyDB-6.2.2 && sudo make install
```

We'll start KeyDB by running: `keydb-server --server-threads 16`, explicitly telling the server to use 16 threads (since we have 16 vCPUs).

Do note that KeyDB requires a number of libraries that need to be installed (the package names are for Ubuntu):

- libcurl4-openssl-dev
- uuid-dev
- libssl-dev

### Installing Memtier

[As recommended in the documentation](https://docs.keydb.dev/docs/benchmarking/), we will use the
[memtier benchmark tool](https://github.com/RedisLabs/memtier_benchmark) from Redis Labs to benchmark KeyDB.

```sh
sudo apt-get install build-essential autoconf automake libpcre3-dev libevent-dev pkg-config zlib1g-dev
git clone https://github.com/RedisLabs/memtier_benchmark.git
cd memtier_benchmark
autoreconf -ivf
./configure
sudo make install
```

For memtier, we have to specify the ratio of sets:gets. We will do them individually to get the best numbers.
So we do `0:1` when we want all GETs and `1:0` when we want all sets.
Also `-c1` means one client per thread (similar to `sky-bench`). We'll also use the defaults to get the highest benchmarks.

### Benchmarking KeyDB

**Bench 1: 1 thread/1 client**

```sh
memtier_benchmark -n 100000 --ratio=0:1 -c1 -t1
memtier_benchmark -n 100000 --ratio=1:0 -c1 -t1
```

Output:

```
Gets: 31717.03
Sets: 30415.94
```

**Bench 2: 2 threads/2 clients**

```sh
memtier_benchmark -n 100000 --ratio=0:1 -c1 -t2
memtier_benchmark -n 100000 --ratio=1:0 -c1 -t2
```

Output:

```
Gets: 82389.43
Sets: 81898.44
```

**Bench 3: 5 threads/5 clients**

```sh
memtier_benchmark -n 100000 --ratio=0:1 -c1 -t5
memtier_benchmark -n 100000 --ratio=1:0 -c1 -t5
```

Output:

```
Gets: 100154.04
Sets: 92910.00
```

**Bench 4: 10 threads/10 clients**

```sh
memtier_benchmark -n 100000 --ratio=0:1 -c1 -t10
memtier_benchmark -n 100000 --ratio=1:0 -c1 -t10
```

Output:

```
Gets: 97822.26
Sets: 103777.23
```

**Bench 5: 20 threads/20 clients**

```sh
memtier_benchmark -n 100000 --ratio=0:1 -c1 -t20
memtier_benchmark -n 100000 --ratio=1:0 -c1 -t20
```

Output:

```
Gets: 109766.90
Sets: 100745.19
```

**Bench 6: 30 threads/30 clients**

```sh
memtier_benchmark -n 100000 --ratio=0:1 -c1 -t30
memtier_benchmark -n 100000 --ratio=1:0 -c1 -t30
```

Output:

```
Gets: 273521.45
Sets: 262737.81
```

**Bench 6: 40 threads/40 clients**

```sh
memtier_benchmark -n 100000 --ratio=0:1 -c1 -t40
memtier_benchmark -n 100000 --ratio=1:0 -c1 -t40
```

Output:

```
Gets: 183007.56
Sets: 183930.85
```

## Contributing

Ofcourse! If you find errors or possible sources of error
in any of the benchmarks, please consider opening an issue.

## License

Distributed under the [MIT License](./LICENSE).
