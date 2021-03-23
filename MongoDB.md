# Mongo DB

[Stephen Grider Course](https://www.udemy.com/course/the-complete-developers-guide-to-mongodb/learn/lecture/6040218#overview)

[Github Repo](https://github.com/StephenGrider/MongoCasts)

## Table of Contents

- [Mongo DB](#mongo-db)
  - [Table of Contents](#table-of-contents)
  - [First Project Users](#first-project-users)
      - [Init](#init)
      - [Setup tests](#setup-tests)
      - [User Model](#user-model)
      - [Basics of Mocha](#basics-of-mocha)
      - [Tests for our User Model](#tests-for-our-user-model)
      - [Default Promise Implementation](#default-promise-implementation)
      - [Test Setup for Finding Users](#test-setup-for-finding-users)
      - [Automating tests with Nodemon](#automating-tests-with-nodemon)
      - [Finding Particular Records](#finding-particular-records)
      - [The Many Ways to Remove Records](#the-many-ways-to-remove-records)

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

before((done)=>{

  mongoose.connect('mongodb://localhost/users_test');

  mongoose.connection.once('open', () => {

      console.log('Good to go!');
      done();

  }).on('error', () => {

      console.warn('Warning', error);

  })

})

```

#### User Model

Inside `user.js`

```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const UserSchema = new Schema({
  name: String
});

const User = mongoose.model('user', UserSchema);

module.exports = User;
```
- When create `User` model, mongoose looks for that **collection**. If not found it creates one
- `User` represents the entire collection of data

#### Basics of Mocha

Inside `test/create_test.js`
```js
const assert = require('assert')

describe('Creating records', () => {
  it('saves a user', () => {
    assert(1 + 1 === 2)
  })
})

```
- `assert` is installed with `mocha`

Let's create a test script in `package.json`

```json
"scripts":{
  "test":"mocha"
}
```

#### Tests for our User Model

```js
const assert = require('assert')
const User = require('../user')

describe('Creating records', () => {
  it('saves a user', (done) => {
    const joe = new User({ name: 'Joe' });
    
    joe.save().then(()=>{
      assert(!joe.isNew);
      done();
    });
  })
})
```
- mongoose has property `isNew`, `true` if the database has **not been saved**
- This causes a promise warning that we will cover next
  
We want to delete our db whenever we run test scripts

Inside `test/test_helper`

```js
...
beforeEach((done)=>{
  mongoose.connection.collections.users.drop(()=>{
    done();
  });
})
```
- This takes some time, so we need to tell mocha to wait for `drop()` to complete, se we add `done()`

#### Default Promise Implementation

Inside `test/test_helper.js`

```js
const mongoose = require('mongoose');

mongoose.Promise = global.Promise;

...
```

#### Test Setup for Finding Users

We can find using `User.find(criteria)` or `User.findOne(criteria)` where find returns all 

Inside `test/reading_test.js`

```js
const assert = require('assert')
const User = require('../src/user')

describe('Reading users out of the database', () => {

  let joe;
  beforeEach((done)=>{
    joe = new User({ name: 'joe' })
    joe.save().then( () => done() )
  })
  it('finds all users iwth a name of joe', (done) => {
    User.findOne({name:'joe'}).then((users)=>{
      assert(users[0]._id.toString() === joe._id.toString())
      done();
    })
  })
})

```
- `users[0]._id` is an `ObjectId`, so `users[0]._id === joe._id` will fail for camprisons. We need to call `.toString()`

#### Automating tests with Nodemon

Inside `package.json`

```json
"scripts":{
  "test": "nodemon --exec 'mocha -R min'"
}
```
- `mocha --watch` sometimes has problems with mongoose, so we will use `nodemon` which restarts from the stary every time a change is found
- `-R` clears out existing input in the console
- `min` displays the minimum output information

#### Finding Particular Records

inside `test/reading_test.js`

```js
...
it('find a user iwth a particular id', (done) => {
  User.findOne({_id: joe._id}).then( (user) => {
    assert(user.name === 'Joe' );
    done();
  })
})
```

#### The Many Ways to Remove Records

**Model Class**
  - `remove`
  - `findOneAndRemove`
  - `findByIdAndRemove`

**Model Instance**
  - `remove`


Inside `test/delete_test`

```js
const assert = require('assert');
const User = require('../src/user');

describe('Deleting a user', (done) => {
  let joe;
  beforeEach(()=>{
    joe = new User({ name: "Joe" })
    joe.save().then(() => done() )
  })

  it('model instance remove', (done) => {
    joe.remove()
      .then(() => User.findOne({ name: 'Joe' }))
      .then((user)=>{
        assert(user === null);
        done();
      })
  })

  it('class method remove', (done) => {
    //remove a bunch of different records
    User.remove({ name: "Joe" }) ...
  })

  it('class method findOneAndRemove', (done) => {
    User.findOneAndRemove({name: "Joe"}) ...
  })
  it('model instance findByIdAndRemove', (done) => {
    User.findByIdAndremove(joe._id) ...
  })
})
```


