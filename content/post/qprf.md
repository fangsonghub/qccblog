+++
title= "Quantum-secure Pseudorandom Permutations"
slug = "QuantumPRP"
date= "2017-06-05"
tags = [
    "quantum security",
    "psedurandom",
    "Feistel",
	]
weight= 10	
+++
  
This is a long due note concerning constructing **quantum-secure
pseudorandom permutations** (QPRP), a problem that has made my
collaborators and myself excited as well as disappointed a couple of
times over the past few years. In a way this makes it a perfect fit
for the debut post of this blog.

A while ago, [Mark Zhandry][MZ] observed in a [note][Zha16PRP] that
some existing constructions are immediately quantum-secure for simple
reasons, hence confirming the _existence_ of QPRPs. This post aims to:

1. give another simple observation that shows the _existence_ of
   QPRP. This is in fact an observation we had a long time ago, but
   probably only heard by a few colleagues over drinks.
2. give a plain summary of Zhandry's observation and some discussion. 
3. more importantly, depict the current status and the big questions
   that remain **unclear** about QPRP, beyond its mere _existence_. By
   sharing our (mostly failed) experience so far on approaching these
   problems, we would love to see more people bringing in new ideas.

I will explain briefly the basic concepts of PRPs and the challenges
of getting quantum-secure PRPs. You can safely skip it if you know
about (and probably have suffered from) the problem.

# PRPs and quantum security 

If you have heard about DES/AES, or more generally _block ciphers_,
you are already on board with _pesudorandom permutations_ (PRP), which
are just theoretical abstractions of secure block ciphers. Before
digging into more specifics, let me just mention that PRPs are a
fundamental primitive in modern cryptography, and likewise block
ciphers are the workhorse in practical cryptosystems.

## What is a PRP?

Roughly speaking, a PRP is a permutation that is _indistinguishable_
from a **truly random permutation** as far as any efficient observers
are concerned when the permutations are given as a _black-box_. More
precisely, let us work with the set \\( X = \\{0,1\\}^{2n}\\), and let
\\(\Pi = \\{\pi \\}\\) be the collection of permutations on \\(X\\),
i.e. the symmetric group $S\_X$. Then we consider a family of
permutations $\\{E\_k\\} \_{k \in \mathcal{K}} \subseteq \Pi $ indexed
by keys in a keyspace $\mathcal{K}$.

At the heart of the definition is a **distinguisher** $A$, which is an
algorithm that is given _oracle_-access to a primitive $f$. It asks
queries to $f$ and output one bit in the end, and is usually denoted
$A^f$. Now consider the difference in the probabilities that a
distinguisher $A$ outputs 1 after making $q$ queries in two cases:

$$\epsilon(q,\lambda) :=| \Pr\_{k\gets \mathcal{K}} [A^{E\_k,
E\_k^{-1} }() = 1] - \Pr\_{\pi \gets \Pi} [A^{\pi,\pi^{-1} }() = 1] |
\, .$$

*  $A$ has access to $E\_k$ and its inverse $E\_k^{-1}$ for a random
   key $k\gets \mathcal{K}$. 
*  $A$ has access to $\pi$ and its inverse $\pi^{-1}$ for a _truly
   random permutation_ $\pi \gets \Pi$. 

This difference $\epsilon(q,n)$ is usually dubbed the distinguishing
**advantage**. Then a PRP is family $\\{E\_k\\}$ so that the advantage
$\epsilon(q,n)$ is tiny (i.e., decreases faster than any inverse
polynomial in $n$) for any $A$ making $q = poly(n)$ queries (and
running in $poly(n)$ time). This is sometimes called a _strong_ PRP in
the sense that both the permutation and its inverse is available to a
distinguisher. If only the forward permutation is allowed, we call the
resulting family a standard PRP. In this post, a PRP means a strong
one unless otherwise specified.

If we consider more generally function families rather than just
permutations, we obtain a similar notion called _pseudorandom
functions_ (PRFs), which are a family of keyed-functions that are
indistinguishable from a _truly random function_.

## How to construct PRPs?

Both theoretical and practical constructions are based on the idea of
_iteration_, where a basic module is iterated with fresh randomness or
different sub-keys. The famous construction in theory is due
to [Luby&Rackoff][LR88] based on the elegant design of
the [Feistel][Feistel] network. $\Xi_f$ converts a function $f:
\\{0,1\\}^{n} \to \\{0,1\\}^{n}$ into a permutation on
$\\{0,1\\}^{2n}$ by the simple rule:

$$\Xi_f(x\_L,x\_R) = (y\_L,y\_R): y\_L = x\_R, y\_R = x\_L\oplus
f(x\_R)\, . $$ 

If we iterate the basic unit $r$ times, we get the $r$-round Feistel
network $\Xi\_{f\_1,\ldots,f\_r}$, and each $f\_i$ is called a _round_
function. Luby and Rackoff proved that 3-round Feistel is a standard
PRP, and 4-round is a (strong) PRP, when each of the round function is
a PRP with independent random keys. Then by a beautiful series of
results, we know the existence of **one-way functions**, functions
that are easy to compute but hard to invert **on-average**, would
imply pseudorandom generators (PRGs), which suffice for PRFs, and
ultimately give rise to PRPs. 

> one-way functions --[HILL]--> PRGs --[GMM]--> PRFs --[LubyRackoff]--> PPRs

The gist of Luby&Rackoff's proof, which is inherent in all most all
analyses, goes in two steps

1) Substituting independently truly random functions for PRFs in
every round. This is legit by a straightforward hybrid argument.

2) Proving **information-theoretically** that the resulting network is
indistinguishable from a truly random permutaion. For example, the
4-round Feistel is shown to be secure up to $\sim \sqrt{2^{2n}}$ queries
against any, possibly **computationally** unbounded, distinguishers.

There are various followups and modern constructions
(e.g., [this][HR10] and [this][MRS09]). Essentially all analysis
proceeds in this way, and a lot of efforts have been on improving the
information-theoretical bound in step 2).

## What about quantum security? 

Everything sounds sweat so far. But the new player
of [quantum computers][QC] will totally change the landscape. You may
have heard about Shor's ground breaking poly-time quantum algorithm
for factorization, which will [smash down][Moscatalk] almost the entire
security infrastructure on the Internet. In current setting, the issue
comes from the unique feature of quantum **superposition**. For a
**quantum** distinguisher, we need to consider "superposed" queries:

$$\sum\_{x\in X}\alpha\_x |x,0\rangle \stackrel{f}{\mapsto} \sum\_{x
\in X} \alpha_x |x, f(x)\rangle \, .$$

We call a PRP **quantum-secure** (QPRP), if it remains
indistinguishable against any efficient quantum distinguishers that
are equipped with quantum superposition access to the given
permutations.

To be clear, although it seems that in one quantum query, all function
values of $f$ is "contained" in the response state, the only way that
$A$ can read them out is by a quantum measurement, which will reveal
one $f(x)$ and then collapse the state. Therefore, it is not that a
quantum distinguisher can effortlessly learn $f$ entirely. Rather, the
real difficult is that the proof and analysis we had for classical
distinguishers may not **make sense** anymore. More specifically, if
we look at the two-step proof of Luby&Rackoff:

1') OK, if we use **quantum-secure** PRFs, which exist assuming
**quantum-secure** one-way functions as shown
by [Zhandry12][Zha12prf].

2') The classical _information-theoretical_ analysis becomes
completely inapplicable. In essence, the analysis goes by defining
"bad events" in the probability space, which captures the possibility
of seeing a difference in the case of the Feistel construction or a
truly random permutation. Bounding the probability of these "bad
events" thus bounds the distinguishing advantage too. However, there
is no (or rather we do not know) sensible counterpart of such "bad
events" when dealing with quantum distinguishers. This is where the
challenge goes wild, mainly due to our lacking of tools in reasoning
about quantum attackers.

### 3-round Feistel broken 

To make things worse, it was shown that [3-round][3rbreak] Feistel is
**broken** by quantum distinguishers using Simon's algorithm, the
inspiration of Shor's algorithm. While this attack does not seem to
generalize to 4 rounds, we did not know if 4-round Feistel is
quantum-secure (even in the standard sense i.e., without inverse
permutations), or how many rounds will make it quantum-secure if any
at all. In fact, whether there exists a QPRP, by any possible
construction under reasonable assumptions, was in doubt.

# Existence of QPRPs

That was the status when we ([Andrew Childs][AC], [Zhengfeng Ji][ZJ]
and myself, all at [IQC][IQC] then) started thinking about this
problem. We were optimistic and decided to salvage the Luby-Rackoff
proof against quantum attacks. Out of various attempts, one approach
based on rapid mixing of random walks seemed promising and left with
us an observation that shows the _existence_ of QPRP, which we thought
insignificant back then (early 2014). None of us had imagined that it
was till last summer (2016) that we resumed on this hanging problem,
with a fresh mind joining force - [Shih-Han Hung][SHH], Andrew's Ph.D
student at UMD. A few months later, the three of us were proven stupid
by Shih-Han that the mixing approach would very likely fail on
Balanced Feistel based on a counting argument. Let me come back to
this later, and let good news come first.

## Our observation: quantum PRPs exist based on Rapid Mixing

What we observed in early 2014 was that at least _existence_ of QPRPs
should not bother us anymore. Basically one can view any iterated
construction (e.g. $\Xi\_{f\_1,\ldots, f\_r}$) as an $r$-step random
walk on the symmetric group $S\_X$. Namely we start from the identity
permutation, and then the random choice of round function $f\_i$
determines the transition at each step. Suppose that the walk has
uniform [stationary distribution][sdist] and it converges up to
statistical error $\delta$ within $r$ steps. This will imply that the
resulting permutation $\Xi_{f\_1,\ldots, f\_r}$ is the _same_ as
drawing a truly random permutation, except with error at most
$\delta$. Therefore, **information-theoretically**, no adversary
(classical nor quantum) can tell a difference regardless of the number
of queries and how much time s/he spends on it. This still holds if
both $\pi$ and $\pi^{-1}$ are given to the adversary. This would
justify step 2) against quantum distinguishers. Everything here can be
made formal. This idea is not new at all (e.g., [Morris08]).

<p><a
href="https://commons.wikimedia.org/wiki/File:Random_walk_25000_not_animated.svg#/media/File:Random_walk_25000_not_animated.svg"><img height="180" width="180"
src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/Random_walk_25000_not_animated.svg/1200px-Random_walk_25000_not_animated.svg.png"
alt="Random walk 25000 not animated.svg"></a></p>
<p style="text-align:center">By <a
href="//commons.wikimedia.org/wiki/User:Nl74" title="User:Nl74">László
Németh</a> - <a
href="http://creativecommons.org/publicdomain/zero/1.0/deed.en"
title="Creative Commons Zero, Public Domain Dedication">CC0</a>, <a
href="https://commons.wikimedia.org/w/index.php?curid=27964423">Link</a></p>



Next, as said earlier, one can replace truly random functions by
quantum-secure PRFs in step 1), and this results in permutations that
are indistinguishable from a truly random permutation against
_poly-time_ quantum adversaries making _poly_-many queries.

The question then becomes, do we have constructions that mix over
$S\_X$ rapidly? Luckily a variation on LubyRackoff construction called
the **maximally-unbalanced** Feistel (Max-UBF), which corresponds
to [Thorp shuffle][Morris08] in the study of card shuffling, has been
shown to **mix rapidly to uniform**. Instead of evenly divide the
input string into two halves in the Luby-Rackoff construction, which we
may call **Balanced Feistel** (BF) hereafter, Max-UBF applies random
round function $f\_i:\\{0,1\\}^{2n-1} \to \\{0,1\\}$ on all but the
first bit and XOR the one-bit output to the first bit of input. Namely
one-round Max-UBF goes as follows:

$$ (x\_1,x\_{2,\ldots,2n}) \mapsto (x\_{2,\ldots, 2n}, f(x\_{2,\ldots,
2n})\oplus x\_1) \, . $$

Therefore, we have

>  **Theorem**. Assuming existence of quantum-secure one-way functions, there exist quantum-secure pseudorandom permutations.

This is all too easy, isn't it? We thought so too back then, and were
reluctant to write it up. What we wanted to answer is: 

> does LubyRackoff construction (Balanced Feistel) also mix rapidly?

Searching its answer ended up in the negative message found by
Shih-Han. But again, let me defer it till later.

### A word of caution

The idea of random walk is very natural, so natural as we sometimes
fail to recognize what "walk" we are indeed taking. We stress that we
are considering a random walk over the permutations (each node is a
permutation and the edge connects nodes that are achievable by
composing one-round Feistel with some round function $f$). 

In contrast, one may also consider a walk over the **strings**
$\\{0,1\\}^{2n}$. Namely take an input string $x$, it will get
permuted to a new string $ x^{\prime}$ after sending it through the
Feistel network. We can analyze how fast, if it does, the string
becomes **uniformly** distributed. In cryptographic terms, this shows
(standard, w.o. inverse permutation) indistinguishability against a
single-query, **non-adaptive** (i.e., query $x$ determined in advance)
distinguisher. We can generalize this idea to analyze how a $q$-tuple
$(x\_1,\ldots,x\_q)$ evolves, and this would establish $q$-query,
non-adaptive security. Indeed, this is the approach many of the
classical works on PRPs take, and this walk on strings is sometimes
called _projective_ walks. It is often easier to analyze than the walk
on the symmetric group. Classical cryptographers are also blessed that
non-adaptive security can be easily [amplified][MPR07] by various
forms of composition to achieve stronger security (e.g., adaptive,
strong PRP).

## Zhandry's observation: fully-secure PRPs are quantum-secure

In a recent note by [Zhandry][Zha16PRP], he noticed another
**sufficient** condition that can make a classically secure PRP also
quantum-secure. He called the constructions satisfying this property a
_full-domain Function-to-Permutation Converter_ (FD-FPC). He then
observed that some classical constructions are indeed FD-FPCs
(e.g., [Mix-&-Cut][RY13] and [Sometimes-recurse][MR14] shuffles in the
design of format-preserving encryption). Then similarly once the
random functions in FD-FPCs are replaced by quantum-secure PRFs, one
gets quantum-secure PRPs.

The formal definition and analysis in Zhandry's note are somehow
technical and not so easy to grasp. But the key idea is extremely
simple. FD-FPC is essentially a notion called _full security_ in the
study of format-preserving encryption. Basically it requires that even
if the distinguisher queires the **entire domain** of a given oracle
$\sigma$ drawn according to some distribution, a distinguisher still
cannot tell it apart from a truly random permutation. Clearly such a
distinguisher can completely reconstruct the permutation $\sigma$, and
hence quantum superposition queries do not add any power. Therefore
quantum-secure follows trivially.

## Reflection on the existence of QPRPs

Well, both observations indicate that _existence_ of QPRPs is in fact
easy to attain, given the hard work in the literature. Let's
reflect on them a bit, before we ask what's next.

### How about _almost_ FD-FPCs?

Zhandry's FD-FPC (i.e. full security) is very stringent and in some
sense captures **perfectly** indistinguishability. Is it possible to
loose it slightly? For instance, consider an notion of
$\epsilon$-**almost** FD-FPC where we require that after querying a
construction $\sigma$ on all but $N^\epsilon$ elements of the domain
of size $N$, it is still difficult for the distinguisher to tell it
from a truly random permutation. It seems to me that, by direct
statistical analysis or some standard techniques in quantum query
complexity (e.g., [BBBV hybrid method][BBBV97]), this notion also
implies quantum security. In fact, Zhandry's definition FD-FPC may
already entails almost FD-FPC when very small $\epsilon$ is concerned.

### Rapid mixing implies (almost) fully-secure

An immediate application about the almost FD-FPC is that rapid mixing
implies almost FD-FPC. This is because rapid mixing means that if one
draws a permutation from some construction $\Xi$ or uniformly random,
they get literally the **same** permutation with the same probability
except with some negligible error. Of course learning the entire truth
table does not tell anything, because the distinguisher is looking at
the same permutation! 

In other words, FD-FPC was considering a special family of
distinguishers who query (almost) the entire domain and make a
decision, whereas rapid mixing ensures security for any
distinguishers. 

# What now? 

Well, maybe we just need to be happy now. But we (maybe just me) are
so annoyed that no quantum security has been said about the original
LubyRackoff's Balanced Feistel (BF) construction. 

> how many round are enough, if any, to make the LubyRackoff Balanced Feistel quantum-secure?

First things off, Zhandry's FD-FPC approach does not help here,
because BF is **NOT** FD-FPC, not even close to almost FD-FPC. It is
known that its security cannot go beyond the birthday bound, i.e.,
$\sqrt{2^{2n}}$ many queries suffice to break it.

How about the rapid mixing approach? Restating the question we
mentioned earlier 

> does LubyRackoff construction (Balanced Feistel) also mix rapidly?

We were very hopeful. First thing to notice is that balanced Feistel
networks only produce _even_ permutations, i.e., they generate the
alternating group $A_{X}$ as shown by [Evan&Goldreich][EG83] several
decades ago. But this is not a problem, since we can show that a
uniformly random _even_ permutation is indistinguishable from a
uniformly random permutation. So it suffices to have BF mix well in
$A_X$.  We tried several techniques (spectral analysis, coupling
argument as Morris used to show the mixing
of [Max-Unbalanced Feistel][Morris08]), but were not successful.

Then came the embarrassing moment. Shih-Han pointed to us that there
is just not enough **randomness** or freedom to produce all even
permutations by BF in poly-many rounds. A simple criterion that
somehow sneaked out of our eyes. Think about how many possible
permutations one round BF is able to generate -- the number of
possible round functions, which is $a = 2^{n2^n}$. So $r$-round
iteration at most gives you $(2^{n2^n})^r = 2^{r n \cdot 2^n}$ many
permutations. But what is the size of alternating group of $2n$-bit
strings? It is $2^{2n}! /2 \approx 2^{2n\cdot 2^{2n}}$. Clearly $r$ needs
to be at least $\Omega(2^n)$ to make ends meet. 

As a sanity check, the Max-Unbalanced Feistel does not suffer from
this, because there are $2^{2^{2n-1}}$ possible functions each round,
and roughly $2n$ rounds will generate the entire symmetric
group. Indeed Morris showed $O(n^3)$ mixing time.

Is this approach totally doomed? One may identify a subgroup in $A_X$
so that BF mixes rapidly on it and meanwhile a random element there
remains indistinguishable from being truly random. But we do not see a
clear route at present. 

## Open question: efficient QPRPs

Maybe we (I) should not be too obsessed by balanced Feistel. The more
significant question is probably how to get more efficient QPRPs. The
QPRPs identified so far (e.g., Max-UBF, Mix-Cut Shuffle) need linear
or polynomially many round in $n$. This may be acceptable on small
domain (e.g., constant $n$), but clearly too expensive for large
domains.

> can we construct more efficient QPRPs? e.g., constant number of iterations and calls of PRFs?

Any ideas? Comments welcome.

[AC]: https://www.cs.umd.edu/~amchilds/
[ZJ]: https://scholar.google.ca/citations?user=2uXdu7AAAAAJ
[IQC]: https://uwaterloo.ca/institute-for-quantum-computing/
[SHH]: http://quics.umd.edu/people/shih-han-hung
[MZ]: https://www.cs.princeton.edu/~mzhandry/
[Zha16PRP]: https://eprint.iacr.org/2016/1076
[LR88]: http://epubs.siam.org/doi/abs/10.1137/0217022
[Feistel]: https://simple.wikipedia.org/wiki/Feistel_cipher
[Zha12prf]: http://eprint.iacr.org/2012/182
[QC]: http://www-03.ibm.com/press/us/en/pressrelease/52403.wss 
[Moscatalk]: https://www.youtube.com/watch?v=vipU_-QGoOg&feature=youtu.be&list=PLUz_4vZOI0H0nfczvYk2C_UbE_BMs8cpY
[3rbreak]: http://ieeexplore.ieee.org/document/5513654/
[sdist]: https://en.wikipedia.org/wiki/Stationary_distribution
[Morris08]: http://epubs.siam.org/doi/abs/10.1137/050636231?journalCode=smjcat
[EG83]: http://math.boisestate.edu/~liljanab/MATH509/DESAlternatingGroup.pdf
[MPR07]: http://eprint.iacr.org/2006/456
[RY13]: https://www.iacr.org/cryptodb/data/paper.php?pubkey=24631
[MR14]: https://eprint.iacr.org/2013/560
[BBBV97]: https://arxiv.org/abs/quant-ph/9701001
[HR10]: http://eprint.iacr.org/2010/301
[MRS09]: http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.149.6371
