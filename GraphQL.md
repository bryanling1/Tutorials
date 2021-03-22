# GraphQL

[Stephen Grider Course](https://www.udemy.com/course/graphql-with-react-course/)
## Table of Contents
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
  - [12. Setting Up Apollo Client](#12-setting-up-apollo-client)
  - [13. Our First React Query](#13-our-first-react-query)
  - [14. Query Variables](#14-query-variables)
  - [15. First React Mutation](#15-first-react-mutation)
  - [16. Cache Flow/Rerunning Queries](#16-cache-flowrerunning-queries)
  - [17. Combining Query and Mutation to a Component](#17-combining-query-and-mutation-to-a-component)
  - [18. Passing Query Variables](#18-passing-query-variables)
  - [19. Apollo Store Caching/Normalizing with dataIdFromObject](#19-apollo-store-cachingnormalizing-with-dataidfromobject)
  - [20. Optimistic Responses](#19-optimistic-responses)

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

## 12. Setting Up Apollo Client
<a href="https://imgbb.com/"><img src="https://i.ibb.co/f4xNr78/image.png" alt="image" border="0"></a>

```js
import ApolloClient from 'apollo-client';
import { AppolloPrivder } from 'react-apollo';

const client = new ApolloClient({});

const Root = () => {
  return(
    <ApolloClient client={client}>
      ...
    </ApolloClient>
  )
}

ReactDOM.render(
  <Root/>,
  ...
);
```
- `ApolloClient({})` assumes we are using `/graphql` on our backend

## 13. Our First React Query

Example project in React
<a href="https://ibb.co/0FCHbV3"><img src="https://i.ibb.co/jWynXZF/image.png" alt="image" border="0"></a>
<a href="https://ibb.co/Cbf6Jjr"><img src="https://i.ibb.co/sKZC5TB/image.png" alt="image" border="0"></a>

Let's write a query to get all the song titles:
```js
import gql from 'graphql-tag';

const query = gql`
  {
    songs{
      title
    }
  }
`

```
- GraphQL is not valid Javacsript, so we use `graphlql-tag`;

Now we need to combine this query with a component:

```js
import {graphql} from 'apollo-client';

...

export default graphql(query)(SomeComponent)
```

<a href="https://ibb.co/y8VcJ1z"><img src="https://i.ibb.co/K9mpHkt/image.png" alt="image" border="0"></a>

- Now we can access the data in props

The object that is returned has a `loading` property which is useful for rendering data

```jsx
<div>
{props.data.loading && <div>Loading</div>}
{!props.data.loading && renderSongs()}
</div>
```

## 14. Query Variables

Used when we want to inject a variable into a gql query
<a href="https://ibb.co/rFfxrvN"><img src="https://i.ibb.co/djJPY7x/image.png" alt="image" border="0"></a>
In GraphiQL:
<a href="https://ibb.co/NTBvrLW"><img src="https://i.ibb.co/phF7Q34/image.png" alt="image" border="0"></a>

- Same for queries, but write `query` instead of `mutation`

## 15. First React Mutation

```jsx
const mutation = gql`
  mutation AddSong($title: String){
    addSong(title: $title){
      title
    }
  }
`

export default graphql(mutation)(Component);
```

Now we can call via:

```js
props.mutate({
  variables:{
    title: title_var
  }
})
```
- returns a promise

## 16. Cache Flow/Rerunning Queries

If we render **"songs"** and then make a new one, we will not see our newest **song**
<a href="https://ibb.co/5smzBZ1"><img src="https://i.ibb.co/MPrjMdg/image.png" alt="image" border="0"></a>

We need to rerun our query

```js
props.mutate({
  variables: {title: titleVar},
  refetchQueries: [{ query:queryVar, variables:variablesVar }]
})

```
- Use the `refetchQueries` property
- If we try to run 2 of the same query, GraphQL will detect this for us

We can also:
```js
props.mutate({...}).then(()=> props.data.refetch())
```
- Only works with `props.data` is associated with the mutation
## 17. Combining Query and Mutation to a Component

```js
export default graphql(mutation)(
  graphql(query)(component)
)
```

## 18. Passing Query Variables

```js
export default graphql(fetchSone,{
  options: (props) => {return {variables: {id: props.params.id}}}
})(Component)
```
<a href="https://imgbb.com/"><img src="https://i.ibb.co/nL4gswK/image.png" alt="image" border="0"></a>

## 19. Apollo Store Caching/Normalizing with dataIdFromObject

Currently, Apollo store cannot identify what we are rendinering:
<a href="https://imgbb.com/"><img src="https://i.ibb.co/92HLL5D/image.png" alt="image" border="0"></a>

We need to tell it to do this so it can rerender our component: 

<a href="https://imgbb.com/"><img src="https://i.ibb.co/ysP8v1W/image.png" alt="image" border="0"></a>

<a href="https://imgbb.com/"><img src="https://i.ibb.co/72VSh0T/image.png" alt="image" border="0"></a>

When we first define our client:
```js
const client = new ApolloClient({
  dataIdFromObject: o => o.id
})
```
- Now evertime we return an id from a mutation, Apollo will know to rerender

## 20. Optimistic Responses

<a href="https://imgbb.com/"><img src="https://i.ibb.co/qRQvNqx/image.png" alt="image" border="0"></a>

```js
const onLike = (id, likes) =>{
  props.mutate({
    variables: ...,
    optimisticResponse:{
      __typename: 'Mutation',
      likeLyric: {
        id: id,
        __typename: 'LyricType',
        likes: likes + 1
      }
    }
  })
}
```
- `likeLyric` is the type of data
  
## Authentication Project from Scratch

[Starter](https://github.com/StephenGrider/auth-graphql-starter)
<a href="https://ibb.co/3kBCjzs"><img src="https://i.ibb.co/d7G5Vgc/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/jgDccCf"><img src="https://i.ibb.co/RzcxxXb/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/THPCDQJ"><img src="https://i.ibb.co/JzrhWY8/image.png" alt="image" border="0"></a>

#### The User Type

Inside `server/schema/types/user_type.js`
```ts
const { graphql } = require('graphql');
const {
  GraphQLObjectType,
  GraphQLStringType,
}

const Usertype = new GraphQLObjectType({
  name: "UserType",
  fields: {
    email: { type: GraphQLString }
  }
})

module.exports = Usertpe;
```

#### Signup Mutation

We want to put all the "business logic" for resolves in a seperate file. This will clean up our mutations

<a href="https://ibb.co/TBF478r"><img src="https://i.ibb.co/XZv7KsV/image.png" alt="image" border="0"></a>

Inside `server/schema/mutations.js`

```ts
const { graphql } = require('graphql');
const {
  GraphQLObjectType,
  GraphQLStringType,
}
const UserType = require('./types/user_type');
const AuthService = require('../services/auth');

const mutation = new GraphQLObjectType({
  name: 'Mutation',
  fields: {
    signup: {
      type: UserType,
      args: {
        email: { type: GraphQLString },
        password: { type: GraphQLString }
      },
      resolve(parenValue, {email, password}, req) {
        return AuthService.signup({email, password, req })
      }
    }
  }
});

module.exports = mutation;
```
  - `request` is the request object from **express**
  - Since `AuthService` returns a promise, we need to make sure we `return`

Now we can test out this query in GraphiQL

Before we do that however, we need to pass a fummyField to our rootQuery temporarily, otherwise GraphQL will throw an error

Inside `server/schema/types/root_query_type.js`

```js
...
const RootQueryType = new GraphQLObjectType({
  name: 'RootQueryType',
  fields: {
    dummyField: { type: GraphQLID }
  }
});
...
```

