# ComfyUI-Flux2Klein-Enhancer vs Qwen Image Edit 2511

**What Qwen Image Edit 2511 already does natively**

Qwen Image Edit 2511 is built on an MMDiT architecture and has meaningfully improved identity and subject preservation baked into the model itself — including better multi-person consistency and reduced image drift. So preservation is a trained behaviour: the model has learned, from its training data, to hold onto what shouldn't change when you give it an instruction. You don't need to explicitly wire up a reference stream — it figures out what to keep from the prompt and the input image together.

**Why the Klein Enhancer's approach doesn't directly apply**

The Klein Enhancer works because FLUX.2 Klein has a very specific, exposed architecture: the reference image is explicitly concatenated into the image stream as its own block of tokens, and the node hooks directly into the K/V vectors of that block to dial its influence up or down. That's a low-level, surgical handle that exists because of how Klein was designed.

Qwen's MMDiT architecture processes the source image differently — it's fed in more holistically alongside the instruction, not as a separately identifiable reference stream with its own token range you can isolate and scale. So there's no equivalent "reference token block" to hook into in the same way.

**Could something similar be built for Qwen?**

Partially. The approaches that *don't* depend on Klein's specific architecture could transfer:

- **Latent-space correction** (like the Identity Guidance node) — running a correction pass after each sampling step to pull the output back toward the source — would work on Qwen, since it operates outside the model on the latent output rather than inside the architecture.
- **Feature similarity matching** (like Identity Feature Transfer) — finding similar features between source and generation inside attention layers — is theoretically possible on any transformer, though you'd need to reverse-engineer where Qwen's attention hooks are.

What *wouldn't* transfer is the precise per-reference-stream K/V scaling, because Qwen simply doesn't have that separate reference token block to target.

**The bottom line**

Qwen handles preservation more implicitly through training and instruction following — it's a smarter black box. Klein exposes the mechanism explicitly but needs this node to make it controllable. They're solving the same problem from opposite ends: Qwen bakes it in, Klein exposes it raw. The enhancer-style control is more necessary for Klein precisely *because* Klein doesn't make it easy by default.
