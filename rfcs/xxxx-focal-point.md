- Start Date: 2023-11-08
- RFC PR:

# Summary

We propose introducing a feature in Strapi that enables users to set a focal point on images directly from the media gallery.

# Motivation

As images are often utilized in resizable contexts, the absence of focal point information hinders the developer's ability to control cropping while preserving the most meaningful portion of the image. This has lead to content editors having to create multiple images, or designing a single piece of artwork with a centered subject that gets cropped on the sides.

Several members of the community expressed interest in this feature too: see [here](https://feedback.strapi.io/feature-requests/p/image-focal-point)

# Detailed design

We suggest the addition of a focal point picker to the `EditAssetDialog` component. This implementation could draw inspiration from Strapi's existing image cropping interface.

The `ActionRow` of the edit media dialog will have an added button that activates the set-focal-point mode. Upon activation, a viewfinder will appear for selecting the focal point on the image preview.

Upon confirmation of the focal point position, these coordinates will be stored within the file's data structure as a JSON object featuring x and y percentage values.

Any existing focal point coordinates will be visible in the image context info on the right side of the edit media dialog. Also, the focal point data will be made accessible within the attributes of any GraphQL file object.

# Example

Please refer to the attached sample PR for a hands-on understanding. Please note that our proposed focal point picker only reacts to click events at present, and not drag events. We've only updated the English translation as our priority was demonstrating the functionality. If this aligns with your project objectives, we can delve further into the specifics.

# Tradeoffs

From a coding perspective, the feature seems not complex.
However, it will constitute a breaking change as a DB migration would be necessary for existing Strapi applications.
On the bright side, it won't require any significant overhaul of existing teching resources as the feature is almost self-explanatory.

# Alternatives

Developing a plugin could be an alternative solution but could result in a less seamless user experience. This functionality seems most suited to the media gallery where content editors are able to set focal points as soon as they upload their artworks.

# Unresolved questions
