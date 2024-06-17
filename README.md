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
  <li><a href="#dealing-with-collections">Dealing with Collections</a>
    <ul>
      <li><a href="#collections-have-operations">Collections have operations</a></li>
      <li><a href="#pagination">Pagination</a></li>
      <li><a href="#sorting">Sorting</a></li>
      <li><a href="#filtering">Filtering</a></li>
      <li><a href="#collections-represents-mn-sided-relations">Collections represents M/N-sided relations</a></li>
    </ul>
  </li>
  <li><a href="#request-and-response-cycle">Request and Response Cycle</a>
    <ul>
      <li><a href="#request-http-methods">Request HTTP methods</a></li>
      <li><a href="#response-http-status-codes">Response HTTP status codes</a></li>
      <li><a href="#json-response-document">JSON Response Document</a></li>
    </ul>
  </li>
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

`POST /cars/1/upload_photo` (using `multipart/form-data`)

This way, the logic of uploading the car image is decoupled for its creation, making the API more composable and making server-side logic easier to maintain, debug and reason about.

> [!NOTE]
> If the flow above requires a single call for a specific client (making car metadata and photo upload atomic), a wrapper endpoint can be exposed through a gateway (reverse proxy). This pattern is know as [BFF (Backend for Frontend)](https://learn.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends).


## Dealing with Collections

A **collection** in simply an array of entity objects.


### Collections have **operations**

Collections support the following operations:
- Pagination
- Sorting
- Filtering


### Pagination

Clients can paginate through a collection using the `page_number` and `page_size` query params (if those values are not sent by the client, server defaults will be used).

`GET /nations?page_number=1&page_size=2`: first page containing two `nations` objects.

```json
[
  {
    "entity": "nations",
    "id": 1,
    "name": "Brazil",
    "continent": "America"
  },
  {
    "entity": "nations",
    "id": 2,
    "name": "Uruguay",
    "continent": "America"
  }
]
```

`GET /nations?page_number=3&page_size=3`: third page containing three `nations` objects.

```json
[
  {
    "entity": "nations",
    "id": 7,
    "name": "Japan",
    "continent": "Asia"
  },
  {
    "entity": "nations",
    "id": 8,
    "name": "China",
    "continent": "Asia"
  },
  {
    "entity": "nations",
    "id": 9,
    "name": "South Korea",
    "continent": "Asia"
  }
]
```


### Sorting

Sorting order can be either `ASCENDING` or `DESCENDING`.

Sort nations by `name` in `ASCENDING` order: `GET /nations?sort=name`

```json
[
  {
    "entity": "nations",
    "id": 122,
    "name": "Afghanistan",
    "continent": "Asia"
  },
  {
    "entity": "nations",
    "id": 83,
    "name": "Albania",
    "continent": "Europe"
  },
  {
    "entity": "nations",
    "id": 57,
    "name": "Algeria",
    "continent": "Africa"
  }
]
```

Sort nations by `name` in `DESCENDING` order: `GET /nations?sort=-name` (hyphen/minus signal in front of the field name)

```json
[
  {
    "entity": "nations",
    "id": 160,
    "name": "Zimbabwe",
    "continent": "Africa"
  },
  {
    "entity": "nations",
    "id": 45,
    "name": "Zambia",
    "continent": "Africa"
  },
  {
    "entity": "nations",
    "id": 37,
    "name": "Yugoslavia",
    "continent": "Europe"
  }
]
```


### Filtering

Clients can filter the collection by entity properties using the `filter` query param. The following table shows available filter operators.

Filter operator          | Description           | Expression example
--------------------     | --------------------- | -----------------------------------------------------
**Comparison Operators** |                       |
eq                       | Equal                 | city eq "Redmond"
ne                       | Not equal             | city ne "London"
gt                       | Greater than          | price gt 20
ge                       | Greater than or equal | price ge 10
lt                       | Less than             | price lt 20
le                       | Less than or equal    | price le 100
**Logical Operators**    |                       |
and                      | Logical and           | price le 200 and price gt 3.5
or                       | Logical or            | price le 3.5 or price gt 200
not                      | Logical negation      | not price le 3.5
**Grouping Operators**   |                       |
( )                      | Precedence grouping   | (priority eq 1 or city eq "Redmond") and price gt 100

Just insert the expression as a value for the `filter` query param:

`GET /nations?filter=continent eq "Europe"`


### Collections represents M/N-sided relations

Recalling, there are 3 types of relationship between entities:
- One-to-One (`1-1`)
- One-to-Many (`1-N`)
- Many-to-Many (`M-N`)

As we've seen, [expandable nested objects](#nested-relationship-objects-are-expandable) represents **1-sided** relations. For **M/N relations** (one-to-many and many-to-many), we use collections.

**One-to-Many (`1-N`)**: an **owner** has a collection of **dogs**.

`GET /users/1/dogs` redirects to `/dogs?filter=owner.id eq 1` (fetch `dogs` collection filtering by the owner)

```json
[
  {
    "entity": "dogs",
    "id": 1,
    "name": "Max",
    "owner": {
      "entity": "users"
      "id": 1
    }
  },
  {
    "entity": "dogs",
    "id": 2,
    "name": "Scott",
    "owner": {
      "entity": "users",
      "id": 1
    }
  }
]
```


**Many-to-Many (`M-N`)**: relationship between `orders` and `products`. An order has a collection of products, and a product has a collection of orders.

`GET /orders/{id}/products`: list products in that order.

`GET /products/{id}/orders`: list the orders that a given product is present.


## Request and Response Cycle

### Request HTTP methods

HTTP methods follows the specifications of [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html#name-methods). A summary is presented below.

HTTP method              | Common usage
--------------------     | ---------------------
POST                     | Create a new resource by transferring a representation; perform an unsafe action
GET                      | Fetch an object or a collection of objects; perform a safe (read-only) action
PATCH                    | Update an existing object by transferring a partial representation of entity data
PUT                      | Used for upserts
DELETE                   | Client requests the deletion of an object


### Response HTTP status codes

Status codes are categorized in four classes:

Status code range        | Result
--------------------     | ---------------------
2xx                      | Success
3xx                      | Redirection
4xx                      | Client-side errors
5xx                      | Server-side exceptions

Most commonly used response status codes are:

Status code                               | Result
---------------------------               | ---------------------
`200 OK`                                  | Fetched an object/collection; updated (PATCH) an object; performed a safe (read-only) action
`201 Created`                             | Request processing resulted in the creation of an object; performed an unsafe action
`202 Accepted`                            | Used for asynchronous requests
`204 No Content`                          | Request succeeded but returned no data
`400 Bad Request`                         | Malformed request syntax or invalid request semantics
`401 Unauthorized`                        | Lacking or invalid authentication credentials
`403 Forbidden`                           | Server understood the request but refuses to fulfill it. Given credentials does not have enough access level to the specified resource
`404 Not Found`                           | Server did not find a current representation for the target resource
`500 Internal Server Error`               | Server encountered an unexpected condition that prevented it from fulfilling the request


### JSON Response Document

Unless returning a `204 No Content`, requests will return a JSON document. This document have defined fields, making the results predictable and stardandized.

The document have possibly 4 root-level fields:
- `meta`: optional object
- `data`: only present if `errors` is absent. Either an object or an array of objects
- `pagination`: only present if `data` is an array
- `errors`: only present if `data` is absent

If the four root-level fields above are absent (meaning a successful request returning no data), it is a `204 No Content` status code.


#### Meta Object

This object represents request metadata. Contains two defaults fields (`status` and `message`) and any other API-specific fields.

Example (default fields; mostly for debugging)

```json
{
  "meta": {
    "status": 404,
    "message": "Could not find any user with provided ID."
  }
}
```

More concrete examples

```json
{
  "meta": {
    "status": 201,
    "message": "Created",
    "request_token": "32606149-b8a2-42b0-b507-92d7f7c22465",
    "remaining_quota": "Your API request limit is current at 69% of your daily usage quota."
  }
}
```


#### Data Object/Array

Primary result of your request. The resulting data after requested was processed.

`GET /nations/8`

```json
{
  "data": {
    "entity": "nations",
    "id": 8,
    "name": "China",
    "continent": "Asia"
  }
}
```


#### Pagination Object

If `data` is an object, `pagination` is present. This object consists of four fields:
- `total_pages`: Total number of pages to represent the collection of objects using the current page size
- `current_page`: Current page number
- `page_size`: Page size controls how many objects each page will return
- `objects_count`: Count of the number of objects existing in this collection

`GET /nations?page_number=3&page_size=3`

```json
{
  "data": [
    {
      "entity": "nations",
      "id": 7,
      "name": "Japan",
      "continent": "Asia"
    },
    {
      "entity": "nations",
      "id": 8,
      "name": "China",
      "continent": "Asia"
    },
    {
      "entity": "nations",
      "id": 9,
      "name": "South Korea",
      "continent": "Asia"
    }
  ],
  "pagination": {
    "total_pages": 4,
    "current_page": 3,
    "page_size": 3,
    "objects_count": 12
  }
}
```


#### Errors array

Array of error objects detailing **client-side** errors. An **error object** consists of three fields:
- `code`: application-specific error code
- `title`: a short, human-readable title of the error caused by the client
- `detail` (optional): error message providing details about the error in the current request context



