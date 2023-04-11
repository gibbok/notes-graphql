# Notes on GraphQL

GraphQL is a query language for your API, and a server-side runtime for executing queries using a type system you define for your data.

GraphQL is created by defined types and fields, then providing functions for each field:

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

GraphQL allows you to request `__typename`, a meta field, at any point in a query to get the name of the object type at that point. This is useful wje we do don't know what type is returned by the service.

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
```
