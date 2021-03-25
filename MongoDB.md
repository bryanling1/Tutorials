# Mongo DB

[Stephen Grider Course](https://www.udemy.com/course/the-complete-developers-guide-to-mongodb/learn/lecture/6040218#overview)

[Github Repo](https://github.com/StephenGrider/MongoCasts)

## Table of Contents

- [Mongo DB](#mongo-db)
  - [Table of Contents](#table-of-contents)
  - [Project 1: Users](#project-1-users)
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
      - [Update Operators](#update-operators)
      - [Adding a Validation](#adding-a-validation)
      - [Handling failed inserts](#handling-failed-inserts)
      - [Embedding a Posts Schema (subdocuments)](#embedding-a-posts-schema-subdocuments)
      - [Testing subdocuments](#testing-subdocuments)
      - [Adding subdocuments to existing records](#adding-subdocuments-to-existing-records)
      - [Removing Subdocuments](#removing-subdocuments)
      - [Virtual types](#virtual-types)
      - [Challenges of Nested Resources](#challenges-of-nested-resources)
      - [Creating Associations with Refs](#creating-associations-with-refs)
      - [Test Setup for Associations](#test-setup-for-associations)
      - [Deeply Nested Associations](#deeply-nested-associations)
      - [Mongoose Middleware](#mongoose-middleware)
      - [Testing Pre-Remove Middleware](#testing-pre-remove-middleware)
      - [Pagination and Sorting](#pagination-and-sorting)
  - [Project 2: Upstar Music](#project-2-upstar-music)
      - [Finding Particular Records](#finding-particular-records-1)
      - [Min and Max values in a collection](#min-and-max-values-in-a-collection)
      - [Sorting, Limiting, Skipping, Search Query](#sorting-limiting-skipping-search-query)
      - [Filtering By Single Properties](#filtering-by-single-properties)
      - [Handling Text Search](#handling-text-search)
      - [Updating Multiple](#updating-multiple)
      - [Fixing the results count](#fixing-the-results-count)
  - [MongoDB and Express: Uber Driver App](#mongodb-and-express-uber-driver-app)
      - [Working with Moch and Express](#working-with-moch-and-express)
      - [Refactoring for Controllers and Models](#refactoring-for-controllers-and-models)
      - [Why we Express needs a bodyParser](#why-we-express-needs-a-bodyparser)
      - [Testing Driver Creation](#testing-driver-creation)

<a href="https://ibb.co/52SRssK"><img src="https://i.ibb.co/fQyxrrD/image.png" alt="image" border="0"></a>

## Project 1: Users

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
  "test": "nodemon --exec 'mocha --recursive -R min'"
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

#### Update Operators

[Mongo Update Operators](https://docs.mongodb.com/manual/reference/operator/update/) are good for updating multiple entries

Let's add a `postCount` properties for our `UserSchema`

Inside `test/update_test`

```js
...
it('A user can have their postCount incremented by 1' , () => {
    User.update({name:"Joe"}, {$inc: {postCount: 1}})
        .then(()=>User.findOne({name:'Joe'}))
        .then((user)=>{
            assert(user.postCount === 1
            done()
        }))
})
```

#### Adding a Validation

<a href="https://imgbb.com/"><img src="https://i.ibb.co/frjnW0v/image.png" alt="image" border="0"></a>

We don't want users to input junk data into our database

Inside `test/validation_test.js`
```js
const assert = require('assert')
const User = require('../src/user')

describe('Validation records', () => {
    it('requires a user name' , () => {
        const user  = new User({name: undefined});
        const validationResult = user.validateSync();
        const message = validationResult.errors.name.message;

        assert(message === "Name is required")
    })

    it('requires a user name longer than 2 characters' , () => {
        const user  = new User({name: 'Al'});
        const validationResult = user.validateSync();
        const message = validationResult.errors.name.message;

        assert(message === 'Name must be longer than 2 characters')
    })
})
```

Inside `src/user`, let's update our UserSchema

```js
const UserSchema = new Schema({
    name: {
        type: String,
        validate: {
            validator: (name) => name.length > 2,
            message: 'Name must be longer than 2 characters'
        },
        required: [true, 'Name is required']
    },
    postCount: Number
})
```

#### Handling failed inserts

Inside `test/validation_test.js`

```js
it('dissalows invalid records from being save', (done)=>{
    const user = new User({name: 'Al'})

    user.save().catch(validationResult=>{
        const {message} = validationResult.errors.name;

        assert(message === 'Name must be longer than 2 characters');
        done();
    })
})
```

#### Embedding a Posts Schema (subdocuments)

<a href="https://imgbb.com/"><img src="https://i.ibb.co/Wtq8XdH/image.png" alt="image" border="0"></a>

Inside `src/posts.js`

```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema;


const PostSchema = new Schema({
    title: String
})

module.exports = PostSchema
```

Inside `src/user.js`

```js
const PostSchema = require('./post');

const UserSchema = new Schema({
    ...
    posts: [PostSchema]
})
```

#### Testing subdocuments

Inside `test/sobducments_test.js`

```js
const assert = require('assert')
const User = require('../src/user')

describe('Subdocuments', () => {
    it('can create a subdocument', (done) => {
        const joe = new User({
            name: 'Joe', 
            posts: [{title: 'PostTitle'}]
        })
        joe.save()
            .then(()=>{
                User.findOne({name: 'Joe'})
            })
            .then(user=>{
                assert(user.posts[0].title === 'PostTitle')
                done();
            })
    })
})
```

#### Adding subdocuments to existing records

We use `push` and then `save`

Inside `test/sobducments_test.js`

```js
it('Can add subdocuments to an existing record', (done)=>{
    const joe = new User({
        name: 'Joe',
        posts: []
    })
    joe.save()
        .then(()=>User.findOne({name: 'Joe'}))
        .then((user)=>{
            user.posts.push({ title: 'New Post' });
            return user.save();
        }).then(()=>{
            return User.findOne({name: 'Joe'})
        }).then((user)=>{
            assert(user.posts[0].title === 'New Post')
            done();
        })
})
```

#### Removing Subdocuments

We can use raw js, but Mongoose knows its kindof a pain to use `splice`

We can use `.remove()`

`.remove()` won't automitacally delete in sub documents, we still need to save the ducment instance

Inside `test/sobducments_test.js`

```js
it('Can remove an existing subdocument', (done)=>{
    const joe = new User({
        name: 'Joe',
        posts: [{title: 'New Title'}]
    })
    joe.save()
        .then(()=>User.findOne({name: 'Joe'}))
        .then((user)=>{
            const post = user.posts[0];
            post.remove();
            return user.save();
        }).then(()=>{
            return User.findOne({name: 'Joe'})
        }).then((user)=>{
            assert(users.posts.length === 0);
            done();
        })
})
```

#### Virtual types

We want `postCount` to reflect the number of posts, we should need to have an additionl counter

We can make use of **Virtual Types**

<a href="https://ibb.co/jwjdvQS"><img src="https://i.ibb.co/JpSNRG9/image.png" alt="image" border="0"></a>

Inside `test/virtual_type_test.js`

Let's first write up the test
```js
const assert = require('assert');
const User = require('../scr/User');

describe('Virtual types', () => {
    it('postCount returns number of posts', (done) => {
        const joe = new User({
            name: 'Joe',
            posts: [{title: 'PostTitle'}]
        })

        joe.save()
            .then(()=>User.findOne({name: 'Joe'}))
            .then((user)=>{
                assert(joe.postCount === 1);
                done();
            })
    })
})
```

Now let's make the Virtual type

Inside `src/user.js`

First we will remove the `postCount` property we defined earlier

We would also have to remove any `increment` tests with that property

```js
...
UserSchema.virtual('postCount').get(function(){
    return this.posts.length;
})
```
- `get` is es6
- We don't use an arrow function because we need access to `this`

#### Challenges of Nested Resources

<a href="https://ibb.co/qx2WgmV"><img src="https://i.ibb.co/b7G2vND/image.png" alt="image" border="0"></a>
- different users

<a href="https://ibb.co/ZXrK86H"><img src="https://i.ibb.co/543v8WY/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/7yqKLP8"><img src="https://i.ibb.co/YjxhwmY/image.png" alt="image" border="0"></a>

- Queries become more complex, can't do a single query like in an SQL database

#### Creating Associations with Refs

We will rename `posts` to `blogPosts`

In `src/blogPost.js`

```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const BlogPostSchema = new Schema({
    title: String,
    content: String,
    comments: [{
        type: Schema.Types.ObjectId,
        ref: 'Comment'
    }]
})

const BlogPost = mongoose.model('blogPost', blogPostScehma);

module.exports = BlogPost;
```
- Ref is the string we defined in model. For example when we created the user model we said `const User = mongoose.model('user', UserSchema)` where `user` would be the ref

In `src/comments.js`

```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const CommentSchema = new Schema({
    content: String;
    user: {
        type: Schema.Types.ObjectId,
        type: 'user',
    }
})

const Comment = mongoose.model('comment', CommentSchema)

export.modules = Comment;
```

In `src/user.js`

```js
...
blogPosts: [{
    type: Schema.Types.ObjectId,
    ref: 'blogPost'
}]
```

#### Test Setup for Associations

We should include deleting all that data in `test/test_helper` for our new `Comment` and `BlogPost` models

```js
beforeEach((done) => {
    const { users, comments, blogposts } = mongoose.connection.collections;
    users.drop(()=>{
        comments.drop(()=>{
            blogposts.drop(()=>{
                done();
            })
        })
    })
})
```
- We can't drop mutliple collections on Mongo at the same time
- `blogposts` because mongo normalize by making collection names all lowercase

Inside `test/association_test.js`

```js
const assert = require('asssert')
const mongoose = require('mongoose');
const User = require('../src/user');
const Comment = require('../src/comment');
const BlogPost = require('../src/blogPost');

describe('Associations', () => {
    let joe, blogPost, comment
    beforeEach((done)=>{
        joe = new User({name: 'Joe'})
        blogPost = new BlogPost({title: 'JS is great', content: 'reee'})
        comment = new Comment({content: 'wow cool post man'})

        joe.blogPosts.push(blogPost);
        blogPost.comments.push(comment);
        comment.user = joe;

        Promise.all([joe.save(), blogPost.save(), comment.save()])
            .then( ()=>{ done() });
    })

    it.only('saves a relation between  user and a blogpost', (done) => {
        User.findOne({name: 'Joe'})
            .populate('blogPost')
            .then(user=>{
                assert(user.blogPost[0].title === 'JS is great')
                done()
            })
    })
})
```
- If we only want to run one test in our test suite, we can run `it.only()`
- `populate` is a mongo modifier. If we did not use this, `comments` would just be an array of `ObjectId`s


<a href="https://ibb.co/jf3GDYp"><img src="https://i.ibb.co/H4GYqyJ/image.png" alt="image" border="0"></a>

#### Deeply Nested Associations

Inside `test/association_test.js`

```js
it('saves a full relation graph', (done) => {
    User.findOne({ name: 'Joe' })
        .popoulate({
            path: 'blogPosts',
            populate: {
                path: 'comments',
                model: 'comment',
                populate:{
                    path: 'user',
                    model: 'user',
                }
            }
        })
        .then(user=>{
            assert(user.name === 'Joe');
            assert(...)
            done();
        })
})
```
- Once we start nesting, we must also specify the `Model` we want to refer to

#### Mongoose Middleware

If a `User` document gets deleted, we will want to also delete their `BlogPost`s and `Comments`

We need a way to do cleanup

We can install `pre` and `post` save hooks in `mongoose`

We create middlewawre on the model definitions

Inside `src/user.js`

```js
...
UserSchema.pre('remove', function(next){
    const BlogPost = mongoose.mofel('blogPost');
    BlogPost.remove({_id: { $in: this.blogPosts}})
        .next(()=>next());
})

```
- We should never `.foreach()` 
- `$in` says look through `BlogPost`, if it is in `this.blogPosts`, remove it

#### Testing Pre-Remove Middleware

Inside `test/middleware_test.js`

```js
const mongoose = require('mongoose');
const assert = require('assert');
const User = require('../src/user');
const BlogPost = require('../src/blogPost');

describe('Middleware', ()=>{
    beforeEach((done)=>{
        joe = new User({name: 'Joe'})
        blogPost = new BlogPost({title: 'JS is great', content: 'reee'})

        joe.blogPosts.push(blogPost);

        Promise.all([joe.save()])
            .then( ()=>{ done() });
    })

    it('users clean up dangling blogposts on remove', (done) => {
        joe.remove()
            .then(()=>{
                return BlogPost.count()
            })
            .then(count=>{
                assert(count === 0);
                done();
            })
    })
})
```

#### Pagination and Sorting

We can use `skip` and `limit`

In `test/reading_test.js`

```js
...
    let joe, maria, alex, zach;

    beforeEach((done)=>{
        alex = new User({name: 'Alex'})
        maria = new User({name: 'Maria'})
        joe = new User({name: 'Joe'})
        zach = new User({name: 'Zach'})

        Promise.All([alex.save(), maria.save(), joe.save(), zach.save()])
            .then(()=>done())
    })

    it('can skip and limit the result set', (done) => {
        User.find({}).skip(1).limit(2).sort({ name: 1 })
            .then((users)=>{
                assert(users.length === 2);
                assert(users[0].name === 'Joe');
                assert(users[1].name === 'Maria');
                done();
            })
    })
```
- We sort in this example as `Promise.all()` does not guarantee the order
- 1: is ascending, -1 is descending

## Project 2: Upstar Music

[Skeleton Project](http://github.com/stephengrider/upstarmusic)

#### Finding Particular Records

Inside `queries/FindArtist.js`

```js
const Artist = require('../models/artist');

module.exports = (_id) =>{
    return Artis.findById(_id);
}
```

#### Min and Max values in a collection

Inside `queries/GetAgeRamge.js`

```js
const Artist = require('../models/artist');

module.exports = () =>{
    const minQuery = Artist
        .find({})
        .sort({age:1})
        .limit(1)
        .then(artists=>artists[0].age)
    
    const maxQuery = Artist
        .find({})
        .sort({age:-1})
        .limit(1)
        .then(artists=>artists[0].age)
    
    return Promise.all([minQuery, maxQuery])
        .then(result=>{
            return{min: result[0], max: result[1]}
        })
}
```
- Do note use `.then(artist=>artists[0])` due to bad performance

#### Sorting, Limiting, Skipping, Search Query

Inside `queries/SearchArtist.js`

```js
const Artist = require('../models/artist');

module.exports = (criteria, sortProperty, offset=0, limit=20) =>{
    
    const query = Artist.find(buildQuery(criteria))
        .sort([sortProperty]: 1)
        .skip(offset)
        .limit(limit)
    
    return Promise.all([query, Artist.count()])
        .then(results=>{
            return{
                all: results[0],
                count: results[1],
                offset,
                limit
            }
        })
}
```

#### Filtering By Single Properties

We will create a helper function `buildQuery()`

Still inside `queries/SearchArtist.js`

```js
const buildQuery = (criteria) => {
    const query = {};

    if(criteria.age){
        query.age = {
            $gte: critera.age.min,
            $lte: criteria.age.max
        }
    }

    if(criteria.yearsActive){
        query.yearsActive = {
            $gte: critera.yearsActive.min,
            $lte: criteria.yearsActive.max
        }
    }

    return query
}
```

#### Handling Text Search

In order to use the `$text` operator, we need to have **Text Index**

**Indexes** help improve performance on Mongo queries and can be added to properties to boost performance

At the time of this course, you can only have one index on a Schema property

We can add it via the command line
```
mongo
```
```
show dbs
```
```
use upstar_music
```
```
db.artists.createIndex({ name: "text" })
```

```js
const buildQuery = (criteria) => {
    const query = {};

    if(criteria.name){
        query.$text = { $search: criteria.name }
    }
    ...
    return query
}
```

#### Updating Multiple

Inside `queries/SetRetired.js`

```js
const Artist = require('../models/artist');

module.exports = (_ids) => {
    return Artist.update(
        { _id: { $in: _ids }},
        { retired: true },
        { multi: true }
    )
}
```
- We need to add `multi` or else it will only update the first

#### Fixing the results count

Back Inside `queries/SearchArtist.js`

```js
const Artist = require('../models/artist');

module.exports = (criteria, sortProperty, offset=0, limit=20) =>{
    
    const query = Artist.find(buildQuery(criteria))
        .sort([sortProperty]: 1)
        .skip(offset)
        .limit(limit)
    
    return Promise.all([query, Artist.find(buildQuery(criteria)).count()])
        .then(results=>{
            return{
                all: results[0],
                count: results[1],
                offset,
                limit
            }
        })
}
```
- We will replace `Artist.count()` with `Artist.find(buildQuery(criteria)).count()`
- At the time of this course, we have to make 2 queries and there is no work around

## MongoDB and Express: Uber Driver App

<a href="https://ibb.co/x8SrRGw"><img src="https://i.ibb.co/Nn6GHpf/image.png" alt="image" border="0"></a>

#### Working with Moch and Express

We can make fake http requests with `supertest`

```
npm install supertest
```

Inside `test/app_test.js`

```js
const assert = require('assert')
const request = require('supertest')
const app = require('../app');

describe('The express app', () => {
    it('handles a GET request to /api', (done) => {
        request(app)
            .get('/api')
            .end((err, response)=>{
                assert(response.body.hi === 'some string');
                done();
            })
    })
})
```

#### Refactoring for Controllers and Models

Inside `routes/routes.js`

```js
const DriversController = require('../controllers/drivers_controller');

app.get('/api', DriversController.greeting)
```

Inside `controllers/drivers_dontroller`

```js
module.exports = {
    greeting(req, res) {
        res.send({hi: 'there'})
    }
}
```

#### Why we Express needs a bodyParser

<a href="https://imgbb.com/"><img src="https://i.ibb.co/PTkc9Ng/image.png" alt="image" border="0"></a>

#### Testing Driver Creation

Inside `test/controllers/drivers_controller_test.js`

```js
const assert = require('assert');
const request = require('supertest');
const app = require('../../app');

describe('Drivers controller', () => {
    it('Creates a new driver', (done) => {
        request(app)
            .post('/api/drivers')
            .send({ email: 'test@test.com' })
            .end(()=>{
                done();
            })
    })
})
```