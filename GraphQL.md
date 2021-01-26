# GraphQL

[Stephen Grider Course](https://www.udemy.com/course/graphql-with-react-course/)
## Table of contents

## REST-ful

- implements **Representational Ssate Transfer**
- allows for interaction with **RESTful web services**
- Types of request and url
  - POST, GET, PUT, etc.
  - e.g. /name/:id

## The Problem
- Nested/relational data can take many http requests and break rest conventions
- May be getting data we don't nead (over-serving)

## Query Example

GraphQL takes something like this:

<a href="https://ibb.co/B6WZtLh"><img src="https://i.ibb.co/Jp67KnY/image.png" alt="image" border="0" width="350px"></a>

And turns it into this: 

<a href="https://imgbb.com/"><img src="https://i.ibb.co/7zQ3Ggq/image.png" alt="image" border="0" width="350px"></a>

If we wanted **User:23 &rarr; friends &rarr; companies**, our query would look like:

```json
query {
    user(id: "23"){
        friends{
            company{
                name
            }
        }
    }
}
```
## Express and GraphQL
<a href="https://imgbb.com/"><img src="https://i.ibb.co/s3KNkBZ/image.png" alt="image" border="0" width="350px"></a>

```js
const express = require('express');
const { graphqlHTTP } = require('express-graphql');

const app = express();
app.use('/graphql', graphqlHTTP({
    graphiql: true,
    schema: ...,
}))
```

## Our First Schema
Example:
<a href="https://imgbb.com/"><img src="https://i.ibb.co/P1VsccQ/image.png" alt="image" border="0" width="350px"></a>

Defing a **User Schema**
```js
const graphql = require('graphql');

const {
    GraphQLObjectType,
    GraphQLString,
    GraphQLInt,
} = graphql;

const UserType = new GraphQLObjectType({
    name: 'User',
    fields: {
        id: {type: GraphQLString},
        firstName: {type: GraphQLString},
        age: {type: GraphQLInt},
    }
})
```
- `GraphQLString` & `GraphQLInt` are types

## Our First Root Query

<a href="https://imgbb.com/"><img src="https://i.ibb.co/nw0zwd7/image.png" alt="image" border="0" width="350px"></a>
The root query describes how to find nodes in our graph.
```js
const RootQuery = new GraphQLObjectType({
    name: 'RootQueryType',
    fields: {
        user:{
            type: UserType,
            args: {id: {type: GraphQLString}},
            resolve(parentValue, args){
                ... //return user
            },
        }
    }
});
```
- `UserType` was defined earler
- Says: give me `id: {type: GraphQLString}` and I will give you `type: UserType`
- `resolve` carriers out the query