# Request id

A good thing to know is that every request passed through this interceptor, has an id.
**This does not mean that is a unique id**. The id is used in a number of ways, but the
most important is to bind a request to its cache.

The id generation is good enough to generate the same id for theoretically sames requests.
A simple example is a request with `{ baseURL: 'https://a.com/', url: '/b' }` results to
the same id with `{ url: 'https://a.com/b/' }`.

Also, a custom id can be used to treat two requests as the same.

```js #runkit
const axios = require('axios');
const { setupCache } = require('axios-cache-interceptor');

// Global
setupCache(axios);

const { id } = await axios.get('https://jsonplaceholder.typicode.com/posts/1', {
  baseURL: 'baseURL',
  query: { name: 'value' }
});

// Generated
console.log('Id 1: ' + id);
console.log('Cache 1:', await axios.storage.get(id));

const { id: id2 } = await axios.get('https://jsonplaceholder.typicode.com/posts/1', {
  id: 'my-overridden-id'
});

// Specified
console.log('Id 2: ' + id2);
console.log('Cache 2:', await axios.storage.get(id2));
```

The
[default](https://github.com/arthurfiorette/axios-cache-interceptor/blob/main/src/util/key-generator.ts)
id generation can clarify this idea.

## Joining requests

Two requests will be treated as the same if they have the same `id` property. If it was
not defined, it will be auto generated by the provided `Key Generator`.

You can override their `id` property by providing a custom one.

```js
const response = await axios.get('url', {
  id: 'my overridden id'
});

// Response.id always returns the resolved it for the specific request. (In this case, 'my overridden id')
response.id === 'my overridden id';
```

## Custom generator

Everything that is used to treat two requests as same or not, is done by the `generateKey`
property.

The value is hashed into a signed integer when the returned value from the provided
generator is not a `string` or a `number`.

By default, it uses the `method`, `baseURL`, `params`, `data` and `url` properties from
the request object into an hash code generated by the
[`object-code`](https://www.npmjs.com/package/object-code) library.

If you need to change or share the cache for two requests based on other properties, like
headers and etc, you can create your own Key Generator:

```js
const generator = buildKeyGenerator(({ headers = {} }) => {
  // In this imaginary example, two requests will
  // be treated as the same if their x-cache-server header is the same.
  return headers['x-cache-server'] || 'not-set';
});

const axios = mockAxios({
  generateKey: keyGenerator
});
```
