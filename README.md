# ComfyUI-Flux2Klein-Enhancer
notes about spatial_fade control control map input

## spatial_fade

From the **Ref Latent Controller** parameter table, `spatial_fade` applies a directional gradient to the reference strength across the image — instead of the reference influencing the whole image equally, it fades in or out across a chosen axis.

The options are:
- **center_out** — strong in the middle, fades toward the edges
- **edges_out** — strong at the edges, fades toward the center
- **top_down** — fades from top to bottom
- **left_right** — fades left to right

The `spatial_fade_strength` slider controls how extreme that gradient is.

In practice, this lets you say something like *"preserve the reference structure strongly in the center but give the prompt more freedom toward the edges"* — useful if your subject is centered and you want the background to feel more generated, for example. It's applied per-token in the attention layer, so the fade maps onto the actual spatial regions of the image rather than being a post-process blend.

## use map to control spatial_fade

The **Ref Latent Controller** itself only supports the preset gradient options — there's no mask input on that node.

However, there's already a separate node in the suite that does exactly this: **FLUX.2 Klein Mask Ref Controller**. It takes a standard ComfyUI mask and uses it to spatially control the reference latent strength across the image — masked areas stay true to the reference structure while unmasked areas are freed up for the prompt. It also has a `feather` parameter to soften the boundary, and a `channel_mode` to target either layout/structure channels or texture/detail channels independently.

The key difference from spatial fade is intent: spatial fade applies a *smooth mathematical gradient* (always symmetric, always the same shape), whereas the mask node lets you draw an *arbitrary shape* — useful when your subject isn't centered or doesn't align with any clean directional gradient.

---

**Could the Ref Latent Controller be modified to accept a mask directly?**

Yes, fairly straightforwardly. The node already computes a per-token strength map to apply to K/V vectors. A mask would just be an alternative way to generate that map — resize the mask to match the latent spatial dimensions, flatten it into a token sequence, and use it as the strength multiplier instead of a preset gradient. The author has clearly already done this logic in the Mask Ref Controller, so it would largely be a matter of merging the mask-resizing code from that node into the Ref Latent Controller and adding a mask as an optional input. If the mask is absent, it falls back to the existing spatial fade behaviour.
