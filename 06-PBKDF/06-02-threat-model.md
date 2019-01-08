# 6. Passphrase-based Key Derivation
## 6.2 Threat model
### 6.2.1 Cost analysis
To assess the security of passphrase keys against brute force attacks, we will consider the question of how long it will take for such an attack to become economically feasible to a highly capable adversary. This is complicated by the fact that the adversary may have purpose-built hardware capable of greater efficiency than a general-purpose computer.

It is not clear if or when such hardware will be available for Argon2, or what its cost or efficiency will be. Argon2 is specifically designed to "maximize the cost of password cracking on ASICs."[1] This is presumed to prevent the extreme efficiency gains observed in SHA256 calculation by moving to purpose-built hardware.

This analysis begins by establishing an estimate for the approximate best efficiency (in hashes per dollar) a party can expect today with off-the-shelf equipment. Then, an adversary is designed with a significantly greater efficiency to account for the possibility of access to more efficient means of hashing. Their efficiency is assumed to improve exponentially over time according to Moore's law. Finally, the budget required for this adversary to break passphrases of various strength is calulated, along with the year at which each passphrase class becomes economical to the adversary at a given budget.

[1] https://password-hashing.net/argon2-specs.pdf, section 2.1

### 6.2.2 Real-world performance on high-end system
A virtual machine instance is used as a benchmark reference for real-world performance. Calculating the cost of computing hashes at large scale is complicated by the fact that the adversary's real-world costs scale in difficult-to-model ways (e.g. utility costs and payroll). Using virtual machine pricing allows an objective abstraction of these costs, since the vendor must be bearing these costs and building them into the price.

The following Amazon EC2 instance was selected as a benchmark for the cost efficiency of computing Argon2 hashes using off-the-shelf computing power as of January 2019.

| | |
|-|-|
| Instance type | c5.18xlarge |
| CPU type | Intel(R) Xeon(R) Platinum 8124M CPU @ 3.00GHz
| vCPU | 72 |
| RAM (GiB) | 281 |
| Price | $1.1/hr [1] |

The instance was able to perform most efficiently when 9 simultaneous Argon2 processes were executed. This led to an average rate of 1.614 seconds per derivation, with a standard deviation of 75.675 milliseconds (n=900). This is equivalent to a throughput of 0.6196 passphrase key derivations per second at a price of $1.1/hr, or 2.230 kHash/$.

It is a foregone conclusion that the pricing of these virtual instances must be greater than their cost to the vendor. Since virtualization is commoditized, the profit margins on these instances at their lowest rate is assumed to be bounded. Thus, while it is plausible that the vendor's cost for these machines is at least 20% lower than the long-term rates observed, it is unlikely that the cost is at least 90% lower.

[1]: This is based on the lowest observed "spot rate" for the instance over a period of days ranging from January 1, 2019 to January 7, 2019, reflecting the market price for unused capacity. This price is roughly equivalent to Amazon's least-expensive published means purchase this instance on a long-term basis, which is $1.104/hr. for a prepaid 3-year contract.

### 6.2.3 Adversary

For the purposes of this model the following assumptions are made:

1. The adversary has dedicated a finite budget to computing key derivations.
2. The adversary can instantly compute passphrase key derivations at a rate of 250 kHash/$.
4. The adversary has no fixed costs, and the entire budget can be dedicated to calculating hashes at the specified rate.
3. The cost decreases exponentially at a rate of 50% per 18 months.

The hash rate per dollar is chosen to be over 100 times greater that what was observed in the reference system. This compensates for the possibility that the adversary is able to calculate hashes significantly more efficiently than what was available on the reference system, possibly through a combination of purpose-built hardware and an ability to purchase computing at a significantly lower cost.

### 6.2.4 Passphrase classes
We will now suppose several different passphrase classes. Each class is listed with the amount of money the adversary described in this section must spend as of 2019 to have a 50% chance of brute forcing a passphrase from that class. An adversary is guaranteed to find the passphrase with a budget of double the average cost.

| Entropy (bits) | Description                                          | Example                              | 2019 average cost
|--------|--------------------------------------------------------------|----------------------------------------------|---
| 13.280 | Random selection from 10,000 most commonly-used passwords    | letmein                                      | $0.01989
| 19.930 | Random selection from 1,000,000 most commonly-used passwords | Nokia6233                                    | $1.998
| 51.700 | 4 random words, 12.925 bits entropy/word (Diceware model)    | correct horse battery staple                 | $7.316 billion
| 64.625 | 5 random words, diceware                                     | semantic defender unfold another penalize    | $56.90 trillion
| 77.550 | 6 random words, diceware                                     | landmark maggot errant ranking renewal going | $442.5 quadrillion
| 256    | Derived key size                                             | L*YVS4}@u%i$/X%\_%^NW&hDsOXx=W{n0#.UI8.y     | $2.316 x 10^71

The 256 bit level represents an upper bound on the complexity of an adversary's attack. For some passphrase strength <= 256 bits, it becomes more economical for the adversary to attack the derived keys directly, bypassing the key derivation algorithm entirely.

### 6.2.5 Analysis of attack by adversary on passphrase classes
The following analysis calculates the year in which an adversary will have a 50% chance of discovering the passphrase via brute force within a fixed budget in US dollars, as valued in 2019. It is expected in this model that inflation will affect the attacker's budget and cost of computing equally.

The largest listed budget is $100 trillion, roughly equivalent to the present global GDP. This is chosen as an upper bound on the budget of any adversary. The minimum budget is $0.001. This is chosen as a point where the cost of brute forcing the passphrase is truly negligible, and the adversary may become indifferent as to whether the filesystem is encrypted. This threshold is not easy to define objectively, and it is also unclear when storage costs will make a rainbow table economical for each passphrase class.

| Entropy (bits) | $0.001 | $1   | $1,000 | $1 million | $1 billion | $100 trillion
|----------------|--------|------|--------|------------|------------|---------------
| 13.280         | 2025   | Now  | Now    | Now        | Now        | Now
| 19.930         | 2035   | 2020 | Now    | Now        | Now        | Now
| 51.700         | 2083   | 2068 | 2053   | 2038       | 2023       | Now
| 64.625         | 2102   | 2087 | 2072   | 2057       | 2042       | 2022
| 77.550         | 2121   | 2106 | 2091   | 2077       | 2062       | 2042
| 256            | 2389   | 2374 | 2359   | 2344       | 2329       | 2309

### 6.2.5 Effect of altering adversary's starting hash rate
The attacker is assumed to calculate 100x more hashes per dollar than the reference system, and double every 18 months. Thus, adding or removing factors of 10 to the adversary's starting multiplier causes 4.98 years to be added or removed from the years listed in the table above.

For instance, if an attacker is instead assumed to be able to perform 10000x more efficiently than the reference system as of 2019, then that adversary will be able to attack a 4-word diceware passphrase on a $1 million budget in 2028.

### 6.2.6 Rainbow tables
An attacker has the option of maintaining a "rainbow table" storing the keys derived from every possible passphrase in a class. They could then attempt any of their cached passphrases with a negligible time or computing cost compared to the cost of the key derivation function. An attacker must still conduct an initial brute force attack and store the results, which is impractical for sufficiently-strong passphrases. For weaker passphrases, this option is economical.

If each row of the table was 512 bits in size, then the 2^19.93 level would require 256MiB in storage, and cost the modeled adversary less than $4 to compute.

### 6.2.7 Overestimate of adversary capabilities
The above model is presented to provide an estimate for the minimum duration EasySafe can resist brute force attacks against the passphrase. The adversary of the model is chosen to be more capable than what is realistically possible. The starting capability of the adversary is chosen to be unrealistically high, and the model assumes an exponential reduction of cost by 50% every 18 months. This may not be sustainable due to limitations imposed by, among other things, the laws of thermodynamics.

In ideal conditions, a perfectly-efficient computer powered with the total energy output of a typical supernova could count to 2^219.[1] This is several hundred billion times less expensive than counting to 2^256. This does not account for any of the energy needed to perform any calculations with this counter, such as Argon2 key derivations.

The prediction that the adversary will have access to billions of supernovas worth of energy for the present-day value of $0.001 suggests that the model strongly overestimates the adversary's capabilities. Therefore, the actual lifespan of the confidentiality provided by a passphrase is likely to be greater than what is predicted in this section.

[1] Schneier, Bruce. (1996). Applied Cryptography, Second Edition. Page 158.

### 6.2.8 Inadequacy of ordinary passphrases
Weak passphrase selection is an ongoing problem in information security. Even given a key derivation function that is extremely expensive by current standards, the most commonly-used passwords are economical to attack even today. Users must select passphrases much stronger than what is typically selected today for EasySafe to provide a reasonable assurance of confidentiality.
