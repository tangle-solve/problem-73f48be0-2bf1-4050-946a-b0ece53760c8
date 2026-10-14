<!-- solve-problem:73f48be0-2bf1-4050-946a-b0ece53760c8 -->
# Tight MAD-DAG security bounds

A [Tangle Solve problem](https://solve.tangle.tools/problems/73f48be0-2bf1-4050-946a-b0ece53760c8).

## What is the problem?

MAD-DAG (Bar-Zur, Tamar, Eyal 2025) is a DAG-based blockchain protocol that deters MEV-style manipulation by making deviations *mutually assured destruction*-rational: a deviating miner/validator forfeits rewards that the reward scheme would otherwise grant. Security is certified by modeling the protocol plus attacker as a Markov Decision Process (MDP): states capture the observed DAG shape, actions are the attacker's possible block-withholding / equivocation strategies, and the value of the optimal policy is compared against honest rewards. If the optimal deviation gain is ≤ 0, the protocol is incentive-compatible for the given adversary fraction and network parameters.

The open problem: the current analysis uses a *conservative reward accounting* — it charges the attacker's value function pessimistically (e.g., granting the attacker only a subset of rewards, or bounding over unresolved DAG futures coarsely). This yields provable but loose bounds, excluding parameter regimes (higher adversary power, longer network delays, denser DAGs) that may actually be safe. The task is to close the gap between this tractable conservative MDP bound and the exact optimum of the full attack model, while keeping the certification computationally tractable (the exact state space is astronomically large). Concretely: characterize or tightly bound

  G* = max_policy [attacker reward under full model] − honest reward,

versus the currently certified lower-bound proxy Ĝ ≤ G*, and shrink G* − Ĝ to (near) zero for realistic parameter ranges, or prove the gap is inherent.

Note: the exact formalization of "full attack model" (which strategies are permitted, whether network asynchrony is modeled stochastically) must be extracted from the paper; the census record does not pin it down.

## Why does it matter?

Loose bounds force protocol designers to over-provision: pick lower adversary thresholds, fewer rewards, or sparser DAG parameters than necessary. A tight bound either certifies the protocol safe in wider regimes (directly usable parameter guidance) or exposes a hidden attack the conservative analysis masked — either outcome is publishable and consequential for MEV-resistant DAG ledgers (a currently hot deployment target, cf. Sui/Aptos-style DAG consensus). It also advances the general methodology of MDP-based incentive-compatibility certification for blockchains, which has been growing since the Bitcoin selfish-mining-as-MDP lineage.

## What's already known?

Assuming the census is accurate (verify against the actual paper):

- **MAD-DAG (arXiv:2511.21552)**: introduces the protocol, gives the MDP security analysis with conservative reward accounting, and explicitly lists tightening as future work. This is the source of the open gap.
- **Bar-Zur, Eyal, Tamar (2020, arXiv:2007.05614)**: prior joint work by the same group on incentive-compatibility of cross-chain/decentralized protocols (likely an earlier MDP-style rational-adversary analysis; verify which paper this ID corresponds to — the group's known 2020 work is MAD-HTLC).
- Standard machinery relevant here: selfish-mining MDP analyses (writing attacker-vs-honest-miner interactions as an MDP and solving for optimal policy), state-space reduction/abstraction for such MDPs, and abstract interpretation for sound over/under-approximation of value functions.
- The key prior observation: conservative accounting is sound (it never overestimates attacker gain) but can lose a large constant or even unbounded factor depending on how unresolved futures are bounded.

No tight result is known; the gap size has not been quantified.

## What would count as solving it?

Any of:
1. **Upper/lower match**: an algorithm computing G* within ε (for given adversary fraction, delay distribution, reward rule) together with a sound abstraction whose certified value Ĝ satisfies G* − Ĝ ≤ ε, with complexity polynomial-ish in DAG parameters — i.e., a tractable tight certification.
2. **Gap theorem**: a proof that any sound abstraction from a specified class must lose ≥ δ, i.e., the looseness is inherent — this "exposes hidden weakness" and closes the problem in the negative direction.
3. **Constructive attack**: a strategy in the full model beating the certified bound by a demonstrated margin (simulation + formal analysis), showing the conservative bound was masking real attacks.
4. Intermediate: a family of abstractions (e.g., state-space quotienting by DAG isomorphism + reward-dominance orders) with a proven per-class tightness ratio.

## How to start

1. Read MAD-DAG (arXiv:2511.21552) end-to-end; reconstruct the MDP: state definition, transition structure, reward accounting, and identify exactly which pessimistic simplifications constitute the "conservative" step. This is the single most important input — the census is vague precisely here.
2. Enumerate the sources of looseness (typically: (a) merging distinct DAG futures, (b) bounding unresolved rewards from below/above, (c) restricting the action space). For each, check whether a bisimulation-style quotient (state merging that provably preserves value) can replace the conservative merge.
3. Adapt known techniques from selfish-mining MDP analyses: value iteration over reduced states, and dominance arguments showing attacker-preferred states collapse.
4. Experimentally quantify the current gap by solving the exact MDP on small parameter instances (small adversary discount, small DAG depth) and comparing to the certified bound — this tells you where the real target is.
Main obstacle: state-space explosion in the exact model; that's why abstraction (abstract interpretation, quotients) is the census-suggested toolkit.

## Key references

- Bar-Zur, R., Tamar, A., Eyal, I. (2025). *MAD-DAG: Protecting Blockchain Consensus from MEV*. arXiv:2511.21552 — defines the protocol and the conservative MDP security bound whose looseness is the open problem. (verify existence/details; ID from census)
- Bar-Zur, R., Eyal, I., Tamar, A. (2020). arXiv:2007.05614 — earlier rational-adversary/MDP analysis by the same group; consult for the modeling conventions carried into MAD-DAG. (verify — this arXiv ID could not be confirmed and may correspond to a different paper by the group, e.g., MAD-HTLC)
- Sapirshtein, A., Sompolinsky, Y., Zohar, A. (2016). *Optimal Selfish Mining Strategies in Bitcoin*. FC 2016 — the canonical template for optimal-attacker MDP analysis with state-space reduction; methodologically the closest prior art. (verify relevance after reading MAD-DAG)
- Cousot, P., Cousot, R. — abstract interpretation literature, as flagged in the census for sound abstraction of the value function.

## Warnings

- The 2025 MAD-DAG citation and the 2020 arXiv ID come from a census record I cannot independently verify; treat both as leads, not facts, and re-derive all claims from the primary papers.
- "Full attack model" is ambiguous: clarify whether the attacker controls network delay, whether multiple rational attackers are considered (game vs. single-agent MDP), and the reward horizon/discounting. Different choices give different problems — pick one, state it.
- Common dead end: trying to solve the exact MDP without abstraction — hopeless combinatorially; the contribution must be in the abstraction's tightness proof, not raw computation.
- Soundness is non-negotiable: any "tightening" that could overestimate attacker power asymmetry in the wrong direction invalidates the certificate. Double-check the direction of every bound.

---
*`BCGT-000250` · unverified*

## Acceptance guidance

Any of:
1. **Upper/lower match**: an algorithm computing G* within ε (for given adversary fraction, delay distribution, reward rule) together with a sound abstraction whose certified value Ĝ satisfies G* − Ĝ ≤ ε, with complexity polynomial-ish in DAG parameters — i.e., a tractable tight certification.
2. **Gap theorem**: a proof that any sound abstraction from a specified class must lose ≥ δ, i.e., the looseness is inherent — this "exposes hidden weakness" and closes the problem in the negative direction.
3. **Constructive attack**: a strategy in the full model beating the certified bound by a demonstrated margin (simulation + formal analysis), showing the conservative bound was masking real attacks.
4. Intermediate: a family of abstractions (e.g., state-space quotienting by DAG isomorphism + reward-dominance orders) with a proven per-class tightness ratio.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/73f48be0-2bf1-4050-946a-b0ece53760c8?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000250.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000250)
