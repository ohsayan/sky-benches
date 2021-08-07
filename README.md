# Skytable v/s the rest

## Benchmark summary

This is **an early benchmark** of [Skytable](https://github.com/skytable/skytable) and [Redis](https://github.com/redis/redis) and [KeyDB](https://github.com/EQ-Alpha/KeyDB). It might have some minimal _glitches_. The intention here is to see how fast each database is, out-of-the-box without any added customizations. Skytable uses multi-threaded I/O and so do Redis and KeyDB.

To summarize, this is what things look like:

<img src="get-benches.png" height="auto" width="100%" alt="Benchmark">

<img src="set-benches.png" height="auto" width="100%" alt="Benchmark">

(click to see an enlarged image)

Reading the graph:

- The bars are in this order: Redis/GET, Skytable/GET and then Redis/SET and Skytable/SET
- Light green and yellow bars are those of Redis
- The blue and dark green bars are those of Skytable
- The x-axis shows the number of threads
- The y-axis shows the throughput (operations/sec)

> ## Notes
>
> 1: **Seeing the _weird throughput bump_?**  
> Yes, I'm aware of the issue, and I will try to fix the fluctuation issue in an upcoming release.
>
> 2: **The KeyDB benchmark is still experimental**

## Machine

Output of `lscpu` (trimmed):

```
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   40 bits physical, 48 bits virtual
CPU(s):                          16
On-line CPU(s) list:             0-15
Thread(s) per core:              1
Core(s) per socket:              16
Socket(s):                       1
NUMA node(s):                    1
Vendor ID:                       GenuineIntel
CPU family:                      15
Model:                           6
Model name:                      Common KVM processor
Stepping:                        1
CPU MHz:                         2199.998
BogoMIPS:                        4399.99
Hypervisor vendor:               KVM
Virtualization type:             full
L1d cache:                       512 KiB
L1i cache:                       512 KiB
L2 cache:                        64 MiB
L3 cache:                        16 MiB
```

Output of `uname -a`:

```
Linux sky-us-central 5.4.0-74-generic #83-Ubuntu SMP Sat May 8 02:35:39 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```

Output of `cat /proc/meminfo | grep MemTotal`:

```
MemTotal:       16395300 kB
```

Output of `lsb_release -a`:

```
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.2 LTS
Release:	20.04
Codename:	focal
```

**Summary**
So we have a 16-Core VM running Ubuntu 20.04.2 with 16GB RAM.

## Benchmarking Redis

### Installing Redis

```sh
wget https://download.redis.io/releases/redis-6.2.5.tar.gz
tar -xf redis-6.2.5.tar.gz
cd redis-6.2.5 && sudo make install
```

Now that Redis is installed, let's benchmark `GET` and `SET` in Redis. First start the server (simply `redis-server &` to run it in the background). We'll pass `--csv` to avoid too much output and only show the
required section of output.

### Benchmarking Redis

**Bench 1: 1 clients, 1 thread**

```sh
redis-benchmark -t set,get -n 100000 -c 1 --threads 1 --csv
```

Output:

```
"SET","16603.02"
"GET","15873.02"
```

**Bench 2: 2 clients, 2 threads**

```sh
redis-benchmark -t set,get -n 100000 -c 2 --threads 2 --csv
```

Output:

```
"SET","36350.42"
"GET","30759.77"
```

**Bench 3: 5 clients, 5 threads**

```sh
redis-benchmark -t set,get -n 100000 -c 5 --threads 5 --csv
```

Output:

```
"SET","44404.97"
"GET","44404.97"
```

**Bench 4: 10 clients, 10 threads**

```sh
redis-benchmark -t set,get -n 100000 -c 10 --threads 10 --csv
```

Output:

```
"SET","39936.10"
"GET","44385.27"
```

**Bench 5: 20 clients, 20 threads**

```sh
redis-benchmark -t set,get -n 100000 -c 20 --threads 20 --csv
```

Output:

```
"SET","44385.27"
"GET","39920.16"
```

**Bench 6: 30 clients, 30 threads**

```sh
redis-benchmark -t set,get -n 100000 -c 30 --threads 30 --csv
```

Output:

```
"SET","39904.23"
"GET","39777.25"
```

## Benchmarking Skytable

We'll use the latest stable Skytable release (v0.6.4)
for benchmarking.

### Installing Skytable

Skytable comes with pre-built binaries by default, unlike Redis that requires you to manually build the binaries. So we'll go ahead and download it.

```sh
wget https://github.com/skytable/skytable/releases/download/v0.6.4/sky-bundle-v0.6.4-x86_64-linux-gnu.zip
unzip sky-bundle-v0.6.4-x86_64-linux-gnu.zip
```

To start Skytable, we'll simply run `./skyd &` to keep it in the background. Like the other benchmark, we'll trim output to only keep what's necessary. Also the number of threads for the benchmark in Skytable is determined by the number of clients, so we don't have to explicitly specify the number of threads.

### Benchmarking Skytable

**Bench 1: 1 client, 1 thread**

```sh
sky-bench -c1 -q100000 --json
```

Output:

```json
[
  { "report": "GET", "stat": 19243.486447917167 },
  { "report": "SET", "stat": 19966.21511150442 }
]
```

**Bench 2: 2 clients, 2 threads**

```sh
sky-bench -c2 -q100000 --json
```

Output:

```json
[
  { "report": "GET", "stat": 38582.11281719525 },
  { "report": "SET", "stat": 36313.763793128164 }
]
```

**Bench 3: 5 clients, 5 threads**

```sh
sky-bench -c5 -q100000 --json
```

Output:

```json
[
  { "report": "GET", "stat": 44292.0943448734 },
  { "report": "SET", "stat": 46593.50782686504 }
]
```

**Bench 4: 10 clients, 10 threads**

```sh
sky-bench -c10 -q100000 --json
```

Output:

```json
[
  { "report": "GET", "stat": 78869.86997358651 },
  { "report": "SET", "stat": 90033.9936397916 }
]
```

**Bench 5: 20 clients, 20 threads**

```sh
sky-bench -c20 -q100000 --json
```

Output:

```json
[
  { "report": "GET", "stat": 346012.6880153796 },
  { "report": "SET", "stat": 341191.7935116819 }
]
```

**Bench 6: 30 clients, 30 threads**

```sh
sky-bench -c30 -q100000 --json
```

Output:

```json
[
  { "report": "GET", "stat": 359502.86118081387 },
  { "report": "SET", "stat": 371029.6102851901 }
]
```

## Benchmarking KeyDB

Please note that the KeyDB benchmark is **still experimental**.

### Installing KeyDB

We'll download and install the latest KeyDB release at this time from [this link](https://github.com/EQ-Alpha/KeyDB/releases/tag/v6.0.16).

```sh
wget https://github.com/EQ-Alpha/KeyDB/archive/refs/tags/v6.0.16.zip
unzip v06.0.16.zip
cd KeyDB-6.0.16 && sudo make install
```

We'll start KeyDB by running: `keydb-server --server-threads 16 &`, explicitly telling the server
to use 16 threads (since we have 16 vCPUs).

Do note that both KeyDB requires a number of libraries that need to be installed (the package names are for Ubuntu):

- libcurl4-openssl-dev
- uuid-dev

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
memtier_benchmark --ratio=0:1 -c1 -t1
memtier_benchmark --ratio=1:0 -c1 -t1
```

Output:

```
Gets: 15004.47
Sets: 15194.45
```

**Bench 2: 2 threads/2 clients**

```sh
memtier_benchmark --ratio=0:1 -c1 -t2
memtier_benchmark --ratio=1:0 -c1 -t2
```

Output:

```
Gets: 31362.12
Sets: 30857.25
```

**Bench 3: 5 threads/5 clients**

```sh
memtier_benchmark --ratio=0:1 -c1 -t5
memtier_benchmark --ratio=1:0 -c1 -t5
```

Output:

```
Gets: 42831.69
Sets: 46195.78
```

**Bench 4: 10 threads/10 clients**

```sh
memtier_benchmark --ratio=0:1 -c1 -t10
memtier_benchmark --ratio=1:0 -c1 -t10
```

Output:

```
Gets: 44949.32
Sets: 43732.82
```

**Bench 5: 20 threads/20 clients**

```sh
memtier_benchmark --ratio=0:1 -c1 -t20
memtier_benchmark --ratio=1:0 -c1 -t20
```

Output:

```
Gets: 48612.15
Sets: 47622.22
```

**Bench 6: 30 threads/30 clients**

```sh
memtier_benchmark --ratio=0:1 -c1 -t30
memtier_benchmark --ratio=1:0 -c1 -t30
```

Output:

```
Gets: 133929.23
Sets: 143908.91
```

### For the adventurous

I wanted to see how high we could go with KeyDB, so I tried this:

```sh
memtier_benchmark --ratio=0:1 -c50 -t30
memtier_benchmark --ratio=1:0 -c50 -t30
```

Equivalent to 50 connections per thread \* 30 threads = 1500 connections. (`-c50` is the default according to the help menu)

Here's what I got:

```
Gets: 154521.92
Sets: 134145.84
```

## Contributing

Ofcourse! If you find errors or possible sources of error
in any of the benchmarks, please consider opening an issue.

## License

Distributed under the [MIT License](./LICENSE).
