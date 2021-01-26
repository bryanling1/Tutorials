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