- Start Date: 2021-07-23
- RFC PR: (leave this empty)

# Summary

The ADF have a huge growing, the ADF is a communication way between Dealer CRM, the ADF can has any information, using xml tags. Once Company can has a template for share the lead information, for example Demo Dealer 1 can share information of the lead, but Dealer Demo 2 can share information about lead, vehicle and vendor.

Both company must has a template for share the information. This template can be storage in the json params of ADF rules. We can set a default Rule with the default template , but if the company need a custom template must be duplicate the default rule and modify the new rule and set the custom template in the params
# Example

```


$documentAsArray = json_decode(json_encode($template), true);

// Recursive function for parse arra xml to array with xml key and database value

function changeValueXMl(array $element, ?array $values = null)
{
    $arrayOfDatabase = $values ?? [
        'example' => 'Valor del Example',
        'examplePath' => [
            'name' => 'Valor del examplePath > name',
            'lastname' => 'Valor del examplePath > lastname',
        ]
    ];
    foreach ($element as $key => $value) {
        //check if is array, in this case search into to array
        if (is_array($value)) {
            if (key_exists($key, $arrayOfDatabase)) {
                $element[$key] = changeValueXMl($value, $arrayOfDatabase[$key]);
                continue;
            }
        }
        echo "Key {$key} <br>";
        $newValue = $arrayOfDatabase[$key];

        $element[$key] = "Cambio de {$value} to {$newValue}";
    }
    return $element;
}

$document = changeValueXMl($documentAsArray);
$document = [
    'adf' => $document
];

$result = ArrayToXml::convert($document, [
    'rootElementName' => 'adf',
    '_attributes' => [
        'version' => '1.0',
    ],
], true, 'UTF-8');

// Get the string xmml and save on file
$root = simplexml_load_string($result);
$root->save('file.xml');

```

# Motivation

Please make sure to explain the motivation for this proposal. 
It means explaining the use case(s) and the functional feature(s) this proposal is trying to solve. 

Try to only talk about the intent not the proposed solution here.

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
