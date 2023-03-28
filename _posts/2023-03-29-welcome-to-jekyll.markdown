---
layout: post
title:  "Formal Guarantees of STAR"
categories: jekyll update
usemathjax: true
---

We now give a description of the formal functionality provided by STAR. We show that STAR satisfies the necessary requirements for a wIND-ENC-secure $K-TA$ scheme. This allows us to build generic $(l, K)-GTA$ schemes with high efficiency.

## Required Cryptographic Primitives

We describe the cryptographic primitives that are required in the STAR protocol, see [\[1\]](#references) for a full description.
Note that some primitives and notation needed are defined in Section 4.1.

**Secret-sharing.** We require a $K$-out-of-$N$ threshold secret sharing scheme $\Pi_{K,N}$ with information-theoretic security, operating in a finite field $F_p$ for some prime $p > 0$, where $p$ is sufficiently large to avoid collisions (e.g. p has 128-bits).
Such a scheme consists of two algorithms:

* $s \leftarrow \Pi_{K,N}.share(z; r)$: a probabilistic algorithm that produces a random $K$-out-of-$N$ share $s \in F_p$ of $z$;
* $(\bar{z},\bot) ← \Pi_{K,N}.recover(\(s_i\)_{i \in [\ell]}\)$: outputs $\bar{z}$ when $\ell \geq K$ and each $s_i$ is a valid share of $\bar{z}$; otherwise, it outputs $\bot$.

For security, we require that any set of shares for an unpredictable value $x$ that remains smaller than $K$ is perfectly indistinguishable from a set of random field elements. We call this property share privacy, and is achieved in secret sharing approaches based on Shamir secret sharing [6].

**Random oracle queries.** The STAR protocol requires clients to interact with a *randomness server* that provides them with pseudorandom function’s evaluations obliviously. The client eventually receives three random values $(r_1, r_2, r_3)$ as the output of a ROM hash function $RO$ on their input measurement $x$.

As shown in [\[1\]](#references), STAR can be minimally constructed using the ROM *2HashDH* oblivious pseudorandom function (OPRF) of [\[2\]](#references), defined in a multiplicative cyclic group $G$ with prime-order $p' = poly(\lambda)$. This OPRF has been used widely in both practical constructions [\[3\]](#references), [\[4\]](#references), [\[5\]](#references), [\[6\]](#references) and in standardisation contexts [\[7\]](#references). However, since these OPRFs require evaluating a ROM hash function to compute the final output, we can model the randomness server as a random oracle, $RO$, that clients query when creating encoding submissions.

**Other algorithms.** The STAR scheme also requires other algorithm $\kappa = derive(r_1)$ that is used to derive a symmetric key $\kappa$ using a pseudorandom generator where $r_1$ is used as the seed.

## Formal Functionality

Let $S$ denote the service provider (or aggregation server), let $C_i$ denote one of $N$ clients, and let $RO$ denote the random oracle that the client queries. The API description of STAR, is described as follows.

* $pp \leftarrow STAR.setup(1^\lambda,1^K,1^v,1^\bar{v})$: An algorithm that defines the publicly agreed parameters, which are the security parameter $\lambda$, threshold $K$, and input and auxiliary data lengths $v, \bar{v}$, respectively.
* $sbm \leftarrow STAR.encode(pp,x_i,aux_i)$: This function involves the client $C_i$ running the following steps:
  * Send $x$ to $RO$ and receive back $(r_1,r_2,r_3) \in \{0,1\}^{3\lambda}$;
  * Let $s = \Pi_{K,N}.share(r_1, r_2)$;
  * Let $\kappa = derive(r_1)$;
  * Let $c = ske.encrypt(\kappa, x_i \mathbin\Vert aux_{i})$.

## References

1. Alex Davidson, Pete Snyder, E. B. Quirk, Joseph Genereux, Benjamin Livshits, and Hamed Haddadi. STAR: Distributed Secret Sharing for Private Threshold Aggregation. In *Proceedings of the 2022 ACM SIGSAC Conference on Computer and Communications Security, CCS ’22*. Association for Computing Machinery, 2022
2. Stanislaw Jarecki, Aggelos Kiayias, and Hugo Krawczyk. Roundoptimal password-protected secret sharing and T-PAKE in the password-only model. In Palash Sarkar and Tetsu Iwata, editors, *ASIACRYPT 2014, Part II*, volume 8874 of LNCS, pages 233–253. Springer, Heidelberg, December 2013.

