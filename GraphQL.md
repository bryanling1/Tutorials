# GraphQL

[Stephen Grider Course](https://www.udemy.com/course/graphql-with-react-course/)
## Table of contents

- [GraphQL](#graphql)
  - [Table of contents](#table-of-contents)
  - [1. REST-ful](#1-rest-ful)
  - [2. Query Example](#2-query-example)
  - [3. Express and GraphQL](#3-express-and-graphql)
  - [4. Our First Schema](#4-our-first-schema)
  - [5. Defining Our First Root Query](#5-defining-our-first-root-query)
  - [6. Making our First Query](#6-making-our-first-query)
  - [7. Our First Relation](#7-our-first-relation)
  - [8. Naming Queries](#8-naming-queries)
  - [9. Query Fragments](#9-query-fragments)
  - [10. Our First Mutation](#10-our-first-mutation)
  - [11. GraphQL Clients](#11-graphql-clients)

<a id="1"></a>
## 1. REST-ful

- implements **Representational Ssate Transfer**
- allows for interaction with **RESTful web services**
- Types of request and url
  - POST, GET, PUT, etc.
  - e.g. /name/:id


**The Problem**
- Nested/relational data can take many http requests and break rest conventions
- May be getting data we don't nead (over-serving)

<a id="2"></a>
## 2. Query Example

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

<a id="3"></a>
## 3. Express and GraphQL
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


<a id="4"></a>
## 4. Our First Schema
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

<a id="5"></a>
## 5. Defining Our First Root Query

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
<a id="6"></a>
## 6. Making our First Query
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

<a id="7"></a>
## 7. Our First Relation
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

<a id="8"></a>
## 8. Naming Queries
We can name our queries:
```js
query findUser{
  user(id: "2"){
    ...
  }
}
```

We can also name keys
```js
{
  apple: company(id: "1"){
    ...
  }
  google: company(id: "2"){
    ...
  }
}
```
- If we don't, an error will be thrown as our output will have two keys of the name `company`

<a id="9"></a>
## 9. Query Fragments
```js
fragment companyDetails on Company{
    id
    name
    description
}
```
apply:
```js
{
  apple: company(id: "1"){
    ...companyDetails
  }
  google: company(id: "2"){
    ...companyDetails
  }
}
```
<a id="10"></a>
## 10. Our First Mutation

Similar structure to a query
<a href="https://ibb.co/3R39RZw"><img src="https://i.ibb.co/TWfxW3Z/image.png" alt="image" border="0"></a>

Let's right a mutation that create a user
```js
const mutation = new GraphQLObjectType({
    name: 'Mutation',
    fields:{
        addUser:{
            type: UserType,
            args: {
                firstName: {type: new GraphQLNonNull(GraphQLString)},
                age: {type:new GraphQLNonNull(GraphQLInt)},
                companyId: {type: GraphQLString}
            },
            resolve(parentValue, {firstName, age}){
                return axios.post(`http://localhost:3000/users`, {
                    firstName, 
                    age,
                }).then(res=>res.data);
            }
        }
    }
});
```
- use `new GraphQLNonNull()` for required values
  
Then, we add it our GraphQL schema
```js
module.exports = new GraphQLSchema({
    query: RootQuery,
    mutation
})
```

To run the mutation:
```js
mutation{
  addUser(firstName: "Stephen", age: 26){
    id
    firstName
    age
  }
}
```
- The properties inside `{}` represent what we expect `res.data` to send back from our `resolve` 

<a id="11"></a>
## 11. GraphQL Clients

Acts as a bonding layer between React (frontend) and our Express/GraphQL server

<a href="https://ibb.co/tcyCkDp"><img src="https://i.ibb.co/Y8vjYhB/image.png" alt="image" border="0"></a>

We can also use **Apollo Server** instead of **GraphQL Express**

<a href="https://imgbb.com/"><img src="https://i.ibb.co/KNKsCcq/image.png" alt="image" border="0"></a>

