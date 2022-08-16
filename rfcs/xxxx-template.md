- Start Date: 2022-08-16
- RFC PR: (leave this empty)

# Summary

Add the option to convert all uploaded images to WebP or AVIF instead of just resizing them.  This would be done via a **opt-in** parameter in `./config/plugins.js`. 

# Motivation

WebP and AVIF are superior formats for the web and are growing in support very fast.

It got to a point where some projects could decide to only work with these formats to improuve the user experience without adding engeneering complexity or hosting costs.

One could even argue that this could make your service more ecologically friendly, by reducing the data transfered on every page load.

### How much better?

"WebP typically achieves an average of 30% more compression than JPEG" -- [Google WebP FAQ](https://developers.google.com/speed/webp/faq#:~:text=WebP%20typically%20achieves%20an%20average,help%20make%20the%20web%20faster.)

- [Comparative study of WebP, JPEG and JPEG 2000, September 2010 - by Google](https://developers.google.com/speed/webp/docs/c_study)
- [WebP Compression Study - by Google](https://developers.google.com/speed/webp/docs/webp_studyhttps://developers.google.com/speed/webp/docs/webp_studyhttps://developers.google.com/speed/webp/docs/webp_study)
- [WebP vs AVIF - by avif.org](https://avif.io/blog/comparisons/avif-vs-webp/)

TLDR: WebP competes with JPEG on image size. AVIF can have a great image quality without being too heavy.

### How good is browser support?

Look for yourself:
- [https://caniuse.com/webp](https://caniuse.com/webp)
- [https://caniuse.com/avif](https://caniuse.com/avif)

As of Agust 2022 of the major browsers that don't support it:
- WebP: Only IE does not support it
- AVIF: Edge and other smaller browsers have no support, and Safari desktop has partial support

# Detailed design

Describe the proposal in detail:

- Explaining the design so that someone who knows Strapi can understand and someone who works on it can implement the proposal. 
- Think about edge cases and include examples.

# Example

If the proposal requires changes to the current API or the creation of new ones, add a basic code example.

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity
- Workload of implementation
- Can this be implemented outside of Strapi's core packages
- How does this proposal integrate with the current features being implemented
- Cost of migrating existing Strapi applications (is it a breaking change?)
- Does implementing this proposal mean reworking teaching resources (videos, tutorials, documentation)?

# Alternatives

What are the alternatives?

# Unresolved questions

Optional, but suggested for first draft proposals. What parts of the design are still TBD(To be defined)?
