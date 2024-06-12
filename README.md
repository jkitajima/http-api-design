# HTTP API Design Guidelines

The purpose of this document is to establish a shared vocabulary and a pragmatic (yet robust) approach to design an HTTP API using [JSON](https://www.json.org) (JavaScript Object Notation) as a data-interchange format.


## Table of Contents

<ul>
  <li><a href="#core-design-principles">Core Design Principles</a>
    <ul>
      <li><a href="#an-api-is-a-relationship-of-entities">An API is a Relationship of Entities</a></li>
      <li><a href="#entities-have-shapes">Entities have shapes</a></li>
      <li><a href="#id-as-a-pair">ID as a Pair</a></li>
      <li><a href="#entities-relationships-are-nested-1-sided-relation">Entities relationships are nested</a></li>
      <li><a href="#the-id-pair-is-a-location">The ID Pair is a Location</a></li>
      <li><a href="#nested-relationship-objects-are-expandable">Nested relationship objects are Expandable</a></li>
      <li><a href="#entities-have-grammar">Entities have grammar</a></li>
      <li><a href="#single-responsibility-and-concern-separation">Single Responsibility and Concern Separation</a></li>
    </ul>
  </li>
  <li>TODO!</li>
</ul>


## Core Design Principles


### An API is a Relationship of Entities

An API (Application Programming Interface) is represented as a **relationship** of **entities**.

These entities cooperate together in order to perform actions, manage state and transfer representations of entity data.


### Entities have shapes

Clients can interact with entities in two shapes:
1. A single **object**: `/objects/{id}`
2. A **collection** of objects: `/objects`


### ID as a Pair

An object **uniqueness** is defined as a **pair** of values (entity + id) and not a single value (id).

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



### Relationships

There are 3 types of relationship between entities:
- One-to-One (`1-1`)
- One-to-Many (`1-N`)
- Many-to-Many (`M-N`)

This means that, in every type of relationship, an object has possibly:
- A nested object representing a foreign key relationship ("1-sided" relation)
- A location for representing the collection of objects ("M/N-sided" relation)


### Entities relationships are nested (1-sided relation)

Since the single value of an id does not define an object, entities relationships are nested.

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

Now that the uniqueness of object is a pair consisting of both entity and id values, these two values together defines a location inside the API.

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

`GET /dogs/1/owner` redirects to `GET /users/1`
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


### Nested relationship objects are Expandable

Since relationships are represented as nested objects, they are **expandable**.

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
> Relationship objects must either return the identification pair (entity + id) **or** the full object data.
> 
> **Do not** return partial representation of data.


### Entities have grammar

Entity endpoints are separated by grammar semantics:
- A `noun` endpoint is used for **transferring entity state representations** (`/objects/{id}`)
- A `verb` endpoint is used for performing **actions** (`/objects/{id}/do`)

An example for a `user_files` entity with the following schema:


```yaml
id: uuid
name: string
extension: string
url: string

```

`POST /user_files` will create a new object by transfering a representation:


```json
{
  "name": "gopher"
  "extension": ".png"
  "url": "https://go.dev/blog/gopher"
}

```

While, for a file upload using `multipart/form-data`, an action should be used: `POST /user_files/upload`. Then, after server-side logic was processed, a response object would be:


```json
{
  "entity": "user_files",
  "id": "13728a84-e1dc-4de6-8f88-f0ba574907ad",
  "name": "gopher"
  "extension": ".png"
  "url": "https://storage.com/blobs/gopher.png"
}

```


### Single Responsibility and Concern Separation

Ideally, requests have a single responsibility. This principle promotes the modularization of the API, making it composable.

For instance, imagine that the client-side wants to create a `cars` object with a photo. Instead of uploading the photo image and submitting other data in a single call, the flow would be:


1. Create a `cars` object

`POST /cars`
```json
{
  "model": "Mazda RX-7",
  "year": 1978
}

```

Possible response:

```json
{
  "entity": "cars",
  "id": 1,
  "model": "Mazda RX-7",
  "year": 1978,
  "photo": null
}

```

2. Upload car image by performing an **action**:

`POST /cars/{id}/upload_photo` (using `multipart/form-data`)

This way, the logic of uploading the car image is decoupled for its creation, making the API more composable and making server-side logic easier to maintain, debug and reason about.

> [!NOTE]
> If the flow above requires a single call (making car metadata and photo upload atomic), a wrapper endpoint can be exposed through a gateway (reverse proxy).
