# EasySafe attack sketch
## Determine authorship of an encrypted page

Adversary model: Extreme

Complexity: 0*

Caveats: This attack cannot be performed in retrospect. The attacker must be active on the network at the time the page is distributed for the first time. The attacker cannot be certain of the authorship, and at best only suspects the IP address of the original author.

## Method
The adversary posseses the `SeedKey` and FSID, and can therefore participate in the peer-to-peer swarm and observe the IP address of the first peer to announce a new encrypted page. Provided that the adversary is connected to this peer, and the peer chooses to announce the page tag to the adversary prior to or simultaneous with all other peers, the adversary will see this page as having coming from the authoring IP address first.