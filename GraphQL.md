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

Defing a **User Schema** in schema.js:
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

## Defining Our First Root Query

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
                return axios.get('/some-url').then(res => res.data)
            },
        }
    }
});
```
- `UserType` was defined earler
- Says: give me `id: {type: GraphQLString}` and I will give you `type: UserType`
- `resolve` carriers out the query
- `.then(res => res.data)` because of how **axios** returns data

Let's export our RootQuery:
```js
module.exports = new GraphQLSchema({
    query: RootQuery
})
```
And apply it to our GraphQL middleware
```js
const schema = require('./schema/schema');

app.use('/graphql', graphqlHTTP({
    graphiql: true,
    schema,
}))
```
## Making our First Query
Using GraphiQL
```js
{
  user(id: "23"){
    id, 
    firstName,
    age
  }
}
```
will return 
```js
{
  "data": {
    "user": {
      "id": "23",
      "firstName": "Bob",
      "age": 20
    }
  }
}
```
- GraphQL does all the type matching for us, in our `resolve` function we don't have to specify of "type user"

## Our First Relation
Let's say each user has a company Id associated with it:
<a href="https://imgbb.com/"><img src="https://i.ibb.co/kXLP6hc/image.png" alt="image" border="0"></a>

Define our company schema
```js
const CompanyType = new GraphQLObjectType({
    name: 'Company',
    fields: ()=> ({
        id: {type: GraphQLString},
        name: {type: GraphQLString},
        description: {type: GraphQLString}
    })
});
```
We can then add this type to our `Users` schema
```js
const UserType = new GraphQLObjectType({
    name: 'User',
    fields: ()=> ({
        id: {type: GraphQLString},
        firstName: {type: GraphQLString},
        age: {type: GraphQLInt},
        company: {
            type: CompanyType
        }
    })
})
```
- We use `fields: ()=>({})` because `fields: {}` will cause a JS error if we are using GraphQLObjectType that are defined later in the code
  
Let's say our **database** looks like this: 
```json
{
    "users": [
        {"id":"23", "firstName": "Bill", "age": 20, "companyId":"1"},
        {"id":"22", "firstName": "Alice", "age": 22,  "companyId":"2"},
        {"id": "24", "firstName": "Gwen", "age": 25, "companyId":"2"}
    ],
    "companies": [
        {"id":"1", "name": "Apple", "description":"iphone"},
        {"id":"2", "name":"Google", "description": "search"}
    ]
}
```
Comparing our **User schema** to the **database**, :

<a href="https://imgbb.com/"><img src="https://i.ibb.co/Yy0kK6K/image.png" alt="image" border="0"></a>

We need to write a resolve to match the **User's companyId** with the id of the company from the **database**

```js
const UserType = new GraphQLObjectType({
    ...
        company: {
            type: CompanyType,
            resolve(parentValue, args){
                return axios.get(`http://localhost:3000/companies/${parentValue.companyId}`)
                        .then(res=>res.data)
            }
        },
})
```
- `parentValue` contains the data from our **User**

Our current process:
<a href="https://imgbb.com/"><img src="https://i.ibb.co/LRMNdsG/image.png" alt="image" border="0"></a>
- `args` is `id:"23"`

<a href="https://imgbb.com/"><img src="https://i.ibb.co/1Jjnx4j/image.png" alt="image" border="0"></a>
- Each arrow is a `resolve` function

