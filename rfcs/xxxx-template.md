- Start Date: 2022-01-05
- RFC PR: 

# Summary

Add more InjectionZones in and config options for the Admin UI.

Add some InjectionZones to a few different areas on the sidebar, and on the dashboard.

Possibly add some new config options to enable changing some wordings like "Strapi Dashboard - Workplace" to "Clients Brand".

# Example

I don't believe this would require and changes to the API or creation of new APIs. Just would be using the already-made InjectionZone component along with adding some new config options.

# Motivation

Me, my clients, and some of my coworkers have all brought up that they would love if they could modify, add to, or remove the dashboard screen entirely.

My clients have also asked about adding their brand's title/name to the dashboard text and removing "workplace".

There is currently no way to adjust the default dashboard, or it's "page title" in the menu.

I believe adding some injection zones, and some basic config options to help modify those desired areas (dashboard, and text on dashboard menu item) would allow more clients to be accepting of using Strapi.

# Detailed design

- Add some InjectionZone components below the current dashboard items.
- Add config option to remove social panel on dashboard.
- Add config option to remove other default panel on dashboard.

or

- Add InjectionZone to dashboard with the default contents being the current dashboard. (if this is possible, havent looked into the InjectionZone component that much yet)

# Tradeoffs

Pros:
- Give developers and brands a better UX by allowing them more configuration over the CMS.

Cons:
- Giving ability to lose a small amount of Strapi's brand - especially in the case of changing the "Strapi Dashboard" text.

# Alternatives

No alternatives I can think of.

# Unresolved questions

