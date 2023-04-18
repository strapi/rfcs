- Start Date: (2023-04-18)
- RFC PR:

# Summary
Implement a category dropdown in the admin UI for both Collection & Single types.

# Motivation
The motivation behind this proposal is to simplify content organization and improve the user experience while managing content in the Strapi admin interface. Currently, users have to navigate through a long list of Collection and Single types, which can be time-consuming and cumbersome, especially when dealing with numerous content types. By introducing a category dropdown, users can easily group and filter their content types, making content management more efficient and user-friendly.

# Detailed design
The proposed solution includes the following design changes:

1. Add a category dropdown in the admin UI for both Collection & Single types, enabling users to filter content types based on the selected category.
2. Allow users to create, edit, and delete categories from the admin UI.
2. Automatically assign newly created content types to a default category (e.g., "Uncategorized") if no category is selected.
2. Enable users to change a content type's category through the content type's settings.
2. Display the category alongside the content type in the main admin UI.

Edge Cases:

- Handling empty categories: If all content types within a category are deleted or moved to another category, the empty category should still be displayed in the dropdown to allow users to assign new content types to it.
- Handling category deletion: If a category is deleted, all associated content types should be moved to the default category.

# Example
No changes to the current API are required for this proposal. The proposed changes are only related to the admin UI.

# Tradeoffs
Some tradeoffs to consider when implementing this proposal:

- Complexity: The addition of categories adds a new layer of complexity to the admin UI. However, the benefits of improved content organization and user experience outweigh this tradeoff.
- Workload of implementation: The implementation of this feature requires updates to the admin UI and possibly some backend changes. This might increase the workload for the Strapi team.
- Integration with current features: This proposal should not conflict with existing features and can be seamlessly integrated into the current admin UI.

The proposal does not require significant migration efforts, nor does it introduce breaking changes to existing Strapi applications. However, it might require updates to existing teaching resources, including videos, tutorials, and documentation to reflect the new category feature.

# Alternatives
An alternative solution could be to allow users to create custom views and filters in the admin UI, enabling them to organize their content types in a more personalized way. However, this solution might add more complexity than a simple category dropdown and require more significant changes to the admin UI.

# Unresolved questions
- How will the default category be handled in different languages? Will there be a translation mechanism for the default category name?
- What will be the maximum number of categories allowed? Is there a need for a limit?
- How will category sorting be managed in the dropdown menu? Will users be able to sort categories manually or will they be sorted alphabetically by default?
