# Microservices
[Course by Stephen Grider](https://www.udemy.com/course/microservices-with-node-js-and-react/)
## Table of Contents
- [Microservices](#microservices)
  - [Table of Contents](#table-of-contents)
  - [Auth Service](#auth-service)
      - [1. Setup express](#1-setup-express)
      - [2. Apply start script to package.json](#2-apply-start-script-to-packagejson)
      - [Setup first docker file](#setup-first-docker-file)
      - [Build and push Auth image](#build-and-push-auth-image)
      - [Create a pod for our docker image](#create-a-pod-for-our-docker-image)
      - [Create a Cluster IP to access the pod](#create-a-cluster-ip-to-access-the-pod)
      - [Create a config file for our Nginx ingress Load-balancer](#create-a-config-file-for-our-nginx-ingress-load-balancer)
      - [Setup Skaffold](#setup-skaffold)
      - [Error Handling](#error-handling)
      - [Error handling in express](#error-handling-in-express)
      - [Using Express Validator](#using-express-validator)
      - [Writing our Error Handler Middleware](#writing-our-error-handler-middleware)
      - [Handling Express Async Errors](#handling-express-async-errors)
      - [Adding a Mongo Database](#adding-a-mongo-database)
      - [Adding Mongoose to our Project](#adding-mongoose-to-our-project)
      - [Creating a User MongoDB Model](#creating-a-user-mongodb-model)
      - [Getting Typescript and Mongoose to Work Together](#getting-typescript-and-mongoose-to-work-together)
      - [User Creation](#user-creation)
      - [Password Hashing](#password-hashing)
      - [Intercept Mongoose save attempt with Password logic](#intercept-mongoose-save-attempt-with-password-logic)
      - [Authentication strategies](#authentication-strategies)
      - [Solving Issues with Option 2](#solving-issues-with-option-2)
      - [JWT vs Cookie](#jwt-vs-cookie)
      - [Authentication Requirements](#authentication-requirements)
      - [Adding cookie-session](#adding-cookie-session)
      - [Generating JWT](#generating-jwt)
      - [Storing secrets with Kubernetes](#storing-secrets-with-kubernetes)
      - [Normalizing responses](#normalizing-responses)
  - [Testing](#testing)
      - [Scope of testing](#scope-of-testing)
      - [Testing Routes](#testing-routes)
      - [Dependencies](#dependencies)
      - [Test Environment Setup](#test-environment-setup)
      - [Writing our first routes test (signup)](#writing-our-first-routes-test-signup)
      - [Note on test-js](#note-on-test-js)
      - [Testing Failed Requests](#testing-failed-requests)
      - [Testing Cookies](#testing-cookies)
      - [Passing cookies](#passing-cookies)
  - [Server-Side-Rendering our React App](#server-side-rendering-our-react-app)

<a id="auth"></a>

## Auth Service

<a id="auth-1"></a>
#### 1. Setup express
Create a "routes" folder that exports like:
```js
import express from 'express';

const router = express.Router();

router.post('/api/users/signout', (req, res)=>{
    ...
})

export {router as signoutRouter};
```

And applies to our express app like:
```js
app.use(signoutRouter);
```
<a id="auth-2"></a>
#### 2. Apply start script to package.json
```json
"scripts": {
  "start": "ts-node-dev src/index.ts",
  ...
},
```
<a id="auth-3"></a>
#### Setup first docker file
```docker
FROM node:alpine

WORKDIR /app
COPY package.json .
RUN npm install --only=prod
COPY . .

CMD ["npm", "start"]
```

- `--only=prod` only installs dev dependences
- multiple COPY for caching

#### Build and push Auth image
```shell
docker build -t bryanling/auth .
```
```shell
docker push bryanling/auth
```
- K8s will pull our most recent version
- Skaffold will automatically do this for us

<a id="auth-4"></a>
#### Create a pod for our docker image
**auth-depl.yaml**
```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-depl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth 
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
        - name: auth
          image: bryanling/auth
```
- `selector` selects template's `metadata`

<a id="auth-5"></a>
#### Create a Cluster IP to access the pod
**In the same file**
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: auth-srv
spec:
  selector:
    app: auth
  ports:
    - name: auth
      protocol: TCP
      port: 3000
      targetPort: 3000
```
<a id="auth-6"></a>
#### Create a config file for our Nginx ingress Load-balancer

[Install Instructions](https://kubernetes.github.io/ingress-nginx/deploy/)

Setup **ingress-srv.yaml**
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: 'true'
spec:
  rules:
    - host: ticketing.dev
      http:
        paths:
          - path: /api/users/?(.*)
            backend:
              serviceName: auth-srv
              servicePort: 3000
```

We need to setup an alias for `ticketing.dev` to point at our localhost found in the `C:\Windows\System32\Driverss\etc\hosts` file

<a id="auth-7"></a>
#### Setup Skaffold

To auto detect changes and update k8s klusters: [Install](https://skaffold.dev/docs/install/)

Yaml in project directory
```yaml
apiVersion: skaffold/v2alpha3
kind: Config
deploy:
  kubectl:
    manifests:
      - ./infra/k8s/*
build:
  local:
    push: false
  artifacts:
    - image: bryanling/auth
      context: auth
      docker:
        dockerfile: Dockerfile
      sync:
        manual:
          - src: 'src/**/*.ts'
            dest: .

```
<a id="auth-8"></a>
#### Error Handling

We want all of our errors to take the same form across our application. We will create an abstract class to build off which extends the built in Error class:
```js
export abstract class CustomError extends Error{
    abstract statusCode:number;
    constructor(message:string){
        super(message);
        Object.setPrototypeOf(this, CustomError.prototype);
    }

    abstract serializeErrors(): {message:string, field?:string}[];
}
```
- We need to call `Object.setPrototypeOf(this, CustomError.prototype);` for built in classes
- Base `Error` class has a `message` property, so we call `super(message)`

<a id="auth-9"></a>
#### Error handling in express

If a sync route, throw passes the error to the next middleware

```js
app.get('/', function(req, res){
  throw new Error('ERROR')
})
```

If an async route, we need to use next
```js
  app.get('/', function(req, res, next){
    fs.readFile('/bad-file', function(err, data){
      err ? next(err) : res.send(data);
    })
  })
```

**Writing error handlers**

If the middleware has 4 functions, express identifies as error handler

```js
app.use(function(err, req, res, next){
  //handle err
})
```

#### Using Express Validator

We can use Express Validator to verify results in our request body
```ts
import express, {Request, Response} from 'express';
import {body} from 'express-validator';
import {validateRequest} from '../middlewares/validate-request';

const router = express.Router();

router.post('/api/users/signin',[
    body('email').isEmail().withMessage('Email Must be valid'),
    body('password').trim().notEmpty().withMessage('You must supply a password')
], validateRequest, async(req:Request, res:Response)=>{
    ...
})
```
**validateRequest:**
```ts
import {Request, Response, NextFunction} from 'express';
import {validationResult} from 'express-validator';

export const validateRequest = (req: Request, res:Response, next: NextFunction) =>{
    const errors = validationResult(req);

    if(!errors.isEmpty()){
        throw new ... //errors.array()
    }

    next();
}
```

#### Writing our Error Handler Middleware

```ts
import { Request, Response, NextFunction} from 'express';
import {CustomError} from '../errors/custom-error';

export const errorHandler = (
  err:Error, 
  req:Request, 
  res:Response, 
  next:NextFunction
) => {
    if(err instanceof CustomError){
        return res.status(err.statusCode).send({errors: err.serializeErrors()});
    }

    res.status(400).send({
        errors:[{
            message: err.message
        }]
    })
}
```
- We would add this to our app with ```app.use(errorHandler)```

#### Handling Express Async Errors

Remember that express handles async errors differently: we need to use `next()`

We can get arround this by using the npm library `express-async-errors` right after we import express where we declare app:

```js
import express from 'express';
import 'express-async-errors';
...
const app = express();
```

#### Adding a Mongo Database

First create our deployment: 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-mongo-depl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth-mongo
  template:
    metadata:
      labels:
        app: auth-mongo
    spec:
      containers:
        - name: auth-mongo
          image: mongo
```

Create a ClusterIP service in the same file:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: auth-mongo-srv
spec:
  selector:
    app: auth-mongo
  ports:
    - name: db
      protocol: TCP
      port: 27017
      targetPort: 27017
```
- `27017` is the default listening port for mongodb
- If we **delete** or **restart** the pod, we will lose all of its data

#### Adding Mongoose to our Project

Let's connect to our MongoDB pod (in the /auth directory)
```js
try{
  await mongoose.connect('mongodb://auth-mongo-srv:27017/auth', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useCreateIndex: true
  })
}
catch(err){
  console.log(err);
}
```
- `auth-mongo-srv` is what we set as `name` from our **ClusterIP's metadata**
- if a `/auth` collection doesn't exist, MongoDB will create on for us

#### Creating a User MongoDB Model

Create a new folder /models

Create a schema for our User model:
```js
import mongoose from 'mongoose';

const userSchema = new mongoose.Schema({
  email: {
    type: string,
    required: true,
  },
  password:{
    type: string,
    required: true,
  }
})

const User = mongoose.model({'User', userSchema})

export {User};
```

#### Getting Typescript and Mongoose to Work Together

Two Issues we want to solve: 
1. We cant TypeScript to check that we are providing the correct properties when creating a user
2. Make TypeScript understand that there will be other properties other than the ones we provide on creation
   
**Issue 1**
Instead of writing `new User({})` we are going to create our own function.
That way we can to type checking in TypeScript.

We can add a static function to our `User` model.

With our `userSchema`:
```ts
interface UserAttrs{
  email: string;
  password: string;
}

userSchema.statics.build = (attrs: UserAttrs) => {
  return new User(attrs)
}
```

However, if we try to run `User.build({})` TypeScript will throw an error. We have to tell TypeScript that `.build()` exists on our **User Model**

```ts
interface UserModel extends mongoose.Model<any>{
  build(attrs: UserAttrs):any;
}

const User = mongoose.model<any, UserModel>({'User', userSchema})
```
**Issue 2**
Now we need to describe the properties that a **User Document** has


Let's apply this interface
```ts
interface UserModel extends mongoose.Model<UserDoc>{
  build(attrs: UserAttrs):UserDoc;
}

const User = mongoose.model<UserDoc, UserModel>({'User', userSchema})
```
- Replace all `any` from before with our `UserDoc` interface

#### User Creation
Let's import our `User` model into our Signup Route
```ts
const {email, password} = req.body;

const existingUser = await User.findOne({email});

//if the user exists
if(existingUser){
  throw new BadRequest('Email in use')
}

const user = User.build({email, password});
await user.save();

res.status(201).send(user);
```
- We create a user by using the static `build({})` method we created earlier
- We would create a `BadRequest()` error from our `CustomError` abstract class
  - We used a statusCode of **400**

#### Password Hashing

We're going to create a class the handles passwords

```ts
import {scrypt, randomBytes} from 'crypto';
import {promisify} from 'util'

const scryptAsync  = promsifu(scrypt);

export class Password{
  static async toHash(password:string){
    ...
  }
  static compare(storedPassword:string, suppliedPassword:string){
    ...
  }
}
```

To hash a password:
```ts
static async toHash(password:string){
  const salt = randomBytes(8).toString('hex');
  const buffer = (await scriptAsync(password, salt, 64)) as Buffer;
  return `${bugger.toString('hex').salt}`
}
```
- We use as Buffer to tell TypeScript the typing
  
To compare, we apply the `salt` to some other string
```ts
static compare(storedPassword:string, suppliedPassword:string){
  const [hashedPassword, salt] = storedPassword.split('.');
  const buffer = (await scriptAsync(suppliedPassword, salt, 64)) as Buffer;

  return buffer.toString('hex') === hashedPassword;
}
```

#### Intercept Mongoose save attempt with Password logic

```ts
userSchema.pre('save', async function(done){
  if (this.isModified('password')){
    const hashed = await Password.toHash(this.get('password'));
    this.set('password', hashed);
  }
  done();
})
```
- Middleware applied before a user is saved
- Mongoose does not have great async support. After we are done, we are responsible for calling `done()`
- We don't use an **arrow function** because `this.` referse to the **user document** instead of **the context of the file** (not overwritten)
- We use an `if` statement, because we only want to hash the password if it has been modified

#### Authentication strategies

There is no one right way

**1. Individual services rely on the auth services**

<a href="https://ibb.co/vXDrSbf"><img src="https://i.ibb.co/y4VMrDK/image.png" alt="image" border="0"></a>
- If the authentication services goes down, all services that are **dependent will fail**

**1.1 Authentication gateway**
<a href="https://ibb.co/frNHvQC"><img src="https://i.ibb.co/qmgp5xR/image.png" alt="image" border="0"></a>

**2. Tell each service how a user is Authenticated**
<a href="https://ibb.co/z6pDZDk"><img src="https://i.ibb.co/Y0mwhwr/image.png" alt="image" border="0"></a>
- Not outside dependices
- Duplicate code (not a big deal, we can make it a class)
- If a user is "banned" on one service, their JWT may still be valid to another service. There is **no centralized** check

#### Solving Issues with Option 2

We can implement a system where the JWT expires after some time.

If the user's token is expired, we go tell the user to log back in.

<a href="https://ibb.co/fX61fCc"><img src="https://i.ibb.co/4FvmzjQ/image.png" alt="image" border="0"></a>
- Short lived because the JWT refreshes every 15 seconds anyway

#### JWT vs Cookie

**Cookies:** 
- Transport mechanism
- Moves any kind of data between vrwoser and server
- Automatically manged by the browser
<a href="https://ibb.co/YRFTMqk"><img src="https://i.ibb.co/FxLqMjw/image.png" alt="image" border="0"></a>

**JWT:**
- A Authentication/Authorization mechanism
- Stores any data we want
- We have to mange it manually
<a href="https://ibb.co/wywtn1H"><img src="https://i.ibb.co/v1Q9N85/image.png" alt="image" border="0"></a>

#### Authentication Requirements

<a href="https://ibb.co/S3pXywP"><img src="https://i.ibb.co/Nyz6C2s/image.png" alt="image" border="0"></a>

- We need to know information about the user: Has credit card to buy, is admin user, etc.

- We need a secure way to expire our JWT after a period of time

- Mechanism should be understood by multiple different languages: We might have express for auth, ruby on rails for orders services, etc.

- Must not require some kind of backing data store

JWT allows us to meet these requirements!

But, we still need a transport mechanism. 

We can do this in 3 ways:

<a href="https://ibb.co/fxV8kk9"><img src="https://i.ibb.co/mcrv990/image.png" alt="image" border="0"></a>

We will use server side rendering for SEO and page speed on older devices

<a href="https://ibb.co/d2dp2JW"><img src="https://i.ibb.co/DCxMCgV/image.png" alt="image" border="0"></a>

The problem:
<a href="https://ibb.co/HpzmSP3"><img src="https://i.ibb.co/BKZ8p6F/image.png" alt="image" border="0"></a>

That means we only have one solution: JWT!
<a href="https://ibb.co/sWWpkxH"><img src="https://i.ibb.co/5KKXtpF/image.png" alt="image" border="0"></a>

We can get around this with service workers, but we do not cover this in this course

<a href="https://imgbb.com/"><img src="https://i.ibb.co/s9hg0qS/image.png" alt="image" border="0"></a>

#### Adding cookie-session
`npm install cookie-session @types/cookie-session`

Wire it up as a middleware

```ts
app.use(
  cookieSession({
    signed: false,
    secure: true
  })
)

```
- `signed` is `false` because we are not encrypting the cookie
- `secure` is https

Since express is under a proxy with nginx, we need to tell express this is okay. Right after we declare our app:
```ts
const app = express();
app.set('trust proxy', true);
```

Now we can access on `req.session`

#### Generating JWT
`npm install jsonwebtoken @types/jsonwebtoken`

Right after our user signup, we will generate:

```ts
import jwt from 'jsonwebtoken'
...
await user.save();
const userJwt = jwt.sign({
  id: user.id,
  email: user.email
}, 'our private key')

req.session = {
  jwt: userJwt
}
```

#### Storing secrets with Kubernetes

<a href="https://ibb.co/R7JSKwv"><img src="https://i.ibb.co/TrNTG3v/image.png" alt="image" border="0"></a>
- access via environment variable

`kubectl create secret generic jwt-secret --from-literal=JWT_KEY=asdf`
- `generic` is a type of secret
- `jwt-secret` is what we named in
- Need to re-run everytime we restart out cluster

We can get a list of all our secrets with: `kubectl get secrets`

Now we have to tell our auth service to reference that secret:

```yaml
    ...
    spec:
      containers:
        - name: auth
          image: bryanling/auth]
          env:
            - name: JWT_KEY
              valueFrom:
                secretKeyRef:
                  name: jwt-secret
                  key: JWT_KEY
```
- If k8s cannot find the secret, it will fail to run the pod

Now we can use that env in our project

We should do a check for Typescript at the very beginning of our server:

```ts
const start = async() =>{
  if(!process.env.JWT_KEY){
    throw new Error('JWT_KEY must be defined')
  }
  ...
}
```
Anywhere else in the code, we can just use `process.env.JWT_KEY` as we can be confident it is defined

#### Normalizing responses
<a href="https://ibb.co/CzddMZ2"><img src="https://i.ibb.co/rmPPcSs/image.png" alt="image" border="0"></a>
- For example: MongoDB has id as _id

We can override how Javascript converts an object to JSON

```js
const person = {name: 'alex', toJSON(){1}}

console.log(person)
//returns "1"
```

We can use this same idea when defined our **Mongoose Schemas**

```ts
const userSchema = new mongoose.Schema({...}, 
{
  toJSON: {
    transform(doc, ret){
      ret.id = ret._id;
      delete ret._id;
      delete ret.password;
      delete ret.__V;
    }
  }
})
```
- `delete` is JS, allows to remove a property from an object

## Testing

#### Scope of testing

We are going to test each service individually. 

**Test Goals**
1. Basic request handling
2. Mongoose models
3. Event emitting + receiving

We are going to run these tests directly from our terminal without using docker

This implies our **local environment** is capable of running each service

More complex projects may make this hard (talk about this later)

<a href="https://ibb.co/xScFWrB"><img src="https://i.ibb.co/wNbcV2P/image.png" alt="image" border="0"></a>

#### Testing Routes

<a href="https://ibb.co/m5MYBrT"><img src="https://i.ibb.co/3mL8s5d/image.png" alt="image" border="0"></a>

We are going to reconfigure our auth service that just exports the express app

#### Dependencies

`npm install --save-dev @types/jest @types/suerptest jest ts-jest supertest mongodb-memory-server`

We don't want to have to reinstall these dev dependencies, so in the docker file for our auth service we can write

```docker
...
RUN npm install --only=prod
...
```

#### Test Environment Setup
We are going to add a script in our package.json
```json
"scripts":{
  ...
  "test":"jest --watchAll --no-chache"
}
```
- `--no-chache` because Jest does not always recognise when we **change Typescript files**

Also in our package.json:
```json
"jest":{
  "preset":"ts-jest",
  "testEnvironment":"node",
  "setupFilesAfterEnv":[
    "./src/test/setup.ts"
  ]
}
```

Let's create `"./src/test/setup.ts"`

Imports
```ts
import {MongoMemoryServer} from 'mongodb-memory-server';
import mongoose from 'mongoose';
import {app} from '../app';
```
- `app` is our express app

Setup the mongo memory server 
```ts
let mongo:any;
beforeAll(async()=>{
  mongo = new MongoMemoryServer();
  const mongoUri = await mongo.getUri();

  await.mongoose.connect(mongoUri, {
    useNewUrlParser: true,
    useUnifiedTopology: true
  })
})
```

Before ever test we want to delete all the collections:
```ts
beforeEach(async()=>{
  const collections = await mongoose.connection.db.collections();

  for(let collection of collections){
    await collection.deleteMany({})
  }
})
```

And disconnect after every test

```ts
afterAll(async()=>{
  await mongo.stop();
  await mongoose.connection.close();
})
```

#### Writing our first routes test (signup)

In our routes folder (following jest convention) we will create:

- &#95;&#95;test&#95;&#95;
  - signup.test.ts

```ts
import request from 'supertest';
import {app} from '../../app';

it('returns a 201 on succesful signup', async()=>{
  return request(app)
    .post('/api/users/signup')
    .send({
      email: 'test@test.com',
      password: 'password'
    })
    .expect(201)
})
```

If we try to run `npm run test` now, we will get a 400 error.

This is because when we create our **JWT**, we are checking for `process.env.JWT_KEY` which our test does not hav access to.

Quick fix, we can make sure we define this environment variable before our test

```ts
let mongo:any;
beforeAll(async()=>{
  process.env.JWT_KEY = '12341234124';
  ...
})
```

#### Note on test-js

Sometimes even after we update our Typescript, our tests may fail. Sometimes jest or ts-jest does not detect changes we make to our files. 

All we have to do is rerun `npm run test`

#### Testing Failed Requests
With supertest
```ts
it('returns a 400 with an invalid email', async()=>{
  await request(app)
    .post('/api/users/signup')
    .send({
      email: 'asefsadfsef',
      password: 'password'
    })
    .expect(400);
})
```
- As long as we put `await`, Jest will automatically return for us. Useful for when we have to create a user first and sign in etc.

#### Testing Cookies
```ts
it('sets a cookie after successful signup', async()=>{
  const response = await request(app)
    .post('/api/users/signup')
    .send({
      email: 'test@test.com',
      password: 'password'
    })
    .expect(201);
  
  expect(respons.get('Set-Cookie')).toBeDefined();
})
```
- We can tell a cookie is set by inspecting the reponse's header
- This test will fail because our reponse is not secure in our test environment. Let's make a change to our express app for cookies (secure property):
  ```ts
  app.use(
    cookieSession({
      signed: false,
      secure: process.env.NODE_ENV !== 'test'
    })
  )
  ```

#### Passing cookies

Supertest does not automatically send cookie data after each request. We can accomplish passing by simply extracting the cookie from the response and setting it in the following:

```ts
const response = await request(app)...

const cookie = response.get('Set-Cookie');

await request(app).set('Cookie', cookie)...
```
We can make this more straight forward by creating a helper function.

We are going to write a global function, but we can also put this inside a nother file.

```ts
declare global{
  namespace NodeJS{
    interface Global{
      signin(): Promise<string[]>
    }
  }
}

global.signin = async () =>{
  const email = 'test@test.com';
  const password = 'password';

  const response = await request(app)
    .post('api/users/signup')
    .send({
      email,
      password
    })
    .expect(201);
  
  const cookie = reqponse.get('Set-Cookie');

  return cookie;
}
```
Now we can get our cookie with 
```ts 
  const cookie = await global.signin()
```

## Server-Side-Rendering our React App