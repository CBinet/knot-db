# KnotDatabase

- [Get started](#get-started)
- [Insert query](#insert-query)
- [Match query](#match-query)
- [Update query](#update-query)

***

## Get started

### Installation

#### Install globally

```dos
npm install -g knot-db
```

Then you can simply run :

```dos
knot-db
```

#### Install locally

```dos
npm install knot-db --save
```

Navigate to the install location and run the following command :

```dos
node knot-db.js
```

The database should now be running on the port specified in the *config.json* file. (*9600* by default)

<img src="https://imgur.com/download/JKL9HRC" style="width: 50%; height: 50%; margin-left: 25%"/>

### Sending queries

To query the database, simply send an HTTP **post** request at ```localhost:<PORT>/query``` with a body like the following :

```json
POST/query
{
    "table" : "employees",
    "query" : "MATCH (a:Person {name: 'Marc'}) RETURN a;"
}
```

where **table** is the name of the table to apply the query and **query** is the actual query to execute.

A typical response would like this :

```json
{
    "a": {
        "P256": {
            "id": "P256",
            "type": "Person",
            "attributes": {
                "name": "Marc",
                "surname": "Davis",
                "sector": "Marketing"
            },
            "relationships": {
                "likes": ["P965"]
            }
        },
        "P443": {
            "id": "P443",
            "type": "Person",
            "attributes": {
                "name": "Marc",
                "surname": "Jones",
                "sector": "Sales",
                "intern": true
            },
            "relationships": {}
        }
    }
}
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
You can also use the wildcard operator (?) to match any type of node.  
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

// Match all of any type with location 'Québec, Canada'
MATCH (a:? {location: 'Québec, Canada'}) RETURN a;
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
