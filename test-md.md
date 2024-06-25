# Groth16 V2

The groth16 algorithm enables a quadratic arithmetic program to be computed by a prover over elliptic curve points derived in a trusted setup and quickly checked by a verifier. It uses auxiliary elliptic curve points from the trusted setup to prevent forged proofs.

## Prerequisites

This article is a chapter in the [RareSkills Book of Zero Knowledge Proofs](https://rareskills.io/zk-book). It assumes you are familiar with the prior chapters.

## Notation and terminology

Letters in **bold** are vectors. For example, $\mathbf{a}$ is a vector. An elliptic curve point is a letter in square brackets; the subscript denotes which group it belongs to. For example, $[A]_1$ is an elliptic curve point that is part of the $\mathbb{G}_1$ group and $[B]_2$ is an elliptic curve point that is a member of the $\mathbb{G_2}$ group. We refer to the generators of the groups as $[G_1]$ and $[G_2]$. The pairing between points is a dot: $[A]_1\cdot[B]_2$.

Given a Rank 1 Constraint System $L\mathbf{a}\circ R\mathbf{a}=O\mathbf{a}$ with $n \times m$ matrices $L, R, O$ we can create a Quadratic Arithmetic Program

$$
\sum_{i=1}^ma_iu_i(x)\sum_{i=1}^ma_iv_i(x)=\sum_{i=1}^ma_iw_i(x)+h(x)t(x)
$$

The polynomials $u_1(x)\dots u_m(x)$, $v_1(x)\dots v_m(x)$, $w_1(x)\dots w_m(x)$ are the Lagrange interpolation of the $m$ columns of $L, R, O$ respectively. Those polynomials have degree $n-1$ (remember, polynomials always have one less degree than the number of points they interpolate).

## Trusted Setup for QAP and Fake Proofs

The trusted setup can evaluate each of the polynomials $u_1(x)\dots u_m(x)$, $v_1(x)\dots v_m(x)$, $w_1(x)\dots w_m(x)$ at the point $x=\tau$ and multiply the result by the generator $[G_1]$ or $[G_2]$. In this sense, we are “evaluating the polynomial for the prover” and “hiding the result behind an elliptic curve point.” Then prover simply plugs in their choice of values for $\mathbf{a}$.

Thus, the trusted setup evaluates the $u_i$ polynomials at $\tau$ to produce the elliptic curve points $[\Theta_1]_1,\dots,[\Theta_m]_1$:

$$
\begin{align*}[\Theta_1]_1 &= u_1(\tau)[G_1] \\ [\Theta_2]_1&=u_2(\tau)[G_1]\\\vdots&\\ [\Theta_m]_1 &= u_m(\tau)[G_1]\end{align*}
$$

The $v_i$ polynomials at $\tau$:

$$
\begin{align*}[\Omega_1]_1 &= v_1(\tau)[G_2] \\ [\Omega_2]_1&=v_2(\tau)[G_2]\\\vdots&\\ [\Omega_m]_1 &= v_m(\tau)[G_2]\end{align*}
$$

And the $w_i$ polynomials at $\tau$:

$$
\begin{align*}[\Psi_1]_1 &= w_1(\tau)[G_1] \\ [\Psi_2]_1&=w_2(\tau)[G_1]\\\vdots&\\ [\Psi_m]_1 &= w_m(\tau)[G_1]\end{align*}
$$

And the evaluations for $h(x)t(\tau)$

$$
\begin{align*}[\Tau_0]_1&=t(\tau)[G_1] \\ [\Tau_1]_1&=\tau t(\tau)[G_1]\\ [\Tau_2]_1&=\tau^2 t(\tau)[G_1] \\\vdots \\ [\Tau_d]_1&=\tau^d t(\tau)[G_1]\end{align*}
$$

For succinctness, we can write the above operations as:

test input Θ Ω Ψ Τ

The trusted setup now consists of the vectors $\mathbf{\Theta}$, $\mathbf{\Omega}$, $\mathbf{\Psi}$, and $\mathbf{\Tau}$. A prover with a satisfying witness vector $\mathbf{a}$ can compute:

$$
\begin{align*}[A]_1&=\sum_{i=1}^ma_i [\Theta_i]_1 \\ 

[B]_2&=\sum_{i=1}^ma_i[\Omega_i(x)]_2 \\

[C]_1&=\sum_{i=1}^ma_i[\Psi_i]_1+\sum_{i=0}^dh_i[T_i]_1\end{align*}
$$

The prover is simply taking the inner product of their witness vector $\mathbf{a}$ with the elliptic curve points from the trusted setup, and taking the inner product of the $h$ polynomial with the vector $T$.

The verifier can check:

$$
[A]_1\cdot[B]_2\stackrel{?}{=}[C]_1\cdot[G_2]
$$

However, the verifier cannot trust this solution because A, B, and C could be arbitrary points chosen by the prover.

## Motivation

In our chapter on [evaluating a Quadratic Arithmetic Program at a hidden point τ](https://www.rareskills.io/post/elliptic-curve-qap), we had a significant issue that the prover can simply invent values a, b, c where ab = c and present those as elliptic curve points to the verifier.

Thus, the verifier has no idea if elliptic curve points $[A]_1,[B]_2,[C]_1$ were the result of a satisfied QAP or made up values.

We need to force the prover to be honest without introducing too much computational overhead. The first algorithm to accomplish this was “[Pinocchio: Nearly Practical Verifiable Computation](https://eprint.iacr.org/2013/279.pdf).” This was usable enough for zcash to base the first version of their blockchain on it.

However, Groth16 accomplished the same thing in much fewer steps, and the algorithm is still widely used today, as it is highly optimized for verifier efficiency — it only requires four pairing operations. As of 2024, variant of Groth16 called [Polymath](https://eprint.iacr.org/2024/916.pdf) was able to accomplish the verification step with two paring operations, and the cost of extra overhead for the prover.

<aside>
💡 Compared to other more modern ZK algorithms, Groth16 has the disadvantage of requiring a per-application trusted setup, is not quantum compute resistant, and has relatively high overhead for the prover. Nonetheless, Groth16 is still useful a tried-and-tested algorithm with multiple implementations and efficient execution in a smart contract setting.

</aside>

## Algorithm without $[\alpha]$ and $[\beta]$

Given a QAP

$$
\sum_{i=1}^ma_iu_i(x)\sum_{i=1}^ma_iv_i(x)=\sum_{i=1}^ma_iw_i(x)+h(x)t(x)
$$

The trusted setup can evaluate each of the polynomials $u_1(x)\dots u_m(x)$, $v_1(x)\dots v_m(x)$, $w_1(x)\dots w_m(x)$ at the point $x=\tau$ and multiply the result by the generator $[G_1]$ or $[G_2]$. In this sense, we are “evaluating the polynomial for the prover” and “hiding the result behind an elliptic curve point.” Then prover simply plugs in their choice of values for $\mathbf{a}$.

Thus, the trusted setup evaluates the $u_i$ polynomials at $\tau$ to produce the elliptic curve points $[\Theta_1]_1,\dots,[\Theta_m]_1$:

$$
\begin{align*}[\Theta_1]_1 &= u_1(\tau)[G_1] \\ [\Theta_2]_1&=u_2(\tau)[G_1]\\\vdots&\\ [\Theta_m]_1 &= u_m(\tau)[G_1]\end{align*}
$$

The $v_i$ polynomials at $\tau$:

$$
\begin{align*}[\Omega_1]_1 &= v_1(\tau)[G_2] \\ [\Omega_2]_1&=v_2(\tau)[G_2]\\\vdots&\\ [\Omega_m]_1 &= v_m(\tau)[G_2]\end{align*}
$$

And the $w_i$ polynomials at $\tau$:

$$
\begin{align*}[\Psi_1]_1 &= w_1(\tau)[G_1] \\ [\Psi_2]_1&=w_2(\tau)[G_1]\\\vdots&\\ [\Psi_m]_1 &= w_m(\tau)[G_1]\end{align*}
$$

And the evaluations for $h(x)t(\tau)$

$$
\begin{align*}[\Tau_0]_1&=t(\tau)[G_1] \\ [\Tau_1]_1&=\tau t(\tau)[G_1]\\ [\Tau_2]_1&=\tau^2 t(\tau)[G_1] \\\vdots \\ [\Tau_d]_1&=\tau^d t(\tau)[G_1]\end{align*}
$$

For succinctness, we can re-write the above notation as:

$$
\begin{align*}
[\Theta_i]_1&=u_i(\tau)[G_1]  &\text{for i = 1 to m} \\

[\Omega_i]_2&=v_i(\tau)[G_2] &\text{for i = 1 to m} \\

[\Psi_i]_1 &= w_i(\tau)[G_1]&\text{for i = 1 to m}\\

[\Tau_i]_1 &= \tau^i t(\tau)[G_1]&\text{for i = 0 to d}\\

\end{align*}
$$

The trusted setup now consists of the vectors $\mathbf{\Theta}$, $\mathbf{\Omega}$, $\mathbf{\Psi}$, and $\mathbf{\Tau}$. A prover with a satisfying witness vector $\mathbf{a}$ can compute:

$$
\begin{align*}[A]_1&=\sum_{i=1}^ma_i [\Theta_i]_1 \\ 

[B]_2&=\sum_{i=1}^ma_i[\Omega_i(x)]_2 \\

[C]_1&=\sum_{i=1}^ma_i[\Psi_i]_1+\sum_{i=0}^dh_i[T_i]_1\end{align*}
$$

The prover is simply taking the inner product of their witness vector $\mathbf{a}$ with the elliptic curve points from the trusted setup, and taking the inner product of the $h$ polynomial with the vector $\mathbf{\Tau}$.

The verifier can check:

$$
[A]_1\cdot[B]_2\stackrel{?}{=}[C]_1\cdot[G_2]
$$

However, the verifier cannot trust this solution because A, B, and C could be arbitrary points chosen by the prover.

## Preventing forgery part 1: introducing α and β

### **Secret shifting**

It’s easy for the prover to fake values because they know the values of $[A]_1$, $[B]_2$, and $[C]_1$. But what if we force the prover to shift the values $[A]_1$ and $[B]_2$ by an unknown amount?

This can be accomplished by adding elliptic curve points $[\alpha]_1$ and $[\beta]_2$ to $[A]_1$ and $[B]_2$ respectively. These are created by the trusted setup randomly sampling $\alpha$ and $\beta$, creating elliptic curve points $[\alpha]_1 =\alpha[G_1]$ and $[\beta]_2=\beta[G_2]$ and then destroying  $\alpha$ and $\beta$.

To see how this works, consider the following algebra:

We start with a QAP

$$
\sum_{i=1}^ma_iu_i(x)\sum_{i=1}^ma_iv_i(x)=\sum_{i=1}^ma_iw_i(x)+h(x)t(x)
$$

We introduce new terms to the left sums and expand the result

$$
(\zeta+\sum_{i=1}^ma_iu_i(x))(\phi+\sum_{i=1}^ma_iv_i(x))
$$

$$
=\zeta\phi + \phi\sum_{i=1}^ma_iu_i(x) + \zeta \sum_{i=1}^ma_iv_i(x) + \boxed{\sum_{i=1}^ma_iu_i(x)\sum_{i=1}^ma_iv_i(x)}
$$

The boxed term on the right is the original left-hand-side of the QAP, so we can substitute it with the right-hand-side of the QAP:

$$
=\zeta\phi + \phi\sum_{i=1}^ma_iu_i(x) + \zeta \sum_{i=1}^ma_iv_i(x) + \boxed{\sum_{i=1}^ma_iw_i(x)+h(x)t(x)}
$$

We can collect all the summations into a single term as follows:

$$
=\zeta\phi + \sum_{i=1}^ma_i\phi u_i(x) + \sum_{i=1}^ma_i\zeta v_i(x) + \sum_{i=1}^ma_iw_i(x)+h(x)t(x)
$$

$$
=\zeta\phi + \sum_{i=1}^ma_i(\phi u_i(x) + \zeta v_i(x) + w_i(x))+h(x)t(x)
$$

This results in the following equality:

$$
(\zeta+\sum_{i=1}^ma_iu_i(x))(\phi+\sum_{i=1}^ma_iv_i(x))=\zeta\phi + \sum_{i=1}^ma_i(\phi u_i(x) + \zeta v_i(x) + w_i(x))+h(x)t(x)
$$

If we substitute $\zeta$ and $\phi$ for $[\alpha]_1$ and $[\beta]_2$ we get the following formula:

$$
([\alpha]_1+\sum_{i=1}^ma_iu_i(x))([\beta]_2+\sum_{i=1}^ma_iv_i(x))=[\alpha]_1\cdot[\beta]_2 + \sum_{i=1}^ma_i\boxed{(\beta u_i(x) + \alpha v_i(x) + w_i(x))}+h(x)t(x)
$$

The polynomial in the box term can be precomputed during the trusted setup, so that the prover doesn’t need to know $\alpha$ or $\beta$.

In order to keep $\alpha$ and $\beta$ hidden from the prover, we need to adjust our trusted setup as follows:

$$
\begin{align*}
[\Theta_i]_1&=u_i(\tau)[G_1]  &\text{for i = 1 to m} \\

[\Omega_i]_2&=v_i(\tau)[G_2] &\text{for i = 1 to m} \\

[\Psi_i]_1 &= (\beta u_i(x) + \alpha v_i(x) + w_i(x))[G_1]&\text{for i = 1 to m}\\

[\Tau_i]_1 &= \tau^i t(\tau)[G_1]&\text{for i = 0 to d}\\

[\alpha]_1&=\alpha[G_1]\\

[\beta]_2&=\beta[G_2]

\end{align*}
$$

The prover now computes:

$$
\begin{align*}[A]_1&=[\alpha]_1+\sum_{i=1}^ma_i [\Theta_i]_1 \\ 

[B]_2&=[\beta]_2+\sum_{i=1}^ma_i[\Omega_i(x)]_2 \\

[C]_1&=\sum_{i=1}^ma_i[\Psi_i]_1+\sum_{i=0}^dh_i[T_i]_1\end{align*}
$$

And the verifier computes:

$$
[A]_1\cdot[B]_2\stackrel{?}{=}[\alpha]_1\cdot[\beta]_2+[C]_1\cdot[G_2]
$$

### Attack 1: Forging A and B and deriving C

Suppose the prover randomly selects a’ and b’ to produce [A]₁ and [B]₂ and tries to derive a value [C’] that is compatible with the verifier’s formula. Because of the [α]₁[β]₂ term, the prover cannot simply provide c’ = a’b’ then compute [C’].

Here is how the malicious prover would try:

First they pick [A’]₁ and [B’]₂ which results in a pairing [D]₁₂. The malicious prover then does:

$$
D_{12} - ([\alpha]_1[\beta]_2)_{12}=C'_{12}
$$

However, the prover now has a $\mathbb{G}_{12}$ point for $C$ instead of a $\mathbb{G}_1$ point, which the verification formula needs. To get that $\mathbb{G}_1$ point, the malicious prover would have to take the discrete logarithm of [C’]₁₂ to get c’ and then multiply that with G₁ to get [c’G₁]₁.

But the discrete logarithm is infeasible, so this attack does not work.

Note that the malicious prover knows [D], but the reason the attack fails is because they do not know αβ.

Let’s try attacking in the other direction.

### **Attack 2: Forging C and deriving A and B**

Here the prover picks a random point c’ and compute [C’]₁. Because they know c’, they can try to discover a compatible combination of a' and b' such that a'b' = c'.

Hence, they compute:

$$
D_{12} = ([\alpha]_1\cdot[\beta]_2)+([C']\cdot[G_2])_{12}
$$

Now they need to split [D’]₁₂ into [A]₁ and [B]₂. However, the malicious prover runs into the discrete logarithm problem again because they don’t know the preimage of [D’]₁₂. Given an elliptic curve point in G₁₂, you cannot find two points from G₁ and G₂ that pair to it unless you know the underlying field element that generated the G₁₂ element.

### **Attack 3: Forging [A’] and [B’] with [a’G₁] + [α]₁ and [b’G₂] + [β]₂ and computing [C’]**

This is a bit more clever. It seems like the equation should identically balance out right?

Here is the steps for the attack

1. Pick a random a’ and b’
2. Generate [A’] = a’[G] + [α], [B’] = b’[G] + [β]

The rest of the attack, and why it fails, is an exercise for the reader.

## Recap for the algorithm with $[\alpha]$ and $[\beta]$

We start with a publicly agreed upon QAP:

$$
\sum_{i=1}^ma_iu_i(x)\sum_{i=1}^ma_iv_i(x)=\sum_{i=1}^ma_iw_i(x)+h(x)t(x)
$$

### Trusted setup

$$
\begin{align*}
[\Theta_i]_1&=u_i(\tau)[G_1]  &\text{for i = 1 to m} \\

[\Omega_i]_2&=v_i(\tau)[G_2] &\text{for i = 1 to m} \\

[\Psi_i]_1 &= (\beta u_i(x) + \alpha v_i(x) + w_i(x))[G_1]&\text{for i = 1 to m}\\

[\Tau_i]_1 &= \tau^i t(\tau)[G_1]&\text{for i = 0 to d}\\

[\alpha]_1&=\alpha[G_1]\\

[\beta]_2&=\beta[G_2]

\end{align*}
$$

The trusted setup publishes $(\mathbf{\Theta}_1,\mathbf{\Omega}_2,\mathbf{\Psi}_1,\mathbf{\Tau}_1,[\alpha]_1,[\beta]_2)$

### Prover

$$
\begin{align*}[A]_1&=[\alpha]_1+\sum_{i=1}^ma_i [\Theta_i]_1 \\ 

[B]_2&=[\beta]_2+\sum_{i=1}^ma_i[\Omega_i(x)]_2 \\

[C]_1&=\sum_{i=1}^ma_i[\Psi_i]_1+\sum_{i=0}^dh_i[T_i]_1\end{align*}
$$

The prover sends $([A]_1,[B]_2,[C]_1)$ to the verifier.

### Verifier

The verifier computes:

$$
[A]_1\cdot[B]_2\stackrel{?}{=}[\alpha]_1[\beta]_2+[C]_1\cdot[G]_2
$$

## Preventing forgeries part 2: separating public and private inputs with $[\gamma]_2$ and $[\delta]_2$

The algorithm does not change substantially when we make a portion of the witness public.

Recall how we split the QAP into the portions computed by the verifier (the public inputs) and the prover (the private inputs):

$$
\sum_{i=1}^ma_iw_i(x)=\sum_{i=1}^\ell a_iw_i(x)+\sum_{i=\ell+1}^ma_iw_i(x)
$$

Here ℓ refers to the first ℓ elements of the witness vector which are public. A typical witness vector looks like [1, x1, x2, x3, …]. so in this case, ℓ will be 1, because element 0 and element 1 are public.

The only thing that changes from above is that [C] is computed over the private inputs, instead of the entire witness vector.

![https://static.wixstatic.com/media/935a00_43045a93932140238e68764e783c8a28~mv2.png/v1/fill/w_1435,h_163,al_c,lg_1,q_85,enc_auto/935a00_43045a93932140238e68764e783c8a28~mv2.png](https://static.wixstatic.com/media/935a00_43045a93932140238e68764e783c8a28~mv2.png/v1/fill/w_1435,h_163,al_c,lg_1,q_85,enc_auto/935a00_43045a93932140238e68764e783c8a28~mv2.png)

and the verifier has to compute the public portion using the elliptic curve points from the trusted setup. This is the new verification formula

![https://static.wixstatic.com/media/935a00_9d12c14a692042868768bb0fcefdff3d~mv2.png/v1/fill/w_1480,h_214,al_c,lg_1,q_85,enc_auto/935a00_9d12c14a692042868768bb0fcefdff3d~mv2.png](https://static.wixstatic.com/media/935a00_9d12c14a692042868768bb0fcefdff3d~mv2.png/v1/fill/w_1480,h_214,al_c,lg_1,q_85,enc_auto/935a00_9d12c14a692042868768bb0fcefdff3d~mv2.png)

## Algorithm with $[\gamma]$ and $[\delta]$

Trusted setup

$$
\begin{align*}
[\Theta_i]_1&=u_i(\tau)[G_1]  &\text{for i = 1 to m} \\

[\Omega_i]_2&=v_i(\tau)[G_2] &\text{for i = 1 to m} \\

[\Psi_i]_1 &= \gamma^{-1}(\alpha v_i(x)+\beta u_i(x)+w_i(\tau))[G_1]&\text{for i = 1 to ℓ}\\

[\Zeta_i]_1 &= \delta^{-1}(\alpha v_i(x)+\beta u_i(x)+w_i(\tau))[G_1]&\text{for i = ℓ+1 to m}\\

\end{align*}
$$

The prover’s algorithm:

The verifier’s algorithm:

$$
[A]_1\cdot[B]_2\stackrel{?}{=}[\alpha]_1\cdot[\beta]_2+\sum_{i=1}^\ell a_i[\Psi_i]_1+[C]_1\cdot[\delta]_2
$$

## Algorithm with $r$ and $s$

Trusted setup

$$
\begin{align*}
[\Theta_i]_1&=u_i(\tau)[G_1]  &\text{for i = 1 to m} \\

[\Omega_i]_1&=v_i(\tau)[G_1] &\text{for i = 1 to m} \\

[\Omega_i]_2&=v_i(\tau)[G_2] &\text{for i = 1 to m} \\

[\Psi_i]_1 &= \gamma^{-1}(\alpha v_i(x)+\beta u_i(x)+w_i(\tau))[G_1]&\text{for i = 1 to ℓ}\\

[\Zeta_i]_1 &= \delta^{-1}(\alpha v_i(x)+\beta u_i(x)+w_i(\tau))[G_1]&\text{for i = ℓ+1 to m}\\

[\Tau_i]_1 &= \delta^{-1}\tau^it(\tau)[G_1]&\text{for i = 0 to d}\\

\end{align*}
$$

Trusted setup publishes $(\mathbf{\Theta}_1,\mathbf{\Omega_1},\mathbf{\Omega}_2,\mathbf{\Psi}_1,\mathbf{\Zeta}_1,\mathbf{\Tau}_1,[\alpha]_1,[\beta]_1,[\beta]_2,[\gamma]_2,[\delta]_1,[\delta]_2)$

The prover does

$$
\begin{align*}[A]_1&=[\alpha]_1+\sum_{i=1}^ma_i [\Theta_i]_1 + r[\delta]_1\\ 

[B]_1&=[\beta]_1+\sum_{i=1}^ma_i[\Omega_i(x)]_1 + s[\delta]_1\\

[B]_2&=[\beta]_2+\sum_{i=1}^ma_i[\Omega_i(x)]_2 + s[\delta]_2\\

[C]_1&=\sum_{i=1}^ma_i[\Psi_i]_1+\sum_{i=0}^dh_i[T_i]_1+s[A]_1+r[B]_1-rs[\delta]_1\end{align*}
$$

Prover publishes $([A]_1,[B]_2,[C]_1)$

The verifier’s algorithm is unchanged

$$
[A]_1\cdot[B]_2\stackrel{?}{=}[\alpha]_1\cdot[\beta]_2*(\sum_{i=1}^\ell a_i[\Psi_i]_1)\cdot[\gamma]_2*[C]_1\cdot[\delta]_2
$$

## Verifier’s algorithm in Python

```python
from py_ecc.bn128 import pairing, final_exponentiate, FQ12, eq, neg, multiply
from functools import reduce

def inner_product(scalars, points):
	return reduce(add, map(multiply, points, scalars))

def verify(A_1, B_2, C_1, alpha_1, beta_2, public_portion_witness, Psi_1, gamma_2, delta_2):
	return eq(
		final_exponentiate(
			pairing(B_2, neg(A_1)) *
			pairing(alpha_1, beta_2) *
			pairing(inner_product(public_portion_witness, Psi_1), gamma_2) *
			pairing(C_1, delta_2)
		),
		FQ12.one()
	)   
```
