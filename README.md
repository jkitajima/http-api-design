# HTTP API Design Guidelines

The purpose of this document is to establish a shared vocabulary and a pragmatic (yet robust) approach to design an HTTP API using [JSON](https://www.json.org) (JavaScript Object Notation) as a data-interchange format.


## Table of Contents

<ul>
  <li><a href="#core-design-principles">Core Design Principles</a>
    <ul>
      <li><a href="#an-api-is-a-relationship-of-entities">An API is a Relationship of Entities</a></li>
      <li><a href="#entities-have-forms">Entities have forms</a></li>
      <li><a href="#id-as-a-pair">ID as a Pair</a></li>
      <li><a href="#entities-relationships-are-nested">Entities relationships are nested</a></li>
      <li><a href="#the-id-pair-is-a-location">The ID Pair is a Location</a></li>
      <li><a href="#nested-relationship-objects-are-expandable">Nested relationship objects are Expandable</a></li>
    </ul>
  </li>
  <li>TODO!</li>
</ul>


## Core Design Principles


### An API is a Relationship of Entities

An API (Application Programming Interface) is represented as a **relationship** of **entities**.

These entities cooperate together in other to perform actions, manage state and transfer representations of entity data.


### Entities have forms

Clients can interact with entities in two forms:
1. A single **object**: `/objects/{id}`
2. A **collection** of objects: `/objects`


### ID as a Pair

An object uniqueness is defined as a **pair** of values (entity + id) and not a single value (id).

In other words, this does **not** define an object:

```json
{
  "id": 1,
  "name": "Max"
}

```

Valid forms of defining an object:

```json
{
  "entity": "users",
  "id": 1,
  "name": "Max"
}

```

```json
{
  "entity": "dogs",
  "id": 1,
  "name": "Max"
}

```


### Entities relationships are nested

Since the single value of a an id does not defined an object, entities relationships are nested.

In other words, this does not define a relationship:
```json
{
  "id": 1,
  "name": "Max"
  "owner_id": 1
}

```

Valid form of defining a relationship:
```json
{
  "entity": "dogs",
  "id": 1,
  "name": "Max"
  "owner": {
    "entity": "users",
    "id": 1
  }
}

```


### The ID Pair is a Location

Now that uniqueness of object is a pair consisting of both entity and id values, these two values together defines a location inside the API.

`GET /dogs/1`
```json
{
  "entity": "dogs",
  "id": 1,
  "name": "Max"
  "owner": {
    "entity": "users",
    "id": 1
  }
}

```

`GET /users/1`
```json
{
  "entity": "users",
  "id": 1,
  "name": "Max",
  "age": 12,
  "citizen_id": "123.456.789-00"
  "phone_number": "91234-5678"
}

```


#### Nested relationship objects are Expandable

Since relationships are represented as nested objects, they can be expandable.

`GET /dogs/1?expand=owner`
```json
{
  "entity": "dogs",
  "id": 1,
  "name": "Max"
  "owner": {
    "entity": "users",
    "id": 1,
    "name": "Max",
    "age": 12,
    "citizen_id": "123.456.789-00"
    "phone_number": "91234-5678"
  }
}

```

> [!IMPORTANT]
> Relationship objects must either return the identification pair (entity + id) **OR** the full object data.
> 
> **DO NOT** return partial representation of data.



