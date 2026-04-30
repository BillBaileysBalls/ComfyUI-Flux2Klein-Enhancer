## ComfyUI-Flux2Klein-Enhancer vs. IPAdapter Plus

IPAdapter works by injecting reference image features into the model's attention layers as a kind of soft style/appearance signal - essentially a temporary, image-based LoRA. It's designed to transfer *general appearance, style, or structure* from one or more reference images onto a fresh generation, and it works across many models because it sits on top of standard CLIP image embeddings.

This node is doing something more targeted and lower-level. FLUX.2 Klein natively takes a reference latent as part of its architecture - the reference image is already *inside* the model's image stream, not injected from outside. What this node does is expose and control that built-in mechanism, which FLUX Klein provides no native sliders for. The goal isn't style transfer across a generation - it's precise structural preservation of a specific source image during an edit. Less "make it look like this," more "don't let it stop looking like this."
