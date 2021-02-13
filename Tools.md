# Useful Tools

## Table of Contents
- [json-server](#json-server)
- [Setting up ESLint](#setting-up-eslint)
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

## Setting up ESLint

### Download VS-code extentions

ESLint, Prettier

### Npm install

```bash
npm i -D eslint prettier eslint-plugin-prettier eslint-config-prettier eslint-plugin-node eslint-config-node
```

Using the AirBnb styling guide
```bash
npx install-peerdeps --dev eslint-config-airbnb
```

### Create Prettier Config File
In .prettierrc:
```json
{
    "singleQuote": true
}
```
- All rules found [Here](https://prettier.io/docs/en/options.html)

### Generate ESLint config

We can generate it if we have ESLinst globally:

Install:
```bash
npm i -g  eslint
```

Generate:
```bash
eslint --init
```

To use the airbnb style guide, we can delete everything in .eslintrc.json and add:
```json
{
    "extends": ["airbnb", "prettier"],
    "plugins": ["prettier"],
    "rules": {
        "no-unused-vars": "warn",
        "no-console": "off",
        "func-names": "off",
    }
}
```