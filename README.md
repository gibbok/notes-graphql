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
