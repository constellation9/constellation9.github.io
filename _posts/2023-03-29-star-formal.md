---
layout: post
title:  "Formal Guarantees of STAR"
categories: jekyll update
usemathjax: true
---

We give a description of the formal functionality provided by STAR. We show that STAR satisfies the necessary requirements for a wIND-ENC-secure $K-TA$ scheme. This allows us to build generic $(l, K)-GTA$ schemes with high efficiency.

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

The function then outputs:

$sbm_i = (ciph, sh, tag) = (c, s, r_3)$.

* $T \leftarrow STAR.decode(pp,[sbm_1,\ldots,sbm_N])$: This function is ran by the server $S$ and it involves the following steps:
  * partition client submissions into sets $I \subseteq [N]$, where for each $j \in I$ the value $tag_j = r_3$ is equivalent;
  * discard those sets $I$ where $[I] < K$;
  * let $T$ be an empty vector;
  * for each remaining partition set $I$:
    * each $I$ is made of $sbm_i = (ciph_j, sh_j, tag_j) = (c_j, s_j, r_{3j})$;
    * let $set$ be the set containing each $sh_j$ where $j \in I$;
    * run $r_1 \leftarrow \pi_{K, N}.recover(set)$;
    * run $\kappa = derive(r_1)$;
    * decrypt each $ciph_j$ for $j \in I$ to reveal $(x_j,aux_j)$ (noting that $x_j$ should be equal for all $j \in I$);
    * for each $j \in I$, let $T[j] = (x_j,aux_j,I)$;
  * output $T$.

As mentioned in [\[1\]](#references), we only prove security in a model where an adversary can query the random oracle $RO$ **before** it receives honest client submissions.

To produce random encodings, use $rencode$ that samples a random measurement $x \leftarrow (0,1)^v$, the internal randomness $(r_1,r_2,r_3)$ is sampled directly from $(0, 1)^{3\lambda}$, and then the encoding proceeds in the same way except that $ciph$ is an encryption of zero.

## Proofs of Requirements

We conclude by showing that STAR is a $K$-Threshold Aggregation ($K-TA$) scheme.

### Theorem 4 (Correctness)

STAR satisfies correctness for $K-TA$ schemes, as defined in a $K-TA$ ideal correctness theorem.

*Proof:* This is shown in [\[1\]](#references): the correctness conditions are identical.

### Theorem 5 (wIND-ENC security)

Let $ske$ be an IND-CPA-secure symmetric key encryption scheme, let $\pi_{K, N}$ be a $K$-out-of-$N$ secret sharing scheme, and let $RO$ be a ROM hash function. Then, STAR satisfies wIND-ENC security for $K-TA$ schemes, as defined in a $K-TA$ ideal security theorem, where $K > 1$.

*Proof:* To show that STAR satisfies wIND-ENC security, we essentially need to show that any adversary cannot distinguish between a submission encoding the pair $(x,aux)$ (chosen by $A$), and an encoding of a random measurement (for large measurement spaces). $\square$

Note that in the w-IND-ENC security game, the adversary is not allowed to learn any encodings of $x$ -as opposed to IND-ENC where the adversary can receive up to $K-2$ other encodings from $O_{enc, Q}^{pp}$ for $x$. Therefore, in wIND-ENC, we can analyse the submission $sbm_b = (ciph_b,sh_b,tag_b)$ that $A$ receives from the challenger, and show that it is indistinguishable from a random encoding.

We can construct the following set of hybrid arguments, where each change results in the advantage $Adv_{A, \Gamma, k}^{ind-enc}$ only changing by a negligible quantity.

* $G_0$: This is the setting given in IND-ENC when $b=0$.
* $G_1$: This is the case where $(r_1,r_2,tag_b = r_3)$ are all sampled randomly from $(0,1)^\lambda$. Since each value is the output of a $RO$ in $G_0$ for a measurement that $A$ never makes a query for, they are indistinguishable from random. Therefore the advantage in distinguishing between both cases is zero.
* $G_2$: This is the case where $sh_b$ is sampled randomly from $F_p$. Since $A$ only has the single share of $r_1$, then by the share privacy of $\Pi_{K,N}$ we know that $sh_b$ is perfectly indistiguishable from random. Therefore, the advantage in distinguishing between both cases is again zero.
* $G_3$: This is the case where $ciph_b$ is constructed as an encryption of zero. Note that $sh_b$ and $tag_b$ are random values. Therefore, by the IND-ENC security of $ske$, the ciphertext $ciph_b$ only allows negligible advantage for a PPT adversary to distinguish between encryptions of $(x,aux)$ and zero.
* $G_4$: This is the case where $sbm_b$ is sampled as the output of $rencode$ (i.e. the situation in IND-ENC where $b=1$. The distribution of $sbm_b$ in both $G_3$ and $G_4$ is identical, so this result in extra gain in adversarial advantage.

Following this hybrid argument, we see that the cases of $b=0$ ($G_0$) and $b=1$ ($G_4$) are computationally indistinguishable, therefore, the proof is complete. $\square$

### Theorem 6 (Extractability)

Let $RO$ be a ROM hash function. Then, STAR satisfies extractability for $K-TA$ schemes, as defined in a $K-TA$ ideal extractability theorem.

*Proof:* This follows from the fact that the $STAR.encode$ function outputs $tag = r_3 \in (0,1)^\lambda$, that is an output directly issued by the ROM hash function $RO$. By the fact that the probability of a collision for $r_3$ for two different measurements is $2^{-\lambda/2} = negl(\lambda)$, a simulator can extract the query $x$ corresponding to any encoded submission $sbm$. $\square$


## References

1. Alex Davidson, Pete Snyder, E. B. Quirk, Joseph Genereux, Benjamin Livshits, and Hamed Haddadi. STAR: Distributed Secret Sharing for Private Threshold Aggregation. In *Proceedings of the 2022 ACM SIGSAC Conference on Computer and Communications Security, CCS ’22*. Association for Computing Machinery, 2022
2. Stanislaw Jarecki, Aggelos Kiayias, and Hugo Krawczyk. Roundoptimal password-protected secret sharing and T-PAKE in the password-only model. In Palash Sarkar and Tetsu Iwata, editors, *ASIACRYPT 2014, Part II*, volume 8874 of LNCS, pages 233–253. Springer, Heidelberg, December 2013.
3. Alex Davidson, Ian Goldberg, Nick Sullivan, George Tankersley, and Filippo Valsorda. Privacy pass: Bypassing internet challenges anonymously. *PoPETs*, 2018(3):164–180, July 2018.
4. Ben Kreuter, Tancrede Lepoint, Michele Orru, and Mariana Raykova. Anonymous tokens with private metadata bit. In Daniele Micciancio and Thomas Ristenpart, editors, *CRYPTO 2020*, Part I, volume 12170 of LNCS, pages 308–336. Springer, Heidelberg, August 2020.
5. Sharon Huang, Subodh Iyengar, Sundar Jeyaraman, Shiv Kushwah, Chen-Kuei, Lee Zutian Luo, Payman Mohassel, Ananth Raghunathan, Shaahid Shaikh, Yen-Chieh Sung, and Albert Zhang. Dit: Deidentified authenticated telemetry at scale. 2021. [https://tinyurl.com/yxt7u2ss](https://tinyurl.
com/yxt7u2ss)
6. Nirvan Tyagi, Sofía Celi, Thomas Ristenpart, Nick Sullivan, Stefano Tessaro, and Christopher A. Wood. A fast and simple partially oblivious PRF, with applications. Cryptology ePrint Archive, Report 2021/864, 2021. [https://eprint.iacr.org/2021/864](https://eprint.iacr.org/2021/864).
7. Alex Davidson, Armando Faz-Hernandez, Nick Sullivan, and Christopher A. Wood. Oblivious pseudorandom functions (oprfs) using prime-order groups. Internet-Draft draft-irtf-cfrg-voprf-14, IETF Secretariat, October 2022. [https://www.ietf.org/archive/id/draft-irtf-cfrgvoprf-14.txt](https://www.ietf.org/archive/id/draft-irtf-cfrgvoprf-14.txt).
