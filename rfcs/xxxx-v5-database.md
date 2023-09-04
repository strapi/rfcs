- Start Date: (fill in today's date, 2023-09-04)
- RFC PR: (leave this empty)

# Summary

We need to support development of new features. Most of them are going towards a more CMS like model. to support this approach we will introduce the notion of `documents` & `links`.

# Motivation

Bringing new features that were not possible in v4. The most important ones:

## Draft and Publish

Introducing a new version of draft & publish. One which allows having one draft and one published entry at the same time.

This version should support having different relations between a draft and a published version.

Consider a scenario where a writer wants to work on an article (draft) while the previously published version remains unaffected. This requires a system that supports simultaneous draft and published entries, leading to the idea of `Documents`.

## Synchronized relations across locales

In Strapi V4, users requested the ability to share the same relation across multiple locales. This led to the concept of `Links`, allowing relations to be shared across different document locales. And also unblocks other use cases mentioned in this RFC.

Instead of breaking v4 relations `Links` will be a different type, and v4 relations will be kept as legacy relations.

# Detailed design

Documents are introduced in v5 as a way to group multiple entries. That is implemented by adding a new `document_id` column in entries:

<p align="center">
	<img src="https://github.com/strapi/rfcs/assets/20578351/4a02817e-4004-45ee-9e4a-a755e66c2064" width="300px" />
</p>

Also, all the variations of that document will have its own column in every entry:

- `locale` for i18n
- `publicationState` for draft and publish

## Document id and entry id

Additionally, in V5, all identifiers (entry ids and document ids) will now be universally unique identifiers (UUIDs). This transition offers significant benefits:

- `Data Transfer`: Previously, transferring data between databases while retaining the same entry IDs was not possible, due to the use of incremental ids.

The switch to UUIDs, which will keep entry and document ids unique, makes this feasible.

- `More secure`: The use of UUIDs also enhances security. With incremental IDs, one could predict the existence of other IDs based on a known ID. For instance, if ID `10` is known, it is reasonable to infer that IDs `9`, `8`, etc., exist in the database. UUIDs, being non-sequential and randomly generated, prevent such easy discovery.

While the probability of two UUIDs colliding is not zero, it is practically negligible, making them an excellent choice for unique identifiers.

## Links

`Links` introduce the following key functionalities:

### Relate entry to document

In V4, you might have encountered this situation:

<p align="center">
	<img src="https://github.com/strapi/rfcs/assets/20578351/01061116-478b-465f-9b3b-a88ea02598c6" width="600px" />
</p>


When relating an article to a specific author in V4, only articles and authors of the same locale could be linked, because localizations were not really grouped in a single document.

Thus, if an author did not have a locale that matched the article's, creating a relationship was impossible.

`Links` will solve this by referencing a document as a whole, not a specific locale:

<p align="center">
	<img src="https://github.com/strapi/rfcs/assets/20578351/da52c5de-57c2-4ced-8986-e0f6787d8892" width="600px" />
</p>


### Synchronize links across locales

In V4, if you want to keep the same relation across the same locales, one would have to manually create them, with the risk of missing one.

<p align="center">
	<img src="https://github.com/strapi/rfcs/assets/20578351/0e2729fe-7673-490b-9493-9ea3a8fb4470" width="600px" />
</p>

In V5, `links` will offer the possibility to be synchronized across locales.

<p align="center">
	<img src="https://github.com/strapi/rfcs/assets/20578351/0d5841dd-89b1-42f6-bed5-b463f40f515b" width="600px" />
</p>

As an additional comment, the synched relation will only affect the locales inside the same publication state. Meaning, relations will not be shared between the `draft` and `published` versions of the document.

<p align="center">
	<img src="https://github.com/strapi/rfcs/assets/20578351/4a69c2e3-628e-45b1-8c77-338272a61c32" width="600px" />
</p>


### Polymorphic links

V5 will eventually support polymorphic links.

Those will allow use cases like Website Menus , linked to many other content types

<p align="center">
	<img src="https://github.com/strapi/rfcs/assets/20578351/29628824-c735-4776-b651-4c99ad11db8f" width="500px" />
</p>

This was just not possible in V4.

## Link content schema

Links will be represented as so in the content type schema:

```jsx
{
  attributes: {
    “myLink”: {
       “type”: "link",
			 // Target one (or multiple) content types.
			 // Target all by not defining target
       “target”: "uid-a" | ["uid-a", "uid-b"] | undefined,
       “multiple”: true|false
		}
	}
}
```

## Links Database Schema

The database schema of `links` join tables :

<p align="center">
	<img src="https://github.com/strapi/rfcs/assets/20578351/f6ebb23e-c6e7-47b6-bdae-8030c80786a6" width="700px" />
</p>

The join table will now contain:

- `Owner side doc id`: Reference to do the document id of the owner side (Article in the example)
- `Inverse side doc id`: Reference ot the document id of the inverse side (Author in the example)
- `pivot columns`: Reference the owner side variations (locale & publication state).
- `order`: Only in `manyWays` and `morphMany` relations.

## Query engine

The query engine will expose the same methods as before.

This layer will be used to interact with the database rows directly, and as so, methods such as `findOne` and `update` will reference entry ids and not document ids.

Interactions with documents will be done in the application layer `DocumentService` , which will be documented in another RFC.

# Tradeoffs

Benefits of this approach:

- Links schema reduces the number of queries needed to read and create relations.
- It is conceptually easy, and similar to our approach.
- It's easy to share the same relation between different document locales.

Downsides:

- Links will not allow for bidirectionality. A relation that references an entry to a document introduces many complexities that prevents bidirectionality, or makes it rather difficult. The first versions of links will not have bidirectionality in mind.

# Unresolved questions

- How do we manage life cycles?
  - A database lifecycle would trigger at an entry level, not a document level. As everything will be document oriented, database life cycles might have less sense from a user perspective in V5.
  - Document life cycles could be managed in a upper layer than the database.
- How are we going to support v4 relation types with the new document ids.
  - Could we connect a v4 relation with a document Id, or do we only support connecting with entry ids?
- Do we prefix internal strapi columns in db?
  - We need to avoid collisions
- Which universal ID to be used. UUID, CUID, NANOID, ULID.
