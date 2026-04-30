# ComfyUI-Flux2Klein-Enhancer overview

## How the Source Image Is "Kept" During Generation

### Step 1: The Reference Image Travels as Its Own Separate Stream

When you feed FLUX.2 Klein a reference/source image, it doesn't get blended into your text prompt. Instead, it gets encoded into its own block of data called a **reference latent** — essentially a compressed numeric description of the image. This latent is kept completely separate from the text conditioning, stored in metadata alongside it but never mixed in.

When the model starts generating, both your **noisy new image** and your **reference image** get broken into small patches and fed in *together* — literally concatenated into a single long sequence. The model sees both at once, side by side, which is how it knows what to preserve.

---

### Step 2: The Model Pays "Attention" to the Reference

Inside the model, there's a mechanism called attention — it's how different parts of the image "talk to each other" during generation. The reference image patches are sitting in this attention sequence right next to the new image patches. So the model can look over at the reference and say *"okay, this area of the new image should resemble that area."*

What the node does is directly dial up or down **how loudly the reference image speaks** during that attention process. Specifically, it scales the **K** (key) and **V** (value) vectors — think of K as "what the reference is about" and V as "what information it passes along." Multiply those up, and the reference shouts louder. Multiply them toward zero, and the reference is effectively ignored.

---

### Step 3: Preservation Modes Give You More Explicit Control

Beyond attention scaling, the node adds a second layer of control specifically for when you're *editing* an image and want to keep the original mostly intact:

- **Dampen mode**: Before any changes are applied, it reduces the strength of the modifications themselves. Less change is attempted in the first place. It's the most reliable approach.
- **Linear/Blend mode**: Applies the full modification, *then* mixes the result back with the original — like fading between two layers in Photoshop.
- **Hybrid**: Does both — dampens first, then blends.

The `edit_text_weight` parameter is the main knob here. Turn it below 1.0 and the prompt has less power to change things; turn it above 1.0 and the prompt takes over more aggressively.

---

### Step 4: Two "Identity" Nodes for Even Stronger Preservation

For face/object identity specifically, there are two bonus nodes that work at different levels:

- **Identity Guidance** — runs *after* each sampling step. The model generates freely, and then the result is nudged back toward the reference image. Think of it like a correction pass: *"you drifted too far, come back."* You control how hard it pulls, and when it stops (so the last 20% of steps can run free for final texture refinement).

- **Identity Feature Transfer** — works *inside* the attention layers themselves, during generation. It finds which parts of the new generation already look similar to the reference and reinforces those features before they get overwritten. It's a subtler, continuous influence rather than a blunt correction.
