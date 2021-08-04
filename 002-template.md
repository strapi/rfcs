- Start Date: 2021-07-23
- RFC PR: (leave this empty)

# Summary

Auto-lead Data Format is an industry standard for sharing lead information between tools that help manufacturers and dealers sell more cars (http://adfxml.info/). Its provides a XML format to communicate between any CRM by sending the lead information in a standard XML format.

```xml
<?ADF VERSION "1.0"?>
<?XML VERSION “1.0”?>
<adf>
    <prospect>
        <requestdate>2000-03-30T15:30:20-08:00</requestdate>
        <vehicle>
            <year>1999</year>
            <make>Chevrolet</make>
            <model>Blazer</model>
        </vehicle>
        <customer>
            <contact>
                <name part="full">John Doe</name>
                <phone>393-999-3922</phone>
            </contact>
        </customer>
        <vendor>
            <contact>
                <name part="full">Acura of Bellevue</name>
            </contact>
        </vendor>
    </prospect>
</adf>
``` 

Key Categories:
- Lead (prospect) information
- Vehicle information
- Customer (buyer) information
- Vendor (dealer) information
- Service provider information

We are proposing is proposing a way for our leads to interact with third party vendors, starting with ADF.

If you don't know what a lead is by this point, please check what project you are working on :P

Leads have a standard format of 
- Id
- Uuid
- Name
- Lastname
- Email
- Phone
- Description
- Status
- Pipeline

Each company can add any additional field to our lead structure via custom fields. Also we provide a table called leads_linked_sources focus on the relationship between our lead and any third party vendors.

Having those two basics we can start with our proposal.

For use ADF must use [Workflows](https://github.com/bakaphp/kanvas-packages/tree/0.3/src/WorkflowsRules)

# Implementation 

Understanding that ADF are specificity focus on the Automobile industry this implementation would only work on leads using the Action with vehicle products.

## Transformer Service

Understanding that our lead system will need to connect to a bunch of third party service, where ADF will just be a additional one, adding a new Trait to our leads class for each one would not be ideal so I'm proposing , is creating a Service where we can pass the lead and to what format we want it to be converted.

```php

namespace Gewaer\Services\Leads\Transformers;

declare(strict_types = 1);

Class Transformers
{
    public static function factory(c $type, \Baka\Database\Model ...$args)
    {
        /**
         * doesn't necessary has to be a switch , we can get it from the DB better so we don't have to hard code
         * each implementation
         **/
        switch ($type) {
            case 'ADF':
                    return new ADF($args);
                break;
        }

    }
}

``` 

Since each transformer will have its own logic we can on the transformer package add ADF logic

```php

namespace Gewaer\Services\Leads\Transformers;

declare(strict_types = 1);

interface TransformerEngine
{
    /**
     * Function that will return the ADF format for specific lead
     */
    public function toFormat() : string;

    /**
     * Function that will return the array data attribute
     */
    public function getData(): array;

}
``` 

```php

namespace Gewaer\Services\Leads\Transformers\ADF;

declare(strict_types = 1);

use Canvas\Models\SystemModules;
use Canvas\Template;

Class ADF implements TransformerEngine
{
    protected array $data;

    public function __construct(Baka\Database\Model ...$args)
    {
        foreach($args as $arg) {
            $systemModule = SystemModules::getByModelName(self::class);
            $this->data[$systemModule->slug] = $args->getData();
        }
    }

    /**
     * Function that will return the ADF format for specific lead
     */
    public function toFormat() : string
    {
        return Template::generate('ADF', $this->getData());
    }
}
```


## Communicators 

Understanding that transformation is just part of the process we need to also add a communication provider to send our leads to it final destination after its formatted correctly

```php

namespace Gewaer\Services\Leads\Communication;

declare(strict_types = 1);

interface CommunicationEngine
{
    /**
     * Function that will send a transformed entity to the third party integration
     */
    public function send() : bool;
}
```

```php

namespace Gewaer\Services\Leads\ADF;

Class ADF implements CommunicationEngine
{
    protected TransformerEngine $transformedLead;
    protected Company $company;

    public function __construct(TransformerEngine $transformedLead, Company $company)
    {
        $this->transformedLead = $transformedLead;
        $this->company = $company;
    }

    /**
     * Function that will return the ADF format for specific lead
     */
    public function send() : bool
    {
        $this->mail->to($company->getEmail())
            ->content($this->transformedLead->toFormat(), 'text/plain')
            ->send();

        //save reference to leads_lead_source

        return true;
    }
}

```
```php

namespace Gewaer\Services\Leads\ADF;

Class Zoho implements CommunicationEngine
{
    protected TransformerEngine $transformedLead;
    protected Company $company;

    public function __construct(TransformerEngine $transformedLead, Company $company)
    {
        $this->transformedLead = $transformedLead;
        $this->company = $company;
    }

    /**
     * Function that will return the ADF format for specific lead
     */
    public function send() : bool
    {
        $this->zoho->insert('lead', $this->transformedLead->toFormat());

        //save reference to leads_lead_source

        return true;
    }
}

```

# Motivation
The main motive of this proposal , its to allow our leads to communicate with any third party provider starting with ADF

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