# 6. Passphrase-based Key Derivation
## 6.1 Parameter selection
### 6.1.1 Argon2
Argon2 is chosen as the algorithm for the `PBKDF` operation. This is due to the extensive peer-review conducted on this algorithm, and its status as the winner of the Password Hashing Competition of 2015. It is specifically designed to maximize the cost of implementation for purpose-built hardware.

## 6.1.2 Argon2 parameter selection

The Argon2i variant is selected for its data-independent memory access and additional memory passes. The authors of Argon2 specifically recommend the Argon2i variant for key derivation use cases.[1]

Argon2 offers a tradeoff between time and resource consumption in the form of its parallelism setting. By allowing a high degree of parallelism, a system with many cores can quickly derive a passphrase key, but must invest multiple cores to do so. There is no requirement that these threads actually run in parallel, and a single-threaded system may still perform the derivation.

As a design goal, the remaining Argon2 parameters were chosen so that 95% home computers would be able to derive a passphrase key in approximately one minute or less. As a further design parameter, it is assumed that a 5th-percentile computer is at 75% utilization due to processes other than EasySafe. This is based on the observation that low-end systems often have high baseline utilization due to the demands of newer operating systems and software, and accumulated background processes.

[1] https://github.com/P-H-C/phc-winner-argon2/blob/master/argon2-specs.pdf, section 3, page 4.

### 6.1.3 Proxy system
A replicable system was defined to act as a proxy to the hypothetical 5th percentile computer running at 75% load. An appropriate number of iterations was then calculated through experimentation on this replicable system.

According to the Steam Hardware and Software Survey of November 2018, over 95% of systems were reported as having at least 4GB of RAM and 2 CPUs.

As the system is assumed to be under 75% load, the Argon2 memory requirement is chosen to be 1GiB based on these figures. The parallelism is set to 16, as an intuitive balance between the number of CPU cores in a typical home computer, and the number likely to be seen in the near future.

The Amazon EC2 t3.small instance was chosen as a suitable proxy system. As a commercial virtual machine, it is highly standardized and easily available as of this writing. The system has 2vCPU and 2GiB RAM, approximating the estimated specifications of the 5th percentile computer.

### 6.1.4 Time cost experiment
The proxy system instance was created with a fresh installation of Ubuntu 18.04[1] in the us-east-1d availability zone. The T3 Unlimited option was enabled. The system reported its CPU type in `/proc/cpuinfo` to be `Intel(R) Xeon(R) Platinum 8175M CPU @ 2.50GHz`. The `build-essential` package was installed, and Argon2 was then downloaded, compiled[2] and run from its reference implementation[3], and executed with 16 threads of parallelism and 1GiB of RAM utilization. The salt and passphrase were chosen to be representative of what is expected for ordinary use of EasySafe.

This was repeated at various iteration counts until a count that consistently completed in an average of approximately 15 seconds was identified. The target time of 15 seconds was chosen under the assumption that it is a useful proxy for a target time of 60 seconds on a system under 75% CPU load.

The following command, specifying 40 iterations, was found to have an average completion time of 15.122 seconds[4], with a standard deviation of 71.065ms (n=100):

```
echo -n "landmark maggot errant ranking renewal going" | \
    time -f "%e" ./argon2 easysafe-argon2-salt -i -k 1048576 -p 16 -l 32 -e -v 13 -t 10 > /dev/null
```

Thus, an iteration count of 40 was selected for the Argon2 parameters of EasySafe.

\[1]: Ubuntu 18.04 was installed from the `ami-0ac019f4fcb7cb7e6` image.

\[2]: The gcc 7.3.0-27ubuntu1~18.04 x86_64-linux-gnu compiler was used with Ubuntu GLIBC 2.27-3ubuntu1

\[3]: Obtained from https://github.com/P-H-C/phc-winner-argon2. Commit 6c8653c was used, as it was the latest commit to the 'master' branch at the time of evaluation.

\[4]: When replicating the experiment, the first Argon2 run on a fresh VM was observed to be above 16 seconds. The first run is excluded from the average as an outlier skewed by the need to initialize caches.

