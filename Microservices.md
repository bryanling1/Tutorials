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
      - [Getting started with NextJS](#getting-started-with-nextjs)
      - [Building our Next app Image](#building-our-next-app-image)
      - [Adding our app to Kubernetes](#adding-our-app-to-kubernetes)
      - [Note on file change detection](#note-on-file-change-detection)
      - [Wiring up Bootstrap](#wiring-up-bootstrap)
      - [Account Signup](#account-signup)
      - [The useRequest Custom Hook](#the-userequest-custom-hook)
      - [Figuring out if the user is signed in](#figuring-out-if-the-user-is-signed-in)
      - [2 NextJS with Ingress Nginx Solutions](#2-nextjs-with-ingress-nginx-solutions)
      - [More on `.getInitialProps`](#more-on-getinitialprops)
      - [Passing through cookies](#passing-through-cookies)
      - [Creating a Reusable API Client](#creating-a-reusable-api-client)
      - [Passing `GetInitialProps` to rest of components](#passing-getinitialprops-to-rest-of-components)
  - [NPM Setup](#npm-setup)
      - [Shared logic Between Services](#shared-logic-between-services)
      - [Options for Code sharing](#options-for-code-sharing)
      - [Publishing NPM Modules](#publishing-npm-modules)
      - [NPM and Typescript](#npm-and-typescript)
      - [Publish command](#publish-command)
      - [Better Importing](#better-importing)
      - [Updating](#updating)
  - [Ticketing Service](#ticketing-service)
      - [Mongo Setup](#mongo-setup)
      - [Writing tests for our routes](#writing-tests-for-our-routes)
      - [Ticket Mongoose Model](#ticket-mongoose-model)
      - [Testing Document Creations](#testing-document-creations)
      - [Testing getting a ticket](#testing-getting-a-ticket)
      - [Updating a ticket](#updating-a-ticket)
  - [NATS Streaming Server - An Event Bus Implementation](#nats-streaming-server---an-event-bus-implementation)
      - [Creating a NATS Streaming Deployment](#creating-a-nats-streaming-deployment)
      - [Big Notes on NATS Streaming](#big-notes-on-nats-streaming)
      - [Building a NATS Test Project](#building-a-nats-test-project)
      - [Client ID Generation](#client-id-generation)
      - [More Subscribe Options](#more-subscribe-options)
      - [Client Health Checks](#client-health-checks)
      - [Core Concurrency Issues](#core-concurrency-issues)
      - [Solving Concurrency Issues](#solving-concurrency-issues)
      - [Event Redelivery](#event-redelivery)
      - [NATS listeners as a class](#nats-listeners-as-a-class)
      - [Extending the Listener](#extending-the-listener)
      - [Overview](#overview)
      - [Custom Publisher](#custom-publisher)
      - [Event Definitions Summary](#event-definitions-summary)
      - [Publishing Ticket Creation Event](#publishing-ticket-creation-event)
      - [Adding NATS client to our ticketing service](#adding-nats-client-to-our-ticketing-service)
      - [Creating NatsWrapper Class](#creating-natswrapper-class)
      - [Graceful shutdown](#graceful-shutdown)
      - [Failed Event Publishing](#failed-event-publishing)

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
```ts
interface UserDoc extends mongoose.Document{
    email: string;
    password: string;
}   
```

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

<a href="https://ibb.co/6JkhmZ4"><img src="https://i.ibb.co/XkqBb4z/image.png" alt="image" border="0"></a>

#### Getting started with NextJS

We will create a client folder as such:
- client
  - package.json
  - pages
    - index.js
  
`npm init -y` then `npm install next react react-dom`

NextJS will interpret the pages folder as distinct routes in our application

In our package.json
```json
...
"scripts":{
  "dev":"next"
}
...
```

For this app, we will not use TS

#### Building our Next app Image

Let's create a Dockerfile in our client directory:
```docker
FROM node:alpine

WORKDIR /app
COPY package.json
RUN npm install
COPY . .

CMD ["npm", "run", "dev"]
```

And a docker ignore file:

```
node_modules

.next
```

Make sure we push this up to dockerHub
#### Adding our app to Kubernetes

Creating client-deply.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-depl
spec:
  replicas: 1
  selector: 
    MatchLabels:
      app: client
  template:
    metadata:
      labels:
        app: client
    spec:
      containers:
        - name: client
        image: bryanling/client
---
apiVersion: v1
kind: Service
metadata:
  name: client-srv
spec: 
  selector:
    app: client
  ports:
    - name: client
      protocol: TCP
      port: 3000
      targetPort: 3000
```

Let's update our scaffold.yaml file:
```yaml
...
    - image: bryanling/client
      context: client
      docker: 
        dockerfile: Dockerfile
      sync:
        manual:
          - src: '**/*.js'
          dest: .
```

Finally, we need to update our ingree configuration so we can access our next app from the outside world

```yaml
...
      - path: /?(.*)
        backend:
          serviceName: client-srv
          servicePort: 3000
```
- We want to list this last because the array matches the paths in order

#### Note on file change detection

NextJS sometimes struggles to see file changes. We can add some config to help:

In our `client` folder, create `next.config.js`
```js
module.exports = {
  webpackDevMiddleware: config => {
    config.watchOptions.poll = 300;
    return config
  }
}
```

This change is applied on start, so we need to restart the pod with `kubectl delete pod podid`. The pod will then automatically restart

We can also restart the pod if we still fail to see changes to our js files

#### Wiring up Bootstrap

First `npm install bootstrap`

In our pages folder, we will create `_app.js`
```js
import 'bootstrap/dist/css/bootstrap.css';

export default ({Component, pageProps}) => {
  return <Component {...pageProps}>
}
```
- Whenever we go to a route in our `pages` folder, Next will pass our react component to `Component`, acts as a wrapper
- We can only import globals through this file (bootstrap in this case)

#### Account Signup

```tsx
import axios from 'axios';

const onSubmit = async event =>{
  event.preventdefault();
  try{
    const response = await axios.post('/api/users/signup', {
      email, password
    })
  }catch(e){
    setError(e)
  }
}
```
- This will automatically set the cookie. You can chen in inspect element, network tab

#### The useRequest Custom Hook

<a href="https://ibb.co/Lrkf8NY"><img src="https://i.ibb.co/qCywdJ7/image.png" alt="image" border="0"></a>

```tsx
import axios from 'axios';
import {useState} from 'react';

export default({url, method, body}) =>{
  const [errors, setErrors] = useState(null);

  const doRequest = async () =>{
    try{
      const response = await axios[method](url, body);
      return response.data
    }catch(err){
      setErrors(
        <div>{err.response.data.errors.map(err=><div key={err}>{err}</div>)}</div>
      )
    }
  }

  return {doRequest, errors}
}
```

Now we can use back in our signup form

```tsx
const {doRequest, erros} = useRequest({
  url: 'api/users/signup',
  method: 'post',
  body:{
    email, password
  }
})

const onSubmit = async event =>{
  event.preventdefault();
  doRequest();
}
```

Let's add a callback so we can redirect the user

To redirect, we can use:

```jsx
import Router from 'next/router';

Router.push('/')
```

Back at our hook
```jsx
export default({url, method, body, onSuccess}) =>{
  ...
  try{
    ...
    if(onSuccess){
      onSuccess();
    }
  }catch(err){
    ...
  }
  return {doRequest, errors}
}
```

#### Figuring out if the user is signed in 
<a href="https://ibb.co/p2FWbyL"><img src="https://i.ibb.co/0qbBmtG/image.png" alt="image" border="0"></a>

We need to figure out how to send a request while our Next JS application is building

<a href="https://ibb.co/LRm5XWb"><img src="https://i.ibb.co/T26LxSp/image.png" alt="image" border="0"></a>

```jsx
const LandingPage = ({color}) =>{
  return <div>{color}</div>
}

LandingPage.getInitialProps = () =>{
  return {color: 'red'}
}

export default LandingPage
```
- with `.getInitialProps`, we can set props on the component
- `.getInitialProps` is executed on the server

Now we can use this to make a request to `/api/users/currentuser`

```js
import axios from 'axios';

...
LandingPage.getInitialProps = async () =>{
  const response = await axios.get('/api/users/currentuser')
  return {color: 'red'}
}
```
- we can't use our `getRequest` custom hook because `.getInitialProps` is not a component
- we are also not allowed to fetch data from inside a component during the server side rendering process

If we run this now we will get this error:

<a href="https://ibb.co/StMG2B3"><img src="https://i.ibb.co/5rSwdLR/image.png" alt="image" border="0"></a>

Becuase we only called `/api/users/currentuser` in axios, our browser assumes we are using the same domain as the client.

<a href="https://ibb.co/37xQZJS"><img src="https://i.ibb.co/JKYgJhr/image.png" alt="image" border="0"></a>

However, since NextJS is running inside of its own container, it tries to map the domain
with itself (its own 1270.0.0.1) and has no reference to nginx's load balancer

#### 2 NextJS with Ingress Nginx Solutions

Tell ther server side to access a different route
<a href="https://ibb.co/82wk63Q"><img src="https://i.ibb.co/L6yKnws/image.png" alt="image" border="0"></a>
- **Option #2** uses the name/domain of our k8s services. This is not ideal because we have to remember the names and which routes correspond to those services
- **Option #1:** Luckily, all that information is already encoded in **Ingress Nginx**
- The question is **what domain** do we use inside a cluster to talk to **Ingress Nginx**

We also need to keep information about cookies

<a href="https://ibb.co/2g9yhxs"><img src="https://i.ibb.co/SRjPybd/image.png" alt="image" border="0"></a>

We need to pass the cookie from the original request off to **Ingress Nginx**

We need to cross namespaces

<a href="https://ibb.co/PCnsGm5"><img src="https://i.ibb.co/6sxc0n4/image.png" alt="image" border="0"></a>

To figure out services that run **insde the ingress-nginx namespace**, we run:
```bash
kubectl get services -n ingress-nginx
```

The name of the **Load balancer** service is also **ingress-nginx**, the domain we use
is `http://ingress-nginx.ingress-nginx.svc.cluster.local/api/users/currentuser`

We can create an **External Name Service** to make this name easier

<a href="https://ibb.co/pW9wcRS"><img src="https://i.ibb.co/Q81QLC4/image.png" alt="image" border="0"></a>

We won't do this in this class

#### More on `.getInitialProps`

<a href="https://ibb.co/pysmjRC"><img src="https://i.ibb.co/HdMJTxm/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/5sT7pST"><img src="https://i.ibb.co/vL1tyT1/image.png" alt="image" border="0"></a>

To see whether we are on the server or the browser:

```js
LandingPage.getInitialProps = async () => {
  if (typeof window === 'undefined'){
    //we are on the server
    const response = await axios.get(
      'http://ingress-nginx.ingress-nginx.svc.cluster.local/api/users/currentuser',
      {
        headers:{
          Host: 'ticketing.dev'
        }
      }
    );

    return response.data

  }else{
    //we are on the browwer
    const response = await axios.get('/api/users/currentUser');

    return response.data;
  }
  return{};
}
```

- Remeber in our *ingress-nginx* service file, we said to point to `ticketing.dev`. 

- When the request comes from the browser, nginx understands their trying to access `ticketing.dev`. 

- But with our server request, there is nothing that tells nginx to use that domain. So we must supply it in the `header`
  
- Currently this request will give us `null` because we telling nginx to pass along cookies

#### Passing through cookies

We can get the `req` property off of `.getInitialProps`

```js
LandingPage.getInitialProps = async ({headers}) => {
  if (typeof window === 'undefined'){
    //we are on the server
    const response = await axios.get(
      'http://ingress-nginx.ingress-nginx.svc.cluster.local/api/users/currentuser',
      {
        headers: req.headers
      }
    );

    return response.data

  }else{
    ...
  }
}
```

#### Creating a Reusable API Client
<a href="https://ibb.co/XyKHWWF"><img src="https://i.ibb.co/tZvRHHY/image.png" alt="image" border="0"></a>

Create `/api/buildClient`

```js
import axios from 'axios';

export default ({req}) => {
  if(typeof window === 'undefined'){
    //we are on the server
    return axios.create({
      baseURL: 'http://ingress...',
      headers: req.headers
    })
  }else{
    return axios.create({
      baseUrl: '/'
    })
  }
}
```

Back in our landing page:
```js
import buildClient from 'api/buildClient';

LandingPage.getInitialProps = async(context) =>{
  const client = await buildCuild(context);
  const {data} = client.get('/api/usres/currentuser');
  return data;
}
```

#### Passing `GetInitialProps` to rest of components

We are going to setup `GetInitialProps` on our root `_app` and there are a couple of things we have to do that are different from **Page components**

<a href="https://ibb.co/k9KQLtt"><img src="https://i.ibb.co/jWhgCSS/image.png" alt="image" border="0"></a>

In `_app.js`
```js
const AppComponent = ({Component, pageProps}) =>{
  return(
    ...
  )
}

AppComponent.getInitialProps = async appContext => {
  const client = buildClient(appContext.ctx);
  const {data} = client.get('/api/users/currentuser');

  return data
}
...
```
- But now our `.getInitialProps` on our **page components** will not get evoked automitacally

We can get all of the `.getInitialProps` from all of our pages like so:

```js
AppComponent.getInitialProps = async appContext => {
  ...
  let pageProps = {};
  if (appContext.Component.getInitialProps){
    pageProps = await appContext.Component.getInitialProps(appContext.ctx)
  }

  return data
}
```
- `appContext` has many other properties aswell

Now to pass these props to all of our components, we would return:

```js
AppComponent.getInitialProps = async appContext => {
  ...

  return {
    pageProps,
    ..data
  }
}
```

## NPM Setup

<a href="https://ibb.co/Fz89ZJW"><img src="https://i.ibb.co/6FD29bZ/image.png" alt="image" border="0"></a>

#### Shared logic Between Services
<a href="https://ibb.co/jVcvNP8"><img src="https://i.ibb.co/mvLCjVR/image.png" alt="image" border="0"></a>

#### Options for Code sharing

1. Copy and past code
   - Won't know when files change, hard to document over time
2. Git Submodule
   - Challening commands
3. Publishing as an npm package
   - We can version the code
   - Can be tedious

#### Publishing NPM Modules

We create a director called `common` that will share code between all of our services

In our `package.json`

```json
{
  "name": "@orgnanization_name/name"
}
```

In order to publish a package, everything in tte package has to be committed with git. Create with `git init`

First make sure we're logged in: 
```bash
npm login
```

Once we commit are changes and want to publish:

```bash
npm publish --access public 
```
- need `--access public` or else npm will want to publish as `private` (costs money)

#### NPM and Typescript
<a href="https://ibb.co/cwYVgpQ"><img src="https://i.ibb.co/dtmxW3D/image.png" alt="image" border="0"></a>

```bash
tsc --init
```
Install `typescript` and `del-cli`

```bash
npm instal typescript del-cli --save-dev
```

In our package.json
```json
"scripts": {
  "clean": "del ./build/*",
  "build": "npm run clean && tsc"
}
```
- `del` ensures deleted our build folder so on `npm run build` we have the latest build

In our tsconfig.json:
```json
{
  "decleration": true,
  "outDir": "./build"
}
```

#### Publish command

In package.json:
```json
{
  "main":"./build/index.js",
  "types": "./build/index.d.ts",
  "files":[
    "./build/**/*"
  ]
}
```
- the `main` property specifies the directory of our imports
- the `types` property specifies the directory of our types imports
- the `files` property specifies which files to include in the package

We should also add a `.gitignore`:
```
build
node_modules
```

We can increment the version number in our `package.json` by updating it manually or by running:

```
npm version patch
```

Then to publish: 

```
npm run build
```
```
npm run publish
```

#### Better Importing

To simply imports for the importer, in index.ts we can add all the imports

```ts
export * from './path/file_1';
export * from './path/file_2';
...
```
- We also need to make sure that dependecies of these imports are reflected in the `package.json` dpendecies by `npm install`ing them (includes `@types`)

#### Updating

Once are packaged is updated to npm, we can update any projects using it with `npm update @organization/name`

## Ticketing Service

<a href="https://ibb.co/9ZcZtMN"><img src="https://i.ibb.co/3R4RzJ7/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/59ysBGG"><img src="https://i.ibb.co/FWM8gmm/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/5s7XPD1"><img src="https://i.ibb.co/9yMm7Xp/image.png" alt="image" border="0"></a>

#### Mongo Setup

We are connecting to a different instance of MongoDB

Let's set the URL as an environment variable

in `tickets-deply.yaml`

```yaml
...
    spec:
      containers:
        - name: tickets
          image: bryanling/tickets
          env:
            - name: MONGO_URI
              value: mongodb://tickets-mongo-srv:27017/tickets
```

In our `index.ts`, we also need to check this exists:
```ts
if (!process.env.MONGO_URI){
  throw new Error('MONGO_URI must be defeind')
}

try{
  await mongoose.connect(process.env.MONGO_URI, {...})
}
...
```

#### Writing tests for our routes

```ts
import request from 'supertest';
import {app} from '../../app';

it('has a route handler listening to /api/tickets for post requests', async ()=>{
  const respose = await request(app).post('api/tickets').send({})

  expect(response.status).not.toEqual(404)
})
```
- We can also do this for all routes first before we implement them

To test routes and look for authentication we first wire up some of the middelwares from our `auth` service so that we get the correct status code for errors (`401`)

To test that we are signed in, we can't use the `global.signin` we did before because there is no route in our **ticketing service** that requires us to pass a body

We will set cookies for testing manually:

```ts
import jwt from 'jsonwebtoken';

declare global{
  namespace NodeJS {
    interface Global {
      signin(): string[];
    }
  }
}
global.signin = () =>{
  //Build a jwt
  const payload = {
    id: 'asfeaf',
    email: 'test@test.com'
  }

  //create the JWT
  const token = jwt.sign(payload, process.env.JWT_KEY!)

  //build session Object
  const session = {jwt: token};

  //turn that session into JSONc
  const sessionJSON = JSON.stringify(session)

  //take JSON and sncode it as base64
  const base64 = Buffer.from(sessionJSON).toStrin('base64')

  //return string thats the cookie with the encoded data

  return [`express:sess=${base64}`]
}

```
- You can see `express:sess=${base64}` in inspect element when we make a request with our `auth` service
- we use the type `string[]` because of `supertest` library

Now we can use this in our test
```ts
it(`returns a status other than 401 if the user is gigned in`, async()=>{
  const respose = await request(app)
    .post('api/tickets/')
    .set('Cookie', global.signin())
    .send({})
  
  expect(response.status).not.toEqual(401);
})
```

#### Ticket Mongoose Model

<a href="https://ibb.co/q7MNspk"><img src="https://i.ibb.co/S5XtnBR/image.png" alt="image" border="0"></a>


#### Testing Document Creations

```ts
import {Ticket} from '/models/ticket';

it('creates a ticket with valid inputs', async ()=>{
  let tickets = await Ticket.find({})
  expect(tickets.length).toEqual(0)
  //we wrote a beforeEach earlier that empties the memory mongo db

  await request(app)
    .post('/api/tickets')
    .set('Cookie', global.signin())
    .send({
      title: 'abc',
      price: 20
    })
    .expect(201);

  tickets = await Ticket.find({})
  expect(tickets.length).toEqual(1);
  expect(tickets[0].price).toEqual(20);
})
```

#### Testing getting a ticket

Getting a ticket
```ts
router.get('/api/tickets/:id', async(req:Request, res:Response)=>{
  const ticket = await Ticket.findById(req.params.id);

  if(!ticket){
    //throw error
  }

  res.send(ticket)
})
```

Test:
```ts
import mongoose from 'mongoose';

it('returns a 404 fi the ticket is not found', async () =>{
  const id = new mongoose.Typs.ObjectId().toHexString();

  const res = await request(app)
    .get(`/api/tickets/${id}`)
    .send()
    .expect(404)
})
```
- We need to generate an id or else `mongo` will throw a different error

#### Updating a ticket

To test an error when the user does not own a ticket, we need to update our `global.signin()` method to generate a new id whenever it is called:
```ts
global.signin = () =>{
const payload ={
  id: new mongoose.Types.ObjectId().toHexString,
  email: 'test@test.com',
}
...
}
```

## NATS Streaming Server - An Event Bus Implementation

<a href="https://ibb.co/7bQTrpV"><img src="https://i.ibb.co/BwG7T6j/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/Z83g4bK"><img src="https://i.ibb.co/tsS2j1c/image.png" alt="image" border="0"></a>


#### Creating a NATS Streaming Deployment

```yaml
apiVersion: apps/v1
kind: Deplyment
metadata:
  name: nats-depl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nats
  template:
    metadata:
      labels:
        app: nats
    spec:
      containers:
        - name: nats
          image: nats-streaming:0.17.0
          args: [
            '-p',
            '4222',
            '-m',
            '8222',
            '-hbi',
            '5s',
            '-hbt',
            '5s',
            '-hbf',
            '2',
            '-SD',
            '-cid',
            'ticketing'
          ]
---
apiVersion: v1
kind: Service
metadata:
  selector:
    app: nats
  ports:
    - name: client
      protocol: TCP
      port: 4222
      targetPort: 4222
    - name: monitoring
      protocol: TCP
      port: 8222
      targetProt: 8222
```
- Documentation and other commands for nats-streaming found here: [nats-streaming](https://hub.docker.com/_/nats-streaming)

#### Big Notes on NATS Streaming

<a href="https://ibb.co/x5V48Hw"><img src="https://i.ibb.co/Bs81KNv/image.png" alt="image" border="0"></a><br />

<a href="https://ibb.co/hFn27KH"><img src="https://i.ibb.co/b5yNWrL/image.png" alt="image" border="0"></a><br />

<a href="https://ibb.co/q1GDZcz"><img src="https://i.ibb.co/DKX1qvN/image.png" alt="image" border="0"></a><br />

#### Building a NATS Test Project

<a href="https://ibb.co/b2FY3K3"><img src="https://i.ibb.co/WP07323/image.png" alt="image" border="0"></a>

After creating a project with `npm init -y` and `tsc --init` we need to install a couple of dependicies

```bash
npm install node-nats-streaming ts-node-dev typescript @types/node
```

Setup our files as such:
- src
  - listener.ts
  - publisher.ts

Add some scripts in `package.json`

```json
{
  "scripts":{
    "publish" : "ts-node-dev --rs --notify false src/publisher.ts",
    "listen" : "ts-node-dev --rs --notifyh false src/listener.ts"
  }
}
```
- We can use the command `rs` to restart the `ts-node-dev` instance

We are going to setup **port forwarding** for this dummy project
<a href="https://ibb.co/pdDxz0H"><img src="https://i.ibb.co/HqvVzFc/image.png" alt="image" border="0"></a>

```bash
kubectl port-forward name-of-pode 4222:4222
```
- get pods with `kubectl get pods`

<a href="https://ibb.co/p21fwpw"><img src="https://i.ibb.co/WnHk515/image.png" alt="image" border="0"></a><br />
- `ticket:created` is the channel, name of the channel is called the **subject**
- `subscription` listens for the channel type and the data


In `publisher.ts`
```ts
import nats from 'node-nats-streaming';

console.clear()
//stan is really  client
const stan = nats.connect('ticketing', 'abc', {
  url: 'http://localhost:4222'
});

stan.on('connect', () => {
  console.log('Publisher connected to NATS')

  const data = JSON.stringify({
    id: '123',
    title: 'cencert',
    price: 20
  });

  stan.publish('ticket:created', data, ()=>{
    console.log('Event published')
  })
})
```
- In documentation, you will see `client` as `stan`, `stan` is `nats` backwards
- With `NATS` we can only share raw data strings, so we have to convert it to **JSON**
- `abc` is the clientID (more on this later)

In `listener.ts`

```ts
import nats, {Message} from 'node-nats-streaming';

console.clear()

const stan = nats.connect('ticketing', '123', {
  url: 'http://localhost:4222'
})

stan.on('connect', () =>{
  console.log('Listener connected to NATS')

  const substriction = stan.subscribe('ticket:created');

  substriction.on('message', (msg: Message) => {
    const data = msg.getData();

    if(typeof data === 'string') {
      console.log(`Received event #${msg.getSequence()}, with data:${JSON.parse(data)}`)
    }
  })
})
```
- `123` is the clientID (more on this later)


#### Client ID Generation

If we tried to start another instance of `listener.ts`, We would get the error: **Error: Client ID already registered**

<a href="https://ibb.co/LN6Lpjj"><img src="https://i.ibb.co/GHkKPYY/image.png" alt="image" border="0"></a>

Usually we can take care of this easily in Kubernetes, but for this case we would generate a random id

Currently, if we were to generate 2 instances of `listener.ts`, they would both console log when we emit one event

If we emit an event, it should only go to one of them

We can solve this in  `node-nats-streaming` with **Queue Groups**

<a href="https://ibb.co/r67vG8S"><img src="https://i.ibb.co/fp1QY65/image.png" alt="image" border="0"></a><br />

Back in `listener.ts`

```ts
stan.on('connect', ()=>{
  ...
  const subscription = stan.subscribe('ticket:created', 'queue-group-name')
})
```

#### More Subscribe Options

By default, if an event results in an error, NATS will throw it out. We can set options to avoid this

```ts
stan.on('connect', ()=>{
  ...
  const options = stan
    .subscriptionOptions()
    .setManualAckMode(true);
  const subscription = stan.subscribe('ticket:created', 'queue-group-name')
})
```
- `Ack` is acknowledgement
- We need to manully acknowledge this event. Right now, the event will try to send the event again after a 30 seconds because it thinks it failed

To Acknowledge:
```ts
subscription.on('message', (msg: Message) => {
  ...
  msg.ack();
})
```

#### Client Health Checks

In our NATS deployment service we added:
```yaml
 - name: monitoring
    protocol: TCP
    port: 8222
    targetProt: 8222
```

Let's forward this port aswell

```bash
kubectl port-forward pod-name 8222:8222
```

Now if we go in our browser to `localhost:8222/streaming` in our browser, we should see:

<a href="https://imgbb.com/"><img src="https://i.ibb.co/7vy0QSL/image.png" alt="image" border="0"></a>

If we type in `localhost:8222/streaming/channelsz?subs=1`, We can see a list of our subscriptions

When we restart one of our subscriptions with `rs`, we will temporarily see 3 subscriptions instead of 2

NATS thinks it may have just disconnected temporarily, but we don't want this behaviour

Inside `listener.ts`
```ts
sta.on('connect', ()=>{
  ...
  stan.on('close', ()=>{
    process.exit()
  })
})
...
process.on('SIGINT', () => stan.close())
process.on('SIGTERM', () => stan.close())
```
- `SIGINT` and `SIGTERM` are termination signals

#### Core Concurrency Issues

<a href="https://ibb.co/QFhqzJ9"><img src="https://i.ibb.co/pvNsG12/image.png" alt="image" border="0"></a>

There are infinant number of ways this system can fail: 
- Listener fails to process the event
- One listener runs more quickly than another
- NATS might think a client is still alive when it is dead
- We might recieve/process the same event more than once if one is still processing

Common questions
<a href="https://ibb.co/0J02NXp"><img src="https://i.ibb.co/mznTsBr/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/CJztxgf"><img src="https://i.ibb.co/4s7fBzH/image.png" alt="image" border="0"></a>

#### Solving Concurrency Issues

<a href="https://ibb.co/0hWLkXd"><img src="https://i.ibb.co/f4KWbSP/image.png" alt="image" border="0"></a>
<br/>
<a href="https://ibb.co/2vD7Xrp"><img src="https://i.ibb.co/K2MjCg8/image.png" alt="image" border="0"></a>
<br/>
<a href="https://ibb.co/v3dTBKG"><img src="https://i.ibb.co/k4DwyWf/image.png" alt="image" border="0"></a>
<br />
<a href="https://ibb.co/2NP1R0V"><img src="https://i.ibb.co/PQ4vJsX/image.png" alt="image" border="0"></a>
<br />
<a href="https://ibb.co/5vPTpBT"><img src="https://i.ibb.co/d6RBCGB/image.png" alt="image" border="0"></a>
<br/>
<a href="https://ibb.co/gPGwpSh"><img src="https://i.ibb.co/hZkHjBw/image.png" alt="image" border="0"></a>
- We can use `number` to make sure that transactions are done in order

#### Event Redelivery

<a href="https://ibb.co/6nQHXCC"><img src="https://i.ibb.co/0D5Csbb/image.png" alt="image" border="0"></a>

Back in `listener.ts`
```ts
const options = stan
  .subscriptionOptions()
  .setManualAckMode(true)
  .setDeliverAllAvailable()
  .setDurableName('nname')
```
- we ideally don't use `setDeliverAllAvailable` on its own because it will get all the events ever created
- but this will run when the service is first deployed,

<a href="https://ibb.co/kcGp8gD"><img src="https://i.ibb.co/VBLdSWC/image.png" alt="image" border="0"></a>

We need need to make sure our subscript has a queu-group-name otherwise the durable subscription will not persist (be dumped once our service goes down)

```ts
...
const subscription = stan.subscribe(
  'ticket:created',
  'queue-group-name',
  options
)
```

#### NATS listeners as a class

<a href="https://ibb.co/JsSrtVP"><img src="https://i.ibb.co/qr2mjKw/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/bF1x3PB"><img src="https://i.ibb.co/vh1bBHP/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/Kyh215c"><img src="https://i.ibb.co/SvytL6D/image.png" alt="image" border="0"></a>

```ts
import { Stan } from 'node-nats-streaming';

abstract class Listener{
  abstract subject: string;
  abstract queueGroupName: string;
  abstract onMessage(data: any, msg: Message): void;
  private client: Stan;
  protected ackWait = 5 * 1000;

  constructor(client: Stan){
    this.client = client;
  }

  subsctiptionOptions(){
    return this.client
      .subscriptionOptions()
      .setDeliverAllAvailable()
      .setManualAckMode(true)
      .setAckWait(this.ackWait)
      .setDurableName(this.queueGroupName)
  }

  listen() {
    const subscription = this.client.subscribe(
      this.subject,
      this.queueGroupName,
      this.subscriptionOptions()
    );

    subscription.on('message', (msg: Message) => {
      console.log(
        `Message received: ${this.subject} / ${this.queueGroupName}`
      )

      const parsedData = this.parseMessage(msg);
      this.onMessage(parsedData, msg)
    })
  }

  parseMessage(msg: Message) {
    const data = msg.getData();
    return typeof data === 'string'
      ? JSON.parse(data)
      : JSON.parse(data.toString('utf8'))
  }
}
```

#### Extending the Listener

Let's create the Class **TicketCreatedListener**

```ts
class TicketCreatedListener extends Listener {
  subject = 'ticket:created';
  queueGroupName = 'payments-service';
  onMessage(data: any, msg: Message){
    console.log('Event data!', data);

    msg.ack();
  }
}
```

Now we can use this

```ts
stan.on('connect', ()=>{
  console.log('listener connectd to NATS');

  stan.on('close', ()=>{
    console.log('NATS connection closed');
    process.exit();
  })

  new TicketCreatedListener(stan).listen();
})

```
We should also add correct typecript types to help

#### Overview
<a href="https://ibb.co/g6bLkp3"><img src="https://i.ibb.co/2501GQy/image.png" alt="image" border="0"></a>

#### Custom Publisher

We are going to create `base-publichser.ts`
```ts
import {Stan} from 'node-nats-streaming';
import {Subjects} from './subjects';

interface Event{
  subject: Subjects;
  data: any;
}

export abstract class Publisher<T extends Event>{
  abstract subject: T['subject'];
  private client: Stan;

  constructor(client: Stan){
    this.client = client;
  }

  publish(data: T['data']):Promise<void> {
    return new Promise((resolve, reject)=>{
        this.client.publish(this.subject, JSON.stringify(data), (err)=>{
        if (err) {
          return reject(err);
        }
        resolve();
      })
    })
  }
}
```

Now create `ticket-created-publisher.ts`

```ts
import {Publisher} from './base-publisher';
import {TicketCreatedEvent} from './ticket-created-event';
import {Subjects} from './subjects';

export class TicketCreatedPublisher extends Publisher<TicketCreatedEvent>{
  subject: Subjects.TicketCreated = Subjects.TicketCreated;
}
```
Now in `publisher.ts`

```ts
stan.on('connect', async ()=>{
  console.log('Publisher connected to NATS');

  const publisher = new TicketCreatePublisher(stan);
  await publisher.publish({
    id: '123',
    title: 'concert',
    price: 20
  })
})
```

#### Event Definitions Summary 

<a href="https://ibb.co/WWNmWBr"><img src="https://i.ibb.co/xSKxSzn/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/Dr2M3LB"><img src="https://i.ibb.co/cYn8zxW/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/XCJTdjP"><img src="https://i.ibb.co/KryHQbg/image.png" alt="image" border="0"></a>


#### Publishing Ticket Creation Event

Inside out `tickets/events` service directory, we will create `ticket-created-publisher.ts`

```ts
import { Publisher, Subjects, TicketCreatedEvent } from '@bltechTickets/common';;

export class TicketCreatedPublisher extends Publisher<TicketCreatdEvent>{
  subject: Subjects.TicketCreated = Subjects.TicketCreated;
}
```

Now in our routes directory

```ts
import { TicketCreatedPublisher } from '../events/ticket-created-publisher';

...
await.ticket.save();
new TicketCreatedPublisher(client).publish({
  id: ticket.id,
  title: ticket.title,
  price: ticket.price,
  userId: ticket.userId
});
```
- We should pull off `title` and `price` from `await.ticket.save()` intstead of `req.body` becuase our mongoose might sanatize the data in different ways
- We need to add nats `client`in our ticketing service

#### Adding NATS client to our ticketing service

<a href="https://ibb.co/ZWWsQRN"><img src="https://i.ibb.co/LnnwTDv/image.png" alt="image" border="0"></a>

We need to share our NATS client, but want to avoid creating circular dependecies

We will do this:

<a href="https://ibb.co/vhCfDPn"><img src="https://i.ibb.co/zFWCNRv/image.png" alt="image" border="0"></a>

When we used mongoose:
```ts
try{
  await mongoose.connect(...)
}
catch(err){
  ...
}
```
we were able to then access this mongoose connection globablly in our `ticketing` service

We want to do something similar with our NATS client

<a href="https://ibb.co/sC6YfXb"><img src="https://i.ibb.co/0KFb7zm/image.png" alt="image" border="0"></a>

We create one instance and export it, and that instance is shared throughout our project

<a href="https://ibb.co/Brnpj6h"><img src="https://i.ibb.co/z76Mb23/image.png" alt="image" border="0"></a>

We will create this in `ticketing/nats-wrapper.ts`

```ts
import nats, { Stan } from 'node-nats-streaming';

class NatsWrapper{
  ...
}

export const natsWrapper = new NatsWrapper();
```

#### Creating NatsWrapper Class

```ts
class NatsWrapper{
  private _client?: Stan;

  get client(){
    if(!this._client){
      throw new Error('Cannot access NATS client before connecting')
    }

    return this._client;
  }

  connect(clusterId: string, clientId: string, url: string){
    this._client = nats.connect(clusterId, clientId, {url});

    return new Promise((resolve, reject) => {
      this._client.on('connect', () =>{
        console.log('Connected to NATS')
        resolve();
      })

      this._client.on('error', (err) => {
        reject(err);
      })
    })
    
  }
}

export const natsWrapper = new NatsWrapper();
```

Now we can use this similar to `mongoose.connect`

In `ticketing/index.ts`

```ts
try{
  await natsWrapper.connect('ticketing', 'abc123', 'http://nats_srv:4222')
  await mongoose.connect(...)
}
```
-`http://nats_srv:4222` comes from our kubernetes pod
- ticketing is the id we specified in `.yaml` file

And we can add it in `routes/new.ts`
```ts
  import { natsWrapper } from '../nats-wrapper';

  ...
  await new TicketCreatedPublisher(natsWrapper.client)...
```

#### Graceful shutdown

Inside `index.ts`
```ts
...
try{
  await natsWrapper.connect('ticketing', 'alsdkj', 'http://nats-srv:4222');
  natsWrapper.client.on('close', () => {
    console.log('NATS connection closed!');
    process.exit();
  });
  process.on('SIGINT', () => natsWrapper.client.close())
  process.on('SIGTERM', () => natsWrapper.client.close())
}
/..
```
- We don't want to do this in our `NatsWrapper` class because it is bad design to have `process.exit()` defined in our shared classes
- We should now restart `skaffold`

#### Failed Event Publishing

<a href="https://ibb.co/2j1JLQc"><img src="https://i.ibb.co/P5vkbn1/image.png" alt="image" border="0"></a>

<a href="https://ibb.co/9spyBjH"><img src="https://i.ibb.co/jg83qFk/image.png" alt="image" border="0"></a>

Turns out `await` isn't relevant to the issue with our current implmentation with our `publishers`

<a href="https://ibb.co/kQfMnRB"><img src="https://i.ibb.co/vZGD57q/image.png" alt="image" border="0"></a>

- If our `transaction event` fails, then the `Transaction Database` and `Accounts Database` would be out of sync

To prevent this, we should **save the event into our database** and include a "sent flag"
<a href="https://ibb.co/MGxwLJ0"><img src="https://i.ibb.co/Sd4YpZC/image.png" alt="image" border="0"></a>

We need to also make sure that saving a `Transaction` and `Event` both succeed

<a href="https://ibb.co/5973Jr5"><img src="https://i.ibb.co/qnZ82NR/image.png" alt="image" border="0"></a>

MongoDB has a built in function called a `database transaction`. Basically says **Do these actions, and if any of them fail, reverse any actions**

We won't do this for this course

