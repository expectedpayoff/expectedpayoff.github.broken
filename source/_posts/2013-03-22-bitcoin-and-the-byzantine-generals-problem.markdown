---
layout: post
title: "Bitcoin &amp; the Byzantine Generals Problem"
date: 2013-03-22 12:52
categories: [bitcoin, economics, computer science]
comments: true
published: true
author: Byron Gibson
---

For those still wrapping their heads around Bitcoin, hopefully this can help.  In a nutshell, try thinking of it not as money but in terms of accounting ledgers and the Byzantine Generals Problem. 
 
Bitcoin is an electronic ledger, a copy of which is kept on every participant's computer in the network, and all of which are continually updated, reconciled, and synchronized in real-time.  Every participant can make entries in this ledger, which records transactions of a certain amount of currency from one participant to another participant, and every one of those entries is then propagated to the network in realtime, so that every copy on every computer is updated near simultaneously and all copies of the ledger are kept synchronized.  The official term for this public, distributed ledger is the ['blockchain'][1] (which [you can see here][2]), and it uses [Bittorrent][11] technology to keep all copies synchronized.

You can also think of Bitcoin not as a currency but as a general solution to a [difficult algorithmic problem][6] in the field of distributed systems, colloquially known as [Byzantine Fault Tolerance][3], the [Byzantine Generals Problem][4], or the [Two Generals Problem][5].

<!-- more -->

The fun, informal statement of the problem is this:  Imagine during the Byzantine era, there is a fortified city-state, Byzantium, that contains within its walls wealth beyond the wildest dreams and avarice of its neighbors.  It is surrounded by ten other city-states, each of which is also wealthy, but nowhere near as wealthy as Byzantium.  All ten neighbors covet Byzantium's wealth, and wish to invade and take it for their own.

However, Byzantium is so well defended that none of the neighboring city-states is strong enough to succeed at an invasion.  Any lone invasion attempt would fail and result in the total annihilation of the invader's army, leaving it vulnerable to invasion and pillaging by any of the other nine, all of which also covet each other's wealth and constantly scheme against each other.  Moreover, Byzantium's defenses are so strong, that it will take more than half the of the ten neighbors' armies attacking simultaneously to overcome.  

So, if six or more neighboring armies attack together, they will succeed and gain the riches of Byzantium.  However, if one or more betrays the rest of the group by agreeing to invade and then holding back while the others attack, so that only five or less armies attack simultaneously, then all the attacking armies will be annihilated, and then pillaged by the remaining neighbors, including the ones that betrayed them.  It is a network of untrusting parties that must nevertheless work together to accomplish a shared mission.

Moreover, the only way that the ten neighbors can communicate and coordinate the attack time is by sending messengers on horseback amongst themselves.  They can't all meet in one place for a conference (none of the kings trust the others outside the safety of his own fortress or army).  However, they can send as many messengers, as frequently as they want, and to whomever they want, at any time.  Each message contains something like "I will attack on Day 4 at 0600, will you join me?". 

If the recipient agrees, they attach a signed/certified/sealed/verified response to the original message, then send out copies of the newly combined message to all 9 neighbors again, asking each of them to do the same.  The eventual objective is to have them all agreeing to the same time by adding all ten of their seals onto the same message chain.  The result will be a message chain with ten seals all agreeing to the same time, and probably several discarded message chains that contained a few seals but not all.

However, the problem is that if each city-state sends one messenger apiece to each of the other nine, then that's 10 cities sending 9 messages each, 90 total in transit at any one time, and each city receiving 9 messages each, ostensibly each with different attack times.  Further, some of the cities will agree to more than one attack time, intending to betray the originator, so they will rebroadcast more than one message chain.  This system quickly degenerates into a completely untrustworthy tangle of conflicting messages and attack times.

Bitcoin solves this problem with a simple (in hindsight) modification to the system - it adds a cost to sending messages, which slows down the rate of message passing, and does so with an element of randomness that ensures that only one city-state will be able to broadcast at a time.  The cost that it adds is a ['proof of work'][7], and it is based on calculating a random hash algorithm.  A hash is an algorithm that does nothing more than takes some input data and performs a calculation on it that results in a 64-digit string of random numbers and letters, like this:

    d70298566aa2f1a66d892dc31fedce6147b5bf509e28d29627078d9a01a8f86b

In the case of bitcoin, the input data consists of the entire ledger/blockchain up to the current point in time.  And while a single hash value can be calculated nearly instantaneously with today's computers, only a hash value where the first thirteen characters are zero is accepted by bitcoin as the 'proof of work'.  Such a thirteen-zero hash code is extremely improbable and rare, and currently takes the entire bitcoin network about 10 minutes of searching before one is found.  Billions of invalid hash codes are generated before one machine in the network randomly lands on a valid one, and this is the 'proof of work' that slows down the rate of message passing and makes the whole system viable.  Here's what they look like:

    f51d0199c4a6d9f6da230b579d850698dff6f695b47d868cc1165c0ce74df5e1

    d70298566aa2f1a66d892dc31fedce6147b5bf509e28d29627078d9a01a8f86b

    119c506ceaa18a973a5dbcfbf23253bc970114edd1063bd1288fbba468dcb7f8

... millions or billions more like this until finally a valid one ... 

    000000000000084b6550604bf21ad8a955b945a0f78c3408c5002af3cdcc14f5

The machine (or city state in our analogy) that discovers the next valid hash code is the one that gets to take all previous messages, append its own, along with its signature/seal/etc, and broadcast out to the rest of the network.  As soon as the other machines in the network receive and verify this new thirteen-zero hash code and attached messages, they stop their current calculations, update their copy of the ledger with new version, and start calculating the hash again by feeding the newly updated ledger/blockchain to the hashing algorithm.  The hash calculation race is reset from a new starting point and begun anew.  Thus the network constantly syncs itself so that all computers on it are using the same version of the ledger for their hashing calculations.

In the meantime, there are about 10 minutes in between successful hash code discoveries and blockchain updates (this is intentional, and the algorithm difficulty is adjusted every two weeks to make sure it takes the network a constant 10 minutes to find a new valid hash code).  During that 10m, participants on the network are sending messages and making transactions, and since every machine on the network is using the same ledger, all those transactions and messages are entered into every copy of the ledger across the entire network.  When the blockchain is updated and synced across the network, all transactions that were entered into the blockchain during the previous 10 minutes are also updated and synced.  Hence the distributed transaction record is reconciled and synchronized among all participants.

Last, when individuals enter a transaction into the network, they use standard [public key cryptography][8] tools built into the bitcoin client program to sign it with their public key and the public key of the recipient.  This is the counterpart to the 'seal' that the Byzantine Generals signed and verified their messages with.  Hence the hash calculation rate limiter combined with public key cryptography, turn an untrusting network into a trusting one, allowing all the participants to agree (on an attack time, or a series of transactions, [domain name records][9], political voting system, or anything else that needs distributed agreement).

So the crux of why Bitcoin is special:  It represents a solution to a difficult algorithmic problem that was not possible until a confluence of historical events:

1.  The creation of the Internet.

2.  The invention of public key cryptography.

3.  The invention of the [P2P][10] [Bitorrent Protocol][11].  Bittorrent was originally developed for sharing many static files between relatively small subsets of the users on the network, but Bitcoin repurposes it to share a single continuously-updating file amongst *all* users.  

4.  The realization that adding a simple time delay into the system along with public key cryptography to verify individual transactions could solve the problem.  

If some of the most brilliant ideas are simple in hindsight, #4 at least certainly qualifies, though overall the project has stood on the shoulders of giants.

Finally, this solution to the Byzantine Generals Problem can be repurposed to any domain where lack of trust on a distributed network is the core problem.  As already mentioned, people are building a [distributed Domain Name System][9] for the Internet, and are working distributed voting systems for use in political elections (no website yet), for starters.  If people thought that mere filesharing disrupted the world, the Bitcoin solution and protocol are just beginning to open the floodgates.

[1]:    https://en.bitcoin.it/wiki/Blockchain
[2]:    http://blockchain.info/
[3]:    https://en.wikipedia.org/wiki/Byzantine_fault_tolerance
[4]:    http://en.wikipedia.org/wiki/Byzantine\_Generals%27\_Problem
[5]:    https://en.wikipedia.org/wiki/Two_Generals%27_Problem
[6]:    https://en.wikipedia.org/wiki/Consensus_(computer_science)
[7]:    https://en.bitcoin.it/wiki/Proof_of_work
[8]:    https://en.wikipedia.org/wiki/Public-key_cryptography
[9]:    https://dot-bit.org/Main_Page
[10]:   https://en.wikipedia.org/wiki/Peer_to_peer
[11]:   https://en.wikipedia.org/wiki/BitTorrent
