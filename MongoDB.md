# Mongo DB

[Stephen Grider Course](https://www.udemy.com/course/the-complete-developers-guide-to-mongodb/learn/lecture/6040218#overview)

[Github Repo](https://github.com/StephenGrider/MongoCasts)

## Table of Contents

- [Mongo DB](#mongo-db)
  - [Table of Contents](#table-of-contents)
  - [First Project Users](#first-project-users)
      - [Init](#init)
      - [Setup tests](#setup-tests)

<a href="https://ibb.co/52SRssK"><img src="https://i.ibb.co/fQyxrrD/image.png" alt="image" border="0"></a>

## First Project Users

<a href="https://ibb.co/xGwdhwt"><img src="https://i.ibb.co/gwQhyQL/image.png" alt="image" border="0"></a>
<a href="https://imgbb.com/"><img src="https://i.ibb.co/jDzxFv1/image.png" alt="image" border="0"></a>

#### Init

```
npm init -y
```

```
npm install mongoose nodemon mocha
```

#### Setup tests

<a href="https://imgbb.com/"><img src="https://i.ibb.co/XbWcfgx/image.png" alt="image" border="0"></a>


Inside `test/test_helper.js`

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/users_test');

mongoose.connection.once('open', () => {

    console.log('Good to go!');

}).on('error', () => {

    console.warn('Warning', error);

})
```


