# NextJS

[Course](https://www.udemy.com/course/server-side-react-apps-with-next-js-2021-edition/learn/lecture/26991080?start=0#overview)

## Table of Contents
- [NextJS](#nextjs)
  - [Table of Contents](#table-of-contents)
  - [Install](#install)
  - [Routing](#routing)
    - [Dynamic Routes](#dynamic-routes)
    - [Catch All Routes](#catch-all-routes)
    - [Navigation](#navigation)
      - [Redirect via function](#redirect-via-function)
      - [Components](#components)
  - [Styles](#styles)
    - [Modules](#modules)
    - [Applying styles](#applying-styles)
    - [File Convention](#file-convention)
  - [Metatags](#metatags)
  - [_document.js](#_documentjs)
  - [Images and Static Files](#images-and-static-files)
  - [Prerendering](#prerendering)
    - [Static Generation](#static-generation)
    - [Types of files](#types-of-files)
      - [SSG](#ssg)
      - [ISR](#isr)
      - [`getStaticPaths`](#getstaticpaths)
      - [`getServerSideProps`](#getserversideprops)
    - [Client Side Fetching](#client-side-fetching)
  - [APIs](#apis)
    - [Get requests](#get-requests)
    - [Post requests](#post-requests)
    - [Dynamic API routes](#dynamic-api-routes)
    - [Connecting to Mongoose](#connecting-to-mongoose)
      - [Setup](#setup)
      - [Issue with models](#issue-with-models)
    - [Redux Work Arounds](#redux-work-arounds)
  - [Deploying](#deploying)
    - [Checking for optimizations](#checking-for-optimizations)
    - [Config file](#config-file)
    - [Building](#building)
    - [Deploying to Vercel](#deploying-to-vercel)
      - [Making Changes](#making-changes)

## Install
```bash
npx create-next-app .
```
- `--typescript`
- `/pages`, `/public`, and `/styles` directory are reserved directory names
## Routing
Each file is a route
<img src="https://i.ibb.co/Csp9fVK/image.png" alt="image" border="0"></img>

### Dynamic Routes
Create the filename as `[id].js`

Use the `useRouter` hook
```jsx
import { useRouter } from 'next/router';
const UsersById = () =>{

  const router = useRouter();

  return(
    <>
      <h1>{router.query.id}</h1>
    </>
  )
}

export default UsersById;
```
- Can only have one for each directory (Next won't know which on to use otherwise)
- For additional dynamic values, we can add `[]` to the folder names

### Catch All Routes
Sometimes long dynamic routes can be annoying

Instead, we can use `[...ids].js`
- This captures all the params leading up to it.
- `useRouter().query` Now returns an array of strings of the query params.
- Use `[[...ids]].js` if the query can be optionaly empty

### Navigation

Similar to `react-router`

```jsx
import Link from 'next/link';

const Home = () => {
  return(
    <>
      <Link href="/users">Users</Link>
    </>
  )
}
```
- To apply styles we can wrap the contents of `Link` with an `<a>` with now `href` and add classes

Another option for Link
```jsx
<Link 
  href={{
    pathname:'/wheels/[id]/[color]/[type]',
    query:{id:1, color: 'red', type:'round'}
  }}
>
  Go
</Link>

```
#### Redirect via function
```jsx
import {useRouter} from 'next/router'

const Page = () =>{
  const router = useRouter();

  const redirect = () =>{
    router.push('/')
    //or
    router.push({
      pathname:'/wheels/[id]/[color]/[type]',
      query:{id:1, color: 'red', type:'round'}
    })
  }
}
```

#### Components
Put all components that are not routes in another folder called `/components` otherwise they will be included as a route


## Styles
Inside `/styles`

### Modules
For files of the form `Home.module.css` they belong to a single page

Importing:
```jsx
import style from '../../styles/test.module.css'
```
- Non `.module.css` files are appled to the entire application
- We can only write classes in `.module.css` files

### Applying styles
`test.module.css`
```css
.red {
  color: red;
}
```
`index.js`
```jsx
import style from '../../styles/test.module.css'

const Main = () =>{
  <h1 className={style.red}>Hello world</h1>
}
```
### File Convention
Sometimes we move the `.module.css` out of `/styles` and place them directly with their associated files. This is because there can be many `.module.css` files in the final application.

## Metatags
```jsx
import Head from 'next/head';

const Page = () =>{
  return(
    <>
      <Head>
        <title>Title</title>
        <meta name="description" content="Some description"/>
        ...
      </Head>
    </>
  );
}
```
- Works with dynamic content
  - Changing the title for example
- Include global metatags by adding them to `_app.js`
- If metatags are set on the page, they overwrite the previous route's metatags
  
## _document.js
There is no html file by default.

We can modify the default html by adding a `_document.js` file in pages

```jsx
import Document, {Html, Head, Main, NextScript} from 'next/document';

class MyDocument extends Document{
  render(){
    return(
      <Html lang="eng">
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    )
  }
}

export default MyDocument
```
## Images and Static Files
`/public`

```jsx
import Image from 'next/image';

<Image
  src="/images/1.png"
  width={300}
  height={300}
/>
```
- Optimizes the image size instead of using standard <img />
- Different properties can be found in Next.js documentation

## Prerendering
<img src="https://i.ibb.co/Vw06VM0/image.png" alt="image" border="0"/>

### Static Generation
<img src="https://i.ibb.co/68xMwYg/image.png" alt="image" border="0"/>
<img src="https://i.ibb.co/1Tsbbwd/image.png" alt="image" border="0"/>
Default bahaviour of next is to pre-build

### Types of files
Can be seen after running `npm run build`
<img src="https://i.ibb.co/725m0LP/image.png" alt="image" border="0">

#### SSG
```jsx
const HomeComponent = (props) =>{
  ...
}
export async function getStaticProps(context){
  //make async requests
  if(some error){
    return {
      notFound: true, //or
      redirect: {
        destination: '/somewhere_else'
      }
    }
  }
  // make more async requests
  // ...
  return{
    props:{
      name:"MyName",
      ...
    }
  }
}
```
- fires before pre-rendering
- passed as props to main component
- should always return an object;
- We can get more info to use throught `context` such as the query params, etc.
- Error handling by returning `notFound`
  - Redirects to 404
  - or we can use `redirect`
#### ISR
If the information changes, we won't see those changes unless we run `npm run build` again
with SSG (one single time).

In `npm run dev`, the sever rebuilds everytime we refresh the application.

We need to use the revalidate option. This will check after a certain time and rebuild that
particular file.

```jsx
export async function getStaticProps(){
  ...
  return {
    props:{...},
    revalidate: 60
  }
}
```
- \# is amount of seconds

#### `getStaticPaths`

We need to use `getStaticPaths` if we want to use `getStaticProps` on dynamic pages (i.e. `[id].js`)

```jsx
const Page = (props) =>{
  ...
}

export async function getStaticProps(context){

  const {params} = context;
  const request = await axios.get(`.../${params.id}`);
  return{
    props:{
      hello:"yes"
    }
  }
}

export async function getStaticPaths(){
  return{
    paths:[
      { params:{id: '1'}},
      { params:{id: '2'}},
      { params:{id: '3'}}
    ],
    fallback: false
  }
}
```
- `getStaticPaths` needs to be told what to generate (static generation)
- Anything that is not in the list will `fallback: false` and fail to a 404
  - If we set `fallback:true`, we get a server error if we try to access any of the props
- If we build, we will see 
  - <img src="https://i.ibb.co/H4QWdGy/image.png" alt="image" border="0">
  - Should only be used if we know what the params are
- If we provided a link to `id:4` it would work from another page because it has time to pregenerate. But refreshing the page will cause the error

If we set `fallback: true`, we can do:
```jsx
import {useRouter} from 'next/router'

const Page = () =>{
  const {isFallback} = useRouter();
  return(
    <>
      {
        isFallback?'oops':<h1>{props.user.name}</h1>
      }
    </>
  )
}
```
- Otherwise, we may get an error when we try to `npm run build`

#### `getServerSideProps`

Access to more properties than if it were static.

Fires upon request.

Shouldn't use unless we need it.

In some file `[id].js`

```jsx
export const getServerSideProps = async(context) =>{
  //similar to express
  console.log(context.params);
  console.log(context.req);
  console.log(context.res);
  return{
    props:{
      id: context.params
    }
  }
}
```
- Now we can access any of the props in our main component that require `params`
  - We won't get an error like in `static` because we don't need to know the paths beforehand

### Client Side Fetching
If we create an app that gets post, we could use `getStaticProps` to get the first 10 posts, and get more using regular react with `useEffect` and `useState`.

We would initiliaze the state with the data fetched from `getStaticProps`.

This is good for SEO.

## APIs

Created in `/api` 

Requests are made to `/api/filename`

### Get requests

Similar to express:

```js
const handler = async (req, res) => {
  if(req.method === 'GET'){
    res.status(200).json({...})
  }
}

export default handler;
```
- cannot be used in `getStaticProps` because that's already on the server

### Post requests

```js
const handler = async (req, res) =>{
  if(req.method === 'POST'){
    const {title, body} = req.body;

    try{
      const request = await axios.post("...", {title, body})
      res.status(200).json({
        data: request.data
      })
    }catch(err){
      res.status(401).json({
        message:"bad"
      })
    }
  }
}
```

### Dynamic API routes

Same notation as pages with `[id].js`

```js
import axios from 'axios';

const handler = async(req, res)=>{
  const id = req.query.id;
  ...
}
```

### Connecting to Mongoose

#### Setup

`npm install mongoose`

Setup in `/helpers/database/mongoose_db.js`

```jsx
import mongoose from 'mongoose';

export async function connectToDb(){
  if(mongoose.connection.readyState >= 1) return; //already have a connection

  return mongoose.connect("...", {
    useNewUrlParser: true,
    useUnifiedTopolofy: true,
    useFindAndModify: true
  })
}
}
```

#### Issue with models

We will run into an error when using models saying `Cannot overwrite 'Name' model once compiled`

This occurs because of NextJS's hot-reloading.

Fix:
```js
const Post = mongoose.models.Post || mongoose.model('Post', postsSchema);
export default Post;
```
- A new model won't be created if it already exists
- Some libraries won't have this work around as nextJS works differently

### Redux Work Arounds

We need to install `next-redux-wrapper`

```jsx
import {Provider} from 'react-redux'
import {createWrapper} from 'next-redux-wrapper'
import store from '../store'

function myApp({Component, pageProps}){
  return(
    <Provider store={store}>
      <Component {...pageProps} />
    </Provider>
  )
}

const makeStore = () => store;
const wrapper = createWrapper(makeStore);

export default wrapper.withRedux(myApp);
```

## Deploying
<img src="https://i.ibb.co/r4qjV9w/image.png" alt="image" border="0">
- Not gonna get the stuff in red

<img src="https://i.ibb.co/w4x1vf6/image.png" alt="image" border="0">

### Checking for optimizations
- Make sure to add headers and metatags
- No Anchor tags, use `Link`

### Config file

Create `next.config.js`

```js
const {PHASE_DEVELOPMENT_SERVER} = require('next/constants');
module.exports = (phase, { defaultConfig }) => {
  if(phase === PHASE_DEVELOPMENT_SERVER){
    return {
      env:{
       ...
      }
    }
  }
}
```
- `env` are environment variables
  - Access via `process.env.name`
- `PHASE_DEVELOPMENT_SERVER` is a constant that checks for dev environment
  - `npm run dev`

### Building
We can add a scriptin `package.json`

```json
"scripts":{
  ...
  "export": "next export"
}
```

### Deploying to Vercel

Works with repositories [vercel.com](vercel.com)

1. `git init`
2. Put the repository on github
3. Login to [vercel.com](vercel.com) via Github
4. Import the Git Repository into Vercel
5. Set settings like environment variables etc.
6. Deploy

#### Making Changes

Whenever master is changed, it will update it