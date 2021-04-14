# Javascript

## Table of Contents
- [Javascript](#javascript)
  - [Table of Contents](#table-of-contents)
  - [Handling different types of errors on the Frontend](#handling-different-types-of-errors-on-the-frontend)
## Handling different types of errors on the Frontend

[Source 2021/04/13](https://stackoverflow.com/questions/49967779/axios-handling-errors)
```js
axios.get('/api/xyz/abcd')
.catch((error) => {
        if (error.response) {
          // Request made and server responded
          console.log(error.response.data);
          console.log(error.response.status);
          console.log(error.response.headers);
        } else if (error.request) {
          // The request was made but no response was received
          console.log(error.request);
        } else {
          // Something happened in setting up the request that triggered an Error
          console.log('Error', error.message);
        }
      });
```