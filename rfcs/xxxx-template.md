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

As of Agust 2022 of the major browsers that don't support it:
- WebP: Only IE does not support it
- AVIF: Edge and other smaller browsers have no support, and Safari desktop has partial support

Sources:
- [https://caniuse.com/webp](https://caniuse.com/webp)
- [https://caniuse.com/avif](https://caniuse.com/avif)

# Detailed design

This could be done directly in [/packages/core/upload/server/services/image-manipulation.js](https://github.com/strapi/strapi/blob/master/packages/core/upload/server/services/image-manipulation.js) file.

A member of the community already had a prototype (no affiliation with him):
https://github.com/devgar/strapi/blob/243d914efeebfdb44b236af76689c30b73fed646/packages/strapi-plugin-upload/services/image-manipulation.js

Basically it would instruct `sharp` package to transform to `webp` or `avif`. This would **not** require an aditional dependency, not even a new import in the file.

# Example

Something like this:
```ts
// path: ./config/plugins.ts

export default ({ env }) => ({
  upload: {
    config: {
        convertAllImagesTo: 'webp' | 'avif'; // To be defined
    },
  },
});
```

# Tradeoffs

Even if I'm pretty confident that is a good decision for many of us, it is **opinionated**. This could be mitigated by adding more formats as targets like `png` or `jpeg`.

What potential tradeoffs are involved with this proposal.

- Complexity
- Workload of implementation
- Can this be implemented outside of Strapi's core packages
- How does this proposal integrate with the current features being implemented
- Cost of migrating existing Strapi applications (is it a breaking change?)
- Does implementing this proposal mean reworking teaching resources (videos, tutorials, documentation)?

# Alternatives

I guess this could be done in a plugin. But this would require replacing the official `strapi-plugin-update` for a community one. This plugin is quite big and important, I wouldn't recommend replacing it with a community-maintained plugin.

If is not in the official one, I wouldn't bother doing it.

# Unresolved questions

- Should it support many different formats (e.g. `png` and `jpeg`) or just work with `webp` and `avif` which are the ones that should be used? More formats would make it less opinionated but would add more maintenance, and may not be that relevant.
- How should the `./config/plugins.ts` look like?
