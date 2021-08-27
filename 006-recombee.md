- Start Date: 2021-08-27
- RFC PR: (leave this empty)

# Summary

Any kanvas app that uses de social component (https://github.com/bakaphp/kanvas-packages/tree/0.3/src/Social) ,in most cases need to use recommendation to provide user with relevant entity to consume or follow . In order to simplify the process we will provide a layer of recommendation using recombee service (recombee.com).

We will provide a layer allowing you to 
- Index data to recombee
- Flush database
- Create DB
- Generate Recommendations Items for Users
- Generate Recommendations Items for Items
- Generate Recommendations Users for Users


Cuando se inicializa recommendbe se debe poder enviar la db que se va a utilizar
  - las base de dato de recommendb nos alamacena el tipo de entidad k keremos manejar, memo, books, etc

- poder pedir recomendaciones de items for user
- poder pedir recomendaciones de items for items
- poder pedir recomendaciones de users for users

debe ser nuestro punto inicial

Que sea facil de ingestar cualquier entidad kanvas model, lo suba a recommedb , puedas llamar esos puntos de recomendaciones para usuario y de entidad, de usuario a usaurio debe ser especificio para un modelo de usuario

# Example

If the proposal requires changes to the current API or the creation of new ones, add a basic code example.

```php

namespace Kanvas\Package\Recommendations

class RecommendationIndex
{

}

class UsersIndex extends RecommendationIndex
{

}
```

```php
namespace Gewaer\Recommendations

class Memos extends RecommendationIndex
{
    /**
     * Specify the reference model to this index
     **/
    public function __construct()
    {
        $this->reference = Memo::class;
    }
}
```

```php
$recommendationIndex = new Gewaer\Recommendations\Memos();
$recommendationIndex->createDatabases();
$recommendationIndex->deleteDatabase();

$memo = Memos::findFirst();
$recommendationIndex->index($memo);
$recommendationIndex->delete($memo);

$memos = Memos::find();
$recommendationIndex->indexBatch($memos);

$user = Users::findFirst();

$recommendationIndex->getRecommendationForEntity($memo, float $rotation); //list of memos
$recommendationIndex->getRecommendationForUser($user, float $rotation); //list of memos

$recommendationIndex->refresh($user) //refresh the user recommendation

$recommendationInteraction = new Recommendation\Interaction($recommendationIndex, $this->userData);
$recommendationInteraction->like($memo);
$recommendationInteraction->view($memo);
$recommendationInteraction->viewPortion($memo, float $portion);
$recommendationInteraction->purchase($memo);
$recommendationInteraction->addToCart($memo);
$recommendationInteraction->addToList($memo); //same as bookmark
$recommendationInteraction->bookmark($memo);
```

# Motivation

Simplify the management of recommendation with recombee with kanvas

# Detailed design

Describe the proposal in details:

- Explaining the design so that someone who knows Kanvas can understand and someone who works on it can implement the proposal. 
- Think about edge-cases and include examples.

# Tradeoffs

What potential tradeoffs are involved with this proposal.

- Complexity
- Work load of implementation
- Can this be implemented outside of Kanvas's core packages
- How does this proposal integrate with the current features being implemented
- Cost of migrating existing Kanvas applications (is it a breaking change?)
- Does implementing this proposal mean reworking teaching resources (videos, tutorials, documentations)?

# Alternatives

What are the alternatives?

# Unresolved questions

Optional, but suggested for first draft proposals. What parts of the design are still TBD(To be defined)?
