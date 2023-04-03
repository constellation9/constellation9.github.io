---
layout: post
title:  "Instantiation with the STAR Protocol"
categories: jekyll update
usemathjax: true
---

We give a brief overview of the algorithms setup, encode, and decode used for specifying the STAR protocol [\[1\]](#references): a highly efficient $K$-TA scheme. The algorithmic description can be seen in Figure 1.

![Constellation instantiated with STAR](/assets/cons-star.jpg "Constellation instantiated with STAR")
*Figure 1: Algorithmic description of Constellation with STAR.*

**Public setup (setup)**. Setup for STAR includes specifying a threshold $K$ that will be used when encoding and decoding client measurements. The maximum lengths of such measurements and auxiliary data (specified by $v$ and $\hat{v}$, respectively) are made public, and data should be padded to this length during encoding. The resulting public parameters, $STAR.pp$, are made available to both the aggregation server and the clients.

**Message encoding (encode)**. An encoded client submission in STAR takes the following form:

$sbm = (cipher, sh, tag)$

To create $sbm$ for a given measurement $x$, the client first samples randomness $r$ by interacting with a randomness server $R$, and deterministically derives three separate portions $r_1, r_2, r_3 \leftarrow r$. It computes sh as a single Shamir secret share of $r_1 \in F_p$ using $r_2$, where the randomness $r_2$ determines a consistent set of polynomial coefficients for sampling secret shares. The clients then sample ephemeral randomness internally for evaluating the polynomial and recovering their specific share, $sh$ (explicit specification of randomness used as input in $K$-out-of-$N$ secret sharing schemes has been documented by Bellare et al. [\[2\]](#references)). Finally, the client derives a symmetric encryption key $\kappa$ from $r_1$, and runs $ciph = ske.encrypt(\kappa, x \mathbin\Vert aux)$ where $aux$ is some additional data of the client’s choosing. Finally, the client sets $tag = r_3$ and constructs $sbm$ as $(ciph, sh, tag)$, and sends it to the server.

**Server-side aggregation (decode)**. The aggregation server, $S$, after receiving $N$ client submissions, constructs subsets containing submissions that share the same value of tag (i.e. they encode the same underlying measurement). For those subsets that contain at least $K$ submissions, $S$ runs the secret sharing recovery algorithm on the collection of $sh$ submissions. As long as $K$ different $sh$ submissions are received, then this allows $S$ to recover $r_1$. Once $S$ has retrieved $r_1$, they are able to derive the encryption key $\kappa$, and decrypt each client submission to learn the common measurement $x$, and each associated auxiliary data $aux$. Note that subsets with less than $K$ submissions cannot be used to retrieve $r_1$, by the security of the secret sharing scheme.

**Randomness server**. The randomness server $R$ is strictly forbidden from colluding with the aggregation server $S$. It runs a verifiable oblivious pseudorandom function (VOPRF) with secret key $sk$. This allows the client to learn evaluations of the form $PRF.eval(sk, x)$, where $x$ is the client input. The security guarantees of the VOPRF ensure that any $x$ remains hidden from $R$, and sk remains hidden from the client. As noted in [\[1\]](#references), the randomness server should rotate their VOPRF key $sk$ at the end of a well-defined epoch $\tau$ . Client submissions sampled in epoch $\tau$ should only be sent to the aggregation server in epoch $\tau + 1$, after key rotation.

## References

1. Alex Davidson, Pete Snyder, E. B. Quirk, Joseph Genereux, Benjamin Livshits, and Hamed Haddadi. STAR: Distributed Secret Sharing for Private Threshold Aggregation. In *Proceedings of the 2022 ACM SIGSAC Conference on Computer and Communications Security, CCS ’22*. Association for Computing Machinery, 2022.
2. Mihir Bellare, Wei Dai, and Phillip Rogaway. Reimagining secret sharing: Creating a safer and more versatile primitive by adding authenticity, correcting errors, and reducing randomness requirements. *PoPETs*, 2020(4):461–490, October 2020.


