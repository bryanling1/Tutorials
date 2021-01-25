# GraphQL

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

## Our first query

GraphQL takes something like this:

<a href="https://ibb.co/B6WZtLh"><img src="https://i.ibb.co/Jp67KnY/image.png" alt="image" border="0" width="400px"></a>

And turns it into this: 

<a href="https://imgbb.com/"><img src="https://i.ibb.co/7zQ3Ggq/image.png" alt="image" border="0" width="400px"></a>

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