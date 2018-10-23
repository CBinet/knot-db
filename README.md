# KnotDatabase

- [Installation](#installation)
- [Insert query](#insert-query)
- [Match query](#match-query)
- [Update query](#update-query)

***

## Installation

### npm

```dos
npm install knot-db --save
```

### Get started

```js
// Import the module
import { KnotdDB } from "node_modules/knot-db";

// Prepare the query
const query = "MATCH (person:Person {origin: 'Canada'}) RETURN person;";

// Execute the query
const results = knotDB.execute(query);

// Extract results
const canadians = results.get("person");
```

***

## Insert query

```js
// Adds a node 'P001' of type 'Person' with name 'John Doe' who owns node 'B001'.
INSERT (P001:Person {name: 'John Doe'} {owns: ['B001']});
```

**Valid Formats**:

- INSERT ( **Id** : **Type** ) ;
- INSERT ( **Id** : **Type** { **Attributes** } ) ;
- INSERT ( **Id** : **Type** { **Attributes** } { **Relations** } ) ;

**Id**: Id of the node. Must be unique.  
*Upper or lower camelcase format required*.

**Type**: Type of the node.  
*Upper or lower camelcase format required*.

**Attributes**: Attributes of the node.  
*JSON format required*.

**Relations**: Relations of the node.  
*JSON format required*.

**Examples**:

```js
// Inserts a node with id '10001' of type 'Person'.
INSERT (10001:Person);

// Inserts a node with id '10001' of type 'Person' with name 'Paul Jones'.
INSERT (10001:Person {name: 'Paul Jones'});

// Inserts a node with id '10001' of type 'Person' with name 'Paul Jones' who likes another node with id '10003'.
INSERT (10001:Person {name: 'Paul Jones'} {likes: ['10003']});
```

### Insert Example

```js
INSERT (P002:Person {name: 'Paul Jones', alive: true, geolocation: [12;23;true;'Québec']} {likes: ['P001'], owns: ['B001';'B002']});
```

```json
{
    "P002": {
        "id": "P002",
        "type": "Person",
        "attributes": {
            "name": "Paul Jones",
            "alive": true,
            "geolocation": [
                12,
                23,
                true,
                "Québec"
            ]
        },
        "relationships": {
            "likes": ["P001"],
            "owns": ["B001","B002"]
        }
}
```

***

## Match query

```js
// Matches all persons from 'Canada' who owns a library.
MATCH (a:Person {origin: 'Canada'})=[owns]>(b:Building {type: 'library'}) RETURN a;
```

### Node Selectors

**Valid Formats**:

- MATCH ( **Name** ) ...
- MATCH ( **Name** : **Type** ) ...
- MATCH ( **Name** : **Type** { **Criterias** } ) ...

**Name**: Name of the selector. This value is used to identify the selector.  
*Upper or lower camelcase format required*.

**Type**: Type of the selector. This value is used to filter using the node type.  
*Upper or lower camelcase format required*.

**Criterias**: Criterias for selection. This value will be used to filter using the node attributes.  
*JSON format required*.

**Examples**:

```js
// Match all of any type
MATCH (a) RETURN a;

// Match all of type 'Person'
MATCH (a:Person) RETURN a;

// Match all of type 'Person' with name 'John Doe'
MATCH (a:Person {name: 'John Doe'}) RETURN a;
```

### Node Relations

**Valid Formats**:

- MATCH **SourceSelector** =[ **Name** ]> **TargetSelector** ...
- MATCH **TargetSelector** <[ **Name** ]= **SourceSelector** ...

**SourceSelector**: Source node selector.  
*See [NodeSelectors](#node-selectors) section above for more details*.

**Name**: Name of the relation. This will be used to filter with the node relationships.  
*Upper or lower camelcase format required*.

**TargetSelector**: Target node selector.  
*See [NodeSelectors](#node-selectors) section above for more details*.

**Examples**:

```js
// Match all of type 'Person' type who 'likes' another 'Person'.
MATCH (a:Person)=[likes]>(b:Person) RETURN a;

// Match all of type 'Building' type who are owned by a 'Person'.
MATCH (a:Building)<[owns]=(b:Person) RETURN a;

// Match all of type 'Person' type who 'likes' another 'Person' who owns a 'Building'.
MATCH (a:Person)=[likes]>(b:Person)=[owns]>(c:Building) RETURN a;
```

### Returns Statements

**Valid Formats**:

- MATCH ... RETURN **Values** ;

**Values**: Return values. If there is more than one return value, values are separated by a comma.  
*Values must be equal to one of the [NodeSelectors](#node-selectors) **Name** to be valid*.

**Examples**:

```js
// Returns the dataset with the selector name 'a'.
MATCH (a:Person) RETURN a;

// Returns the datasets with the selector names 'a' and 'b'.
MATCH (a:Person)=[owns]>(b:Building) RETURN a,b;
```

***

## Update query

```js
// Update name of node 'P001' to value 'John Doe'
UPDATE (P001:Person => {name: 'John Doe'});
```

**Valid Formats**:

- UPDATE ( **Id** : **Type** => { **UpdateValue** } ) ;
- UPDATE ( ? : **Type** => { **UpdateValue** } ) ;
- UPDATE ( ? : **Type** { **Criterias** } => { **UpdateValue** } ) ;

**Id**: Id of the selector. If the value is '?', the query will update all nodes of fitting the **Type** and **Criterias**.  
*Upper or lower camelcase format required*.

**Type**: Type of the selector. This value is used to filter with the node type.  
*Upper or lower camelcase format required*.

**Criterias**: Criterias for selection. This value will be used to filter with the node attributes.  
*JSON format required*.

**UpdateValue**: Properties of the node to update.  
*JSON format required*.

**Examples**:

```js
// Updates name of node 'P001' to value 'Paul Jones'.
UPDATE (P001:Person => {name: 'Paul Jones'});

// Updates name of all nodes of type 'Person' to value 'Paul Jones'.
UPDATE (?:Person => {name: 'Paul Jones'});

// Updates name of all nodes of type 'Person' with name 'Marie Jones' to value 'Paul Jones'.
UPDATE (?:Person {name: 'Marie Jones'} => {name: 'Paul Jones'});
```
