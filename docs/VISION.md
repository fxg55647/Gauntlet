# Gauntlet — Vision & Theory

This document explains why Gauntlet is built the way it is and where it is going.

---

# Part I — Theory

## Why in-model defence doesn't work

### "Don't think about the elephant"

Tell someone not to think about an elephant and they immediately think about one. The instruction activates exactly what it tries to suppress. An LLM instructed to "ignore" a prompt processes it regardless — that is how attention works. This is not a weakness to be patched with better prompting. It is a structural property of the architecture.

### Marking your own homework

Asking an LLM to detect whether its own input is malicious is like asking a student to mark their own homework. The evaluator and the subject share the same blind spots, the same biases, the same failure modes. Objective evaluation is structurally impossible from inside the same process.

### The 286 analogy

In the 1980s, security was bolted onto the processor — protected mode added to the same chip that ran everything else. It never worked well. The solution was a separate component: MMU for memory, TPM for security, GPU for graphics. Each component does one thing, does it well, and cannot be compromised by failures in the others.

The LLM field is repeating the same mistake. Billions are being spent teaching models to be "more secure" — adding protected mode to the same chip. Gauntlet applies the lesson that took hardware architecture a decade to learn: **security belongs in a separate component.**

---

## The attacker's impossible triangle

A successful injection requires all three simultaneously:

```
C > θ_model      — model understands the instruction
D < θ_gauntlet   — detection fails across all modules
R > θ_iteration  — survives dynamic multi-pass analysis
```

These are in direct conflict. Maximising coherence increases detectability. Maximising concealment destroys coherence. Surviving iteration requires exposing structural signals.

As analysis passes increase, θ_gauntlet and θ_iteration rise monotonically while C remains fixed. The feasible region shrinks with each pass. In practice, 2–3 passes already collapse it to a point where the input is either too incoherent to execute or too structured to evade detection.

**Attacks require coherent structure. Detection does not.**

---

## The attacker's arsenal is finite

Language is finite. Every obfuscation technique, every roleplay bypass, every encoding trick occupies a region of the semantic space. Each time a new technique is detected and added to the network, that region is closed. The attacker must find a new technique. The defender adds one rule.

Eventually the attacker's remaining space is so constrained that any input coherent enough to work as an injection is detectable. The endpoint is not reached by training a better model — it is reached by systematically exhausting the space of viable attacks.

---

## Why now

- ZK-proofs are practical today — not five years ago
- LLM agents are proliferating faster than defences
- Token costs are low enough that 2–3 analysis passes cost fractions of a cent
- No one has built the federated layer yet — the window is open before large players close it with proprietary solutions

The field has spent years trying to make models more secure from the inside. Gauntlet is the external layer the field has been missing.

---

# Part II — Possible Features

*Ordered from near-term to speculative. Earlier items are close to current architecture. Later items require significant new infrastructure.*

---

## Output analysis *(near-term)*

Current Gauntlet analyses inputs. An injection that passes input analysis may still reveal itself in the output. Gauntlet can scan generated responses for:

- Goal drift — response diverges from the stated task
- Anomalous content — information that should not appear given the input
- Code vulnerabilities — insecure patterns in generated code (CodeShield)

This closes the loop: input is checked before the model, output is checked after. An injection that survives the input layer still has to survive the output layer.

---

## LLM operator *(near-term)*

Gauntlet produces detailed logs: which modules triggered, at what scores, on what input types, with what false positive rates. A human operator tunes the pipeline based on this data. A future LLM operator does it automatically.

The LLM operator reads logs periodically and proposes or applies configuration changes:

- *"pg2 triggered 847 times this week but pidd confirmed only 12 — pg2 threshold may be too low"*
- *"false positive rate spiked after new product launch — benign_precision weight should increase"*

`advisory` — operator proposes, human approves.
`autonomous` — operator adjusts within defined bounds.

The yksisuuntainen principle applies: the operator can tighten configuration, never loosen it below the deployment minimum.

---

## Social engineering defence *(medium-term)*

Prompt injection and social engineering are the same problem at different levels. Both attempt to install a foreign intent into a trusted process. Gauntlet's structural analysis applies to both.

Social engineering relies on urgency, authority, sympathy, and scarcity. These are detectable patterns. Critically: the more pressure an input applies, the stronger the signal. A message that creates extreme urgency and requests an irreversible action is not more likely to be legitimate — it is more likely to be an attack.

**The attacker's most effective weapon is also their strongest signal.**

In guardian mode, Gauntlet analyses incoming communications on behalf of a human user — not just inputs to an LLM. Gauntlet does not block the message. It annotates it: *"This message contains urgency signals combined with a financial request. Risk: high."* The human decides. Gauntlet informs.

---

## Multimodal extension *(medium-term)*

The structural disarming principle is not limited to text. Any modality that requires coherent structure to carry intent can be defended with the same approach.

- **Image** — patch permutation instead of token shuffle
- **Audio** — segment reordering
- **Documents** — structural decomposition
- **Video** — frame-level analysis

Version roadmap: `v1` text → `v2` text + document structure → `v3` multimodal.

---

## Federated threat network *(long-term)*

No single operator can cover all languages, dialects, encodings, and cultural contexts. The federated network closes this gap: instances share abstract attack structures — not raw content — so every participant benefits from every discovery.

The network can only tighten protection. No external signal can lower an instance's protection level below its local baseline. An attacker who floods the network with false signals drives all instances toward maximum scrutiny — the worst-case outcome of a poisoning attempt is degraded availability, not a security breach.

Instances share the *structure* of an attack, not the content. Zero-knowledge proofs allow an instance to prove it has observed a pattern matching a known attack class without revealing the input itself.

When multiple instances simultaneously report a novel attack pattern, the network enters elevated threat mode automatically. The analogy is an immune system that learns at population level.

Not all threat intelligence is shareable. A three-ring model:

- **Public core** — open to all, basic patterns, low barrier
- **Private consortia** — banks, governments, sector-specific groups
- **Bridge layer** — sufficiently anonymised signals graduate from private to public

---

## Blockchain trust model *(speculative)*

The federated network requires trust infrastructure. Blockchain primitives map naturally onto the requirements:

- **Staking + slashing** — instances post a bond. False signals result in slashing.
- **On-chain reputation** — decaying trust score, inactive or inaccurate instances lose voting weight.
- **Threshold voting** — a signal requires confirmation from multiple instances before propagating.
- **Smart contract** — enforces additive-only protection level. Cannot be configured to lower scrutiny.
- **ZK-proofs** — share attack signatures without exposing content.
- **On-chain forensics** — honeypot data recorded immutably for later analysis.

Public and private rings can coexist: private consortia enforce their own trust rules, public core is open. Sufficiently anonymised signals bridge between them.
