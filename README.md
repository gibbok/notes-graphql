# Notes on GraphQL

GraphQL is a query language for your API, and a server-side runtime for executing queries using a type system you define for your data.

GraphQL is created by defined types and fields, then providing functions for each field:

Resolver in GraphQL can be done for fields or endpoints.

> GraphQL query language is basically about selecting fields on objects.

The shape of a GraphQL query closely matches the result, you can predict what the query will return without knowing that much about the server.

```
type Query {
  me: User
}

type User {
  id: ID
  name: String
}

function Query_me(request) {
  return request.auth.user;
}

function User_name(user) {
  return user.getName();
}
```

A GraphQL service is running on a URL it can receive queries to validate and execute by calling the provided functions to produce a result.

At is simplest, GraphQL is about asking for specific fields on objects, the query has the same shape of the result.

> GraphQL queries can traverse related objects and their fields, letting clients fetch lots of related data in one request, instead of making several roundtrips as one would need in a classic REST architecture.

GraphQL queries look the same for both single items or lists of items; however, we know which one to expect based on what is indicated in the schema.

GraphQL is typically served over HTTP via a single endpoint which expresses the full set of capabilities of the service. This is in contrast to REST APIs which expose a suite of URLs each of which expose a single resource.

### Advantage

> Instead of doing one API request to get basic information about an object, and then multiple subsequent API requests to find out more information about that object like in REST, you can get all of that information in one API request. That saves bandwidth, makes your app run faster, and simplifies your client-side logic.


## Arguments

In GraphQL, every field and nested object can get its own set of arguments, making GraphQL a complete replacement for making multiple API fetches.

Arguments can be of many different types.

GraphQL server can also declare its own custom types, as long as they can be serialized into your transport format.

```
{
  human(id: "1000") {
    name
    height
  }
}

{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72
    }
  }
}
```

## Aliases

The result object fields match the name of the field in the query but don't include arguments, you can't directly query for the same field with different arguments. 

Are used to avoid conflicts as we can rename the results.

```
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

Aliases can be used also to rename fields for instance we can use rename to camel case like:

```
query GetEntries {
  entries {
    id
    updatedAt: updated_at
  }
}
```

https://devinschulz.com/rename-fields-by-using-aliases-in-graphql/

## Fragments

Fragment are reusable units. A fragment let me create a set of fields to include theme in queries.

Fragments are frequently used to split complicated application data requirements into smaller chunks, especially when you need to combine lots of UI components with different fragments into one initial data fetch.

It is possible for fragments to access variables declared in the query or mutation.

## Operation name

The operation type is either query, mutation, or subscription and describes what type of operation you're intending to do. 

The operation type is required unless you're using the query shorthand syntax.

It is only required in multi-operation documents, but its use is encouraged because it is very helpful for debugging and server-side logging.

## Variables

In most applications, the arguments to fields will be dynamic but wouldn't be a good idea to pass these dynamic arguments directly in the query string, because then our client-side code would need to dynamically manipulate the query string at runtime, and serialize it into a GraphQL-specific format.
 
GraphQL instead can pass values of of the query by passing as dicionary, these values are called variables.

In this way we can simply pass a different variable rather than needing to construct an entirely new query.

All declared variables must be either scalars, enums, or input object types.

Variable definitions can be optional or required.

```
// query
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
// variables
{
  "episode": "JEDI" // change this value
}
// result json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

Default values can also be assigned to the variables

```
query HeroNameAndFriends($episode: Episode = JEDI) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

## Directives

Allow to  dynamically change the structure and shape of our queries using variables. 

A directive can be attached to a field or fragment inclusion, and can affect execution of the query in any way the server desires.

The core GraphQL specification includes exactly two directives, which must be supported by any spec-compliant GraphQL server implementation:

@include(if: Boolean) Only include this field in the result if the argument is true.
@skip(if: Boolean) Skip this field if the argument is true.

Directives are useful in order to avoid string manipolation to add and remove fields in your query.

```
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
// variable
{
  "episode": "JEDI",
  "withFriends":true // change the result based on this variable
}
```

## Mutations

They are used to modify service-side data.

> In REST, any request might end up causing some side-effects on the server, but by convention it's suggested that one doesn't use GET requests to modify data. GraphQL is simila.

If the mutation field returns an object type, you can ask for nested fields. This can be useful for fetching the new state of an object after an update, so it is possible to mutate and query the new value of the field with one request.

A mutation can contain multiple fields, just like a query.

> While query fields are executed in parallel, mutation fields run in series, one after the other, this assure we do not get race conditions.

## Inline Fragments

GraphQL schemas include the ability to define interfaces and union types.
If you are querying a field that returns an interface or a union type, you will need to use inline fragments to access data on the underlying concrete type. This is done using the `on`.

```
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}
```

To ask for a field on the concrete type, you need to use an inline fragment with a type condition. Because the first fragment is labeled as ... on Droid, the primaryFunction field will only be executed if the Character returned from hero is of the Droid type. Similarly for the height field for the Human type.

## Meta fields

GraphQL allows you to request `__typename`, a meta field, at any point in a query to get the name of the object type at that point. This is useful when we do don't know what type is returned by the service and we need get more information on an union for instance.

```
  search(text: "an") {
    __typename
    ... on Human {
      name
    }
    ... on Droid {
      name
    }
    ... on Starship {
      name
    }
  }
}

{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo"
      },
      {
        "__typename": "Human",
        "name": "Leia Organa"
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1"
      }
    ]
  }
}
```

## The input type in a GraphQL

The input type in a GraphQL schema is a special object type that groups a set of arguments together, and can then be used as an argument to another field. Using input types helps us group and understand arguments, especially for mutations.

Using input types helps us group and understand arguments, especially for mutations.

Example:
```
input SearchListingsInput {
  checkInDate: String!
  checkOutDate: String!
  numOfBeds: Int
  page: Int
  limit: Int
  sortBy: SortByCriteria # this is an enum type
}

// use it

type Query {
  # ...
  "Search results for listings that fit the criteria provided"
  searchListings(criteria: SearchListingsInput): [Listing]! 
  # ...
}
```


## Authentication

In a REST API, authentication is often handled with a header, that contains an auth token which proves what user is making this request.
The same applies to GraphQL as the token can be passed in the header.

GraphQL offers very granular control over data. In GraphQL servers, individual field resolvers have the ability to check user roles and make decisions as to what to return for each user.

Alternative methods to pass a bearer token in GraphQL besides adding an Authorization header to the HTTP request.

- Include the token directly in the GraphQL query using a custom GraphQL directive. This approach is often referred to as "schema stitching" and can be useful when you have multiple services that require authentication with different token formats.

- A middleware function that intercepts the HTTP request before it reaches the GraphQL server and adds the Authorization header with the bearer token. This approach can be useful if you're using a third-party library or service to handle authentication and don't want to modify your GraphQL schema.

> Delegate authorization logic to the business logic layer not on the GraphQL layer.



## Resources

https://graphql.org/learn/queries/

https://the-guild.dev/blog/graphql-modules-auth


## GraphQL schema language

Every GraphQL service defines a set of types which completely describe the set of possible data you can query on that service. Then, when queries come in, they are validated and executed against that schema.

GraphQL services can be written in any language.


### Object types and fields

The most basic components of a GraphQL schema are object types

```
type Character { // Objet Type (type with some fields)
  name: String! // field String is a built-in scalar type, the ! means not nullable (otherwise all fields are nullable by default)
  appearsIn: [Episode!]! // array ! means not nullable
}
```

### Arguments

Every field on a GraphQL object type can have zero or more arguments. 

All arguments are named.

Arguments can be either required or optional, if optional we can define a default value.

```
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

### The Query and Mutation types

Every GraphQL service has a query type and may or may not have a mutation type. 

These types are the same as a regular object type, but they are special because they define the entry point of every GraphQL query.
 
```
schema {
  query: Query
  mutation: Mutation
}

query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```

That means that the GraphQL service needs to have a Query type with hero and droid fields:
 
 ```
 type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
 ```
 
Mutations work in a similar way - you define fields on the Mutation type, and those are available as the root mutation fields you can call in your query.

Query and Mutation types are the same as any other GraphQL object type, and their fields work exactly the same way.


## Scalar types

GaphQL comes with a set of default scalar types out of the box:

Int, Float, String, Boolean, ID (unique identifier not human readable)

In some GraphQL implementation is possible to have also Date


## Other types

Enum
```
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

List
`[Type]`

Non-Null
Using the `!`


example:

```
myField: [String]!

myField: null // error
myField: [] // valid
myField: ["a", "b"] // valid
myField: ["a", null, "b"] // valid
```

## Interfaces

 An Interface is an abstract type that includes a certain set of fields that a type must include to implement the interface.
 
 This means that any type that implements Character needs to have these exact fields, with these arguments and return types.
 
 ```
 interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
 ```
 
 You can add extra fields on top of an interface:
 
 ```
 type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}
 ```
 
 Interfaces are useful when you want to return an object or set of objects, but those might be of several different types.
 
 ## Union types
 
 Union types are very similar to interfaces, but they don't get to specify any common fields between the types.

```
union SearchResult = Human | Droid | Starship
```

 Note that members of a union type need to be concrete object types; you can't create a union type out of interfaces or other unions.
 
 ```
 {
  search(text: "an") {
    __typename
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}

{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo",
        "height": 1.8
      },
      {
        "__typename": "Human",
        "name": "Leia Organa",
        "height": 1.5
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1",
        "length": 9.2
      }
    ]
  }
}
 
 ```
 
 The __typename field resolves to a String which lets you differentiate different data types from each other on the client.
 
## Input types
 
You can also easily pass complex objects. This is particularly valuable in the case of mutations where you might want to pass in a whole object to be created.

## Validation

Type system, it can be predetermined whether a GraphQL query is valid or not. This allows servers and clients to effectively inform developers when an invalid query has been created, without having to rely on runtime checks.

Validation rules in place to ensure that a GraphQL query is semantically meaningful

##  Execution

After being validated, a GraphQL query is executed by a GraphQL server which returns a result that mirrors the shape of the requested query, typically as JSON.

GraphQL cannot execute a query without a type system.

> You can think of each field in a GraphQL query as a function or method of the previous type which returns the next type. In fact, this is exactly how GraphQL works. Each field on each type is backed by a function called the resolver which is provided by the GraphQL server developer. When a field is executed, the corresponding resolver is called to produce the next value. If a field produces a scalar value like a string or number, then the execution completes. However if a field produces an object value then the query will contain another selection of fields which apply to that object. This continues until scalar values are reached. GraphQL queries always end at scalar values.

## Root fields & resolvers

At the top level of every GraphQL server is a type that represents all of the possible entry points into the GraphQL API, it's often called the Root type or the Query type.

JavaScript example:
```
Query: {
  human(obj, args, context, info) {
    return context.db.loadHumanByID(args.id).then(
      userData => new Human(userData)
    )
  }
}
```

- obj The previous object, which for a field on the root Query type is often not used.
- args The arguments provided to the field in the GraphQL query.
- context A value which is provided to every resolver and holds important contextual information like the currently logged in user, or access to a database.
- info A value which holds field-specific information relevant to the current query 

## Asynchronous resolvers

Example:

```
human(obj, args, context, info) {
  return context.db.loadHumanByID(args.id).then(
    userData => new Human(userData)
 
```

The context is used to provide access to a database. Notice that while the resolver function needs to be aware of Promises, the GraphQL query does not.

A GraphQL server is powered by a type system which is used to determine what to do next.

## Scalar coercion

For conversion of scalatype for instance convert Enumb values to numbers.

## List resolvers

In this example. The resolver for this field is not just returning a Promise, it's returning a list of Promises. GraphQL will wait for all of these Promises concurrently before continuing, and when left with a list of objects, it will concurrently continue yet again to load the name field on each of these items.

```
Human: {
  starships(obj, args, context, info) {
    return obj.starshipIDs.map(
      id => context.db.loadStarshipByID(id).then(
        shipData => new Starship(shipData)
      )
    )
  }
}

```

## Producing the result 

As each field is resolved, the resulting value is placed into a key-value map with the field name (or alias) as the key and the resolved value as the value. This continues from the bottom leaf fields of the query all the way back up to the original field on the root Query type. Collectively these produce a structure that mirrors the original query which can then be sent (typically as JSON) to the client which requested it.

## Introspection

By querying the `__schema` field, always available on the root type of a Query it is possible to introspect the schema.

Double underscore, indicating that they are part of the introspection system the others are type system and built-in scalars.

Introspection system particularly useful for tooling.

## Best pacticsing

GraphQL services typically respond using JSON, however the GraphQL spec does not require it. JSON may seem like an odd choice for an API layer promising better network performance, however because it is mostly text, it compresses exceptionally well with GZIP.

GraphQL takes a strong opinion on avoiding versioning by providing the tools for the continuous evolution of a GraphQL schema.

GraphQL only returns the data that's explicitly requested, so new capabilities can be added via new types and new fields on those types without creating a breaking change. This has led to a common practice of always avoiding breaking changes and serving a versionless API.

In a GraphQL type system, every field is nullable by default. This is because there are many things that can go awry in a networked service backed by databases and other services. A database could go down, an asynchronous action could fail, an exception could be thrown. Beyond simply system failures, authorization can often be granular, where individual fields within a request can have different authorization rules. By defaulting every field to nullable, any of these reasons may result in just that field returned "null" rather than having a complete failure for the request. By defaulting every field to nullable, any of these reasons may result in just that field returned "null" rather than having a complete failure for the request. 

Typically fields that could return long lists accept arguments "first" and "after" to allow for specifying a specific region of a list, where "after" is a unique identifier of each of the values in the list.Ultimately designing APIs with feature-rich pagination led to a best practice pattern called "Connections". Some client tools for GraphQL, such as Relay, know about the Connections pattern and can automatically provide support for client-side pagination when a GraphQL API employs this pattern.
Different pagination models enable different client capabilities there are different ways, like Plurals:

```
{
  hero {
    name
    friends {
      name
    }
  }
}

{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

Or Slicing:
A client might want to be able to specify how many friends they want to fetch; maybe they only want the first two:

```
{
  hero {
    name
    friends(first: 2) {
      name
    }
  }
}
```

Pagination can be implemented using something like:

- We could do something like friends(first:2 offset:2) to ask for the next two in the list.
- We could do something like friends(first:2 after:$friendId), to ask for the next two after the last friend we fetched.
- We could do something like friends(first:2 after:$friendCursor), where we get a cursor from the last item and use that to paginate.

Cursor-based pagination is the most powerful of those designed (by making the cursor the offset or the ID and use base64 encoding them)

```
{
  hero {
    name
    friends(first: 2) {
      edges {
        node {
          name
        }
        cursor
      }
    }
  }
}

```

A connection object are used to provide additional information on the connection like totals and so on for instance:

Example with an `edge` if there is information that is specific to the edge, rather than to one of the objects.
The node rappresent the object info we want the cursor are kept in a different place.

```
{
  hero {
    name
    friendsConnection(first: 2, after: "Y3Vyc29yMQ==") {
      totalCount
      edges {
        node {
          name
        }
        cursor
      }
      pageInfo {
        endCursor
        hasNextPage
      }
    }
  }
}
```

This allow us to: 

- paginate through the list
- ask for information about the connection itself, like totalCount or pageInfo.
- information about the edge itself, like cursor or friendshipTime.


GraphQL is designed in a way that allows you to write clean code on the server, where every field on every type has a focused single-purpose function for resolving that value. However without additional consideration, a naive GraphQL service could be very "chatty" or repeatedly load data from your databases. This is commonly solved by a batching technique, where multiple requests for data from a backend are collected over a short period of time and then dispatched in a single request to an underlying database or microservice by using a tool like Facebook's DataLoader.

## Caching

> Providing Object Identifiers allows clients to build rich caches

GraphQL servers need to expose object identifiers in a standardized way.

The server must provide an interface called Node. That interface must include exactly one field, called id that returns a non-null ID.

This id should be a globally unique identifier for this object, and given just this id, the server should be able to refetch the object.

In GraphQL, there's no URL-like primitive that provides this globally unique identifier for a given object. It's hence a best practice for the API to expose such an identifier for clients to use.

In the same way that the URLs of a resource-based API provided a globally unique key, the id field in this system provides a globally unique key.


https://github.com/graphql/dataloader
