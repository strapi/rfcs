- Start Date: 03-16-2022
- RFC PR: (leave this empty)

_Disclaimer_: This RFC is at an early stage and covers only high-level requirements. We are hoping the community can help us identify and spec the technical requirements. Thank you in advance for your help.

# Summary

Custom fields are a way to extend Strapi’s capabilities. They allow users to add new fields to content types for a better content edition experience (nicer display, enriched data, more actions, etc). 

# Examples

Based on all the input we got from the community, we identified a few key use cases we’d like to focus on first:

### Map

A content editor could directly search for a location on a map. Instead of entering latitude and longitude coordinates.

### Videos

Embedding a Youtube video would display a nice preview, which would make it easier to parse the content on a glimpse. Instead of having to click on the link to see what the video is about.

### Color picker

Select any color in any format (HEX, RGB, CMYK) directly in Strapi.

### URL

Fetch the data from a URL to show the favicon of the page, as well as the title. Instead of plain text.

### Shopify

A more complex is the one where a field would fetch external data. Products from a Shopify store for instance. And directly display them in the Content Manager.


# Motivation

Please make sure to explain the motivation for this proposal. 
It means explaining the use case(s) and the fonctional feature(s) this proposal is trying to solve. 

Try to only talk about the intent not the proposed solution here.

# Detailed design
![Installation steps](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d0643e00-04d0-49c8-b849-32881ce0097e/Untitled.png)

Describe the proposal in details:

- Explaining the design so that someone who knows Strapi can understand and someone who works on it can implement the proposal. 
- Think about edge-cases and include examples.

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity
- Work load of implementation
- Can this be implemented outside of Strapi's core packages
- How does this proposal integrate with the current features being implemented
- Cost of migrating existing Strapi applications (is it a breaking change?)
- Does implementing this proposal mean reworking teaching resources (videos, tutorials, documentations)?

# Alternatives

What are the alternatives?

# Unresolved questions

Optional, but suggested for first draft proposals. What parts of the design are still TBD(To be defined)?
