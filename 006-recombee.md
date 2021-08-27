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
//manage db
$recommendationIndex = new Gewaer\Recommendations\Memos();
$recommendationIndex->create();
$recommendationIndex->delete();

$memo = Memos::findFirst();
$recommendationIndex->index($memo);
$recommendationIndex->delete($memo);

$memos = Memos::find();
$recommendationIndex->indexBatch($memos);

$user = Users::findFirst();

$recommendationIndex->getRecommendationForEntity($memo, float $rotation); //list of memos
$recommendationIndex->getRecommendationForUser($user, float $rotation); //list of memos

$recommendationIndex->refresh($user) //refresh the user recommendation

//manage interaction

$recommendationInteraction = new Recommendation\Interaction($recommendationIndex, $this->userData);
$recommendationInteraction->like($memo);
$recommendationInteraction->view($memo);
$recommendationInteraction->viewPortion($memo, float $portion);
$recommendationInteraction->purchase($memo);
$recommendationInteraction->addToCart($memo);
$recommendationInteraction->addToList($memo); //same as bookmark
$recommendationInteraction->bookmark($memo);

//Or
$recommendationIndex = new Gewaer\Recommendations\Memos();
$recommendationIndex->interactions($user)->like($memo);
$recommendationIndex->interactions($user)->view($memo);
```

# Motivation

Simplify the management of recommendation with recombee with kanvas

# Detailed design

Create a new package called Recommendation 
  - Contracts
    - Database : definitions of how a database recommendation works  (memos)
    - Items : definitions how items works within a database
    - Users : definitions users behavior
    - Interactions : define the method that interact with a database  (like, view, purchase)
    - Engine : define the logic to connect to the recommendation provider
  - Drivers
    - Recombee.php
  - Database.php
  - Items.php
  - Interactions.php

## Contracts
```php

namespace Kanvas\Packages\Recommendation\Contracts;

class Database
{
    public function create();
    public function delete();
    public function getSource();
}
```

```php

namespace Kanvas\Packages\Recommendation\Contracts;

class Items
{
    public function add();
    public function addMultiple(); //or batch index?
    public function delete(); //or batch index?
    public function list(); //or batch index?
}
```

```php

namespace Kanvas\Packages\Recommendation\Contracts;

class Interactions
{
    public function like(ModelInterface $model);
    public function view(ModelInterface $model);
    public function purchase(ModelInterface $model);
    public function bookmark(ModelInterface $model);
}
```

```php

namespace Kanvas\Packages\Recommendation\Contracts;

class Users
{
    public function add(UsersInterface $model);
    public function delete(UsersInterface $model);
}
```

```php

namespace Kanvas\Packages\Recommendation\Contracts;

class Engine
{
    public static function connect(Database $database);
}
```
## Drivers

```php

namespace Kanvas\Packages\Recommendation\Drivers;

use Recombee\RecommApi\Client;

class Recommendation implements Engine
{
    private static array $instances = [];
    
    public static function connect(Database $database)
    {
        $source = $database->getSource();
        if (!isset(self::$instances[$source])) {
            self::$instances[$source] = new Client($source, '--db-private-token--');
        }

        return self::$instances[$source];
    }
}
```

## Source

```php

namespace Kanvas\Packages\Recommendation\Drivers;


class Database implements Database
{
    protected Engine $client;

    public final function __constructs(Di $container)
    {
        $engine = $container->get('recommendation');

        if(!$engine instanceOf Engine) {
            throw new Exception('Not a recommendation driver');
        }

        $this->client = $container->get('recommendation')->connect($this->getSource());    
    }

    public function getClient() : Engine;
    public function create() : self;
    public function delete() : bool;

    public function interactions() : Interactions;
    public function items() : Items;
    public function users() : Items;
    public function getRecommendationForEntity($memo, float $rotation);
    public function getRecommendationForUser($memo, float $rotation);
    public function refresh($memo);

    
}
```

```php

namespace Kanvas\Packages\Recommendation\Drivers;

use Recombee\RecommApi\Client;

class Items implements Items
{
    protected Database $database
    ;
    public function __construct(Database $database)
    {
        $this->database = $database;
    }

    public function add();
    public function addMultiple(); //or batch index?
    public function delete(); //or batch index?
    public function list(); //or batch index?
}
```

```php

namespace Kanvas\Packages\Recommendation\Drivers;

use Recombee\RecommApi\Client;

class Users extends Items
{

}
```

```php

namespace Kanvas\Packages\Recommendation\Drivers;

use Recombee\RecommApi\Client;

class Interactions implements Interactions
{
    protected Database $database
    ;
    public function __construct(Database $database)
    {
        $this->database = $database;
    }

    public function like(ModelInterface $model);
    public function view(ModelInterface $model);
    public function purchase(ModelInterface $model);
    public function bookmark(ModelInterface $model);
}
```

# Tradeoffs

- Limit ourself to only one recommendation provider. In the future we should expand to more services
- Cache managements is critical , since all recommendation service have some limit on # of get calls

# Unresolved questions

Optional, but suggested for first draft proposals. What parts of the design are still TBD(To be defined)?
