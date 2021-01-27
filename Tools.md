# Useful Tools

## Table of Contents

##[json-server](https://www.npmjs.com/package/json-server)

Create a db.json file
```json
{
    "users": [
        {"id":"23", "firstName": "Bill", "age": 20},
        {"id":"22", "firstName": "Alice", "age": 22}
    ]
}
```

Startup in package.json
```json
"scripts": {
    "server": "json-server --watch db.json"
  },
```