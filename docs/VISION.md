# Gauntlet — Vision & Theory

This document explains why Gauntlet is built the way it is, where it is going, and the theoretical foundation behind its design.

---

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

### The attacker's arsenal is finite

Language is finite. Every obfuscation technique, every roleplay bypass, every encoding trick occupies a region of the semantic space. Each time a new technique is detected and added to the network, that region is closed. The attacker must find a new technique. The defender adds one rule.

Eventually the attacker's remaining space is so constrained that any input coherent enough to work as an injection is detectable. The endpoint is not reached by training a better model — it is reached by systematically exhausting the space of viable attacks.

---

## The federated threat network

### Collective immunity

No single operator can cover all languages, dialects, encodings, and cultural contexts. A Spanish-language attack pattern discovered by one instance is unknown to a Finnish deployment. The federated network closes this gap: instances share abstract attack structures — not raw content — so every participant benefits from every discovery.

### Additive-only signals

The network can only tighten protection. No external signal can lower an instance's protection level below its local baseline. An attacker who floods the network with false signals drives all instances toward maximum scrutiny — the worst-case outcome of a poisoning attempt is degraded availability, not a security breach.

### Sharing without exposure

Instances share the *structure* of an attack, not the content. Zero-knowledge proofs allow an instance to prove it has observed a pattern matching a known attack class without revealing the input itself. Privacy is preserved. The network learns anyway.

### Trust model

- **Staking** — instances post a bond to join. False signals result in slashing.
- **Decaying trust score** — inactive or inaccurate instances lose voting weight over time.
- **Consensus threshold** — a signal requires confirmation from multiple instances before propagating.
- **Cryptographic identity** — every signal is signed. Unsigned signals are discarded.
- **Three-layer verification** — economic (staking), behavioural (track record), operational (token reporting).

### Public and private rings

Not all threat intelligence is shareable. A three-ring model:

- **Public core** — open to all, basic patterns, low barrier
- **Private consortia** — banks, governments, sector-specific groups sharing sensitive signals within a trusted boundary
- **Bridge layer** — sufficiently anonymised signals can graduate from private to public

This mirrors how traditional threat intelligence (ISACs) works, but with cryptographic enforcement instead of legal agreements.

### Epidemic response

When multiple instances simultaneously report a novel attack pattern, the network enters elevated threat mode. All instances tighten their analysis automatically. When the pattern subsides, they return to baseline. The analogy is an immune system that learns at population level.

---

## Social engineering defence

Prompt injection and social engineering are the same problem at different levels. Both attempt to install a foreign intent into a trusted process. Gauntlet's structural analysis applies to both.

### Psychological triggers as signals

Social engineering relies on urgency, authority, sympathy, and scarcity. These are detectable patterns. Critically: the more pressure an input applies, the stronger the signal. A message that creates extreme urgency and requests an irreversible action is not more likely to be legitimate — it is more likely to be an attack.

**The attacker's most effective weapon is also their strongest signal.**

### Guardian mode

In guardian mode, Gauntlet analyses incoming communications on behalf of a human user — not just inputs to an LLM. The same structural analysis that detects prompt injection detects romance scam patterns, fake authority impersonation, and social engineering scripts.

Gauntlet does not need to block the message. It can annotate it: *"This message contains urgency signals combined with a financial request. Risk: high."* The human decides. Gauntlet informs.

### Identity verification

Blockchain-based identity oracles allow Gauntlet to verify claimed identities cryptographically. A message claiming to be from a bank that cannot produce a valid on-chain signature is flagged regardless of how convincing the content is.

---

## Bidirectional analysis

Current Gauntlet analyses inputs. Future Gauntlet analyses in both directions.

### Output layer

An injection that passes input analysis may still reveal itself in the output. Gauntlet can scan generated responses for:
- Goal drift — response diverges from the stated task
- Anomalous content — information that should not appear given the input
- Code vulnerabilities — insecure patterns in generated code (CodeShield)

### Reading the model's mind

Some models expose internal signals via their API — logprobs, token probabilities, finish reasons. A model that has been successfully injected behaves differently from one that is processing a normal input. An injected model returning an unusually confident verdict on a borderline input is itself a signal.

A small local classifier reads these signals passively, with no added latency. Optionally, a second LLM can be given the logprobs and asked to reason about what they reveal — one model judges the input, another judges the judge.

This applies inside Gauntlet's own pipeline: when an LLM judge is used as an escalation step, its internal state is readable. An attacker who compromises the judge must also produce consistent logprob patterns — a significantly harder problem.

---

## LLM operator

Gauntlet produces detailed logs: which modules triggered, at what scores, on what input types, with what false positive rates. A human operator can tune the pipeline based on this data. A future LLM operator does this automatically.

The LLM operator reads logs periodically and proposes or applies configuration changes:

- *"pg2 triggered 847 times this week but pidd confirmed only 12 — pg2 threshold may be too low"*
- *"false positive rate spiked after new product launch — benign_precision weight should increase"*

**Two modes:**

`advisory` — operator proposes, human approves. Suitable for production environments where changes have business impact.

`autonomous` — operator adjusts parameters within defined bounds. Suitable when bounds are set conservatively.

The yksisuuntainen principle applies: the operator can tighten configuration, never loosen it below the deployment minimum.

Combined with the federated network, the LLM operator reads both local logs and network signals — optimising for the local deployment's specific traffic profile while benefiting from global threat intelligence.

---

## Multimodal extension

The structural disarming principle is not limited to text. Any modality that requires coherent structure to carry intent can be defended with the same approach.

- **Image** — patch permutation instead of token shuffle
- **Audio** — segment reordering
- **Documents** — structural decomposition
- **Video** — frame-level analysis

The federated network extends naturally: an instance that detects a novel image-based injection shares the abstract structural signature, not the image itself.

**Version roadmap:**

- `v1` — text
- `v2` — text + document structure
- `v3` — multimodal

---

## Why now

- ZK-proofs are practical today — not five years ago
- LLM agents are proliferating faster than defences
- Token costs are low enough that 2–3 analysis passes cost fractions of a cent
- No one has built the federated layer yet — the window is open before large players close it with proprietary solutions

The field has spent years trying to make models more secure from the inside. Gauntlet is the external layer the field has been missing.
