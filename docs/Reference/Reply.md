<h1 align="center">Fastify</h1>

## Reply
- [Reply](#reply)
  - [Introduction](#introduction)
  - [.code(statusCode)](#codestatuscode)
  - [.statusCode](#statuscode)
  - [.server](#server)
  - [.header(key, value)](#headerkey-value)
      - [set-cookie](#set-cookie)
  - [.headers(object)](#headersobject)
  - [.getHeader(key)](#getheaderkey)
  - [.getHeaders()](#getheaders)
  - [.removeHeader(key)](#removeheaderkey)
  - [.hasHeader(key)](#hasheaderkey)
  - [.redirect([code,] dest)](#redirectcode--dest)
  - [.callNotFound()](#callnotfound)
  - [.getResponseTime()](#getresponsetime)
  - [.type(contentType)](#typecontenttype)
  - [.serializer(func)](#serializerfunc)
  - [.raw](#raw)
  - [.sent](#sent)
  - [.hijack()](#hijack)
  - [.send(data)](#senddata)
    - [Objects](#objects)
    - [Strings](#strings)
    - [Streams](#streams)
    - [Buffers](#buffers)
    - [Errors](#errors)
    - [Type of the final payload](#type-of-the-final-payload)
    - [Async-Await and Promises](#async-await-and-promises)
  - [.then(fulfilled, rejected)](#thenfulfilled-rejected)

### Introduction
<a id="introduction"></a>

The second parameter of the handler function is `Reply`. Reply is a core Fastify
object that exposes the following functions and properties:

- `.code(statusCode)` - Sets the status code.
- `.status(statusCode)` - An alias for `.code(statusCode)`.
- `.statusCode` - Read and set the HTTP status code.
- `.server` - A reference to the fastify instance object.
- `.header(name, value)` - Sets a response header.
- `.headers(object)` - Sets all the keys of the object as response headers.
- `.getHeader(name)` - Retrieve value of already set header.
- `.getHeaders()` - Gets a shallow copy of all current response headers.
- `.removeHeader(key)` - Remove the value of a previously set header.
- `.hasHeader(name)` - Determine if a header has been set.
- `.type(value)` - Sets the header `Content-Type`.
- `.redirect([code,] dest)` - Redirect to the specified url, the status code is
  optional (default to `302`).
- `.callNotFound()` - Invokes the custom not found handler.
- `.serialize(payload)` - Serializes the specified payload using the default
  JSON serializer or using the custom serializer (if one is set) and returns the
  serialized payload.
- `.serializer(function)` - Sets a custom serializer for the payload.
- `.send(payload)` - Sends the payload to the user, could be a plain text, a
  buffer, JSON, stream, or an Error object.
- `.sent` - A boolean value that you can use if you need to know if `send` has
  already been called.
- `.hijack()` - interrupt the normal request lifecycle.
- `.raw` - The
  [`http.ServerResponse`](https://nodejs.org/dist/latest-v14.x/docs/api/http.html#http_class_http_serverresponse)
  from Node core.
- `.log` - The logger instance of the incoming request.
- `.request` - The incoming request.
- `.context` - Access the [Request's context](./Request.md) property.

```js
fastify.get('/', options, function (request, reply) {
  // Your code
  reply
    .code(200)
    .header('Content-Type', 'application/json; charset=utf-8')
    .send({ hello: 'world' })
})
```

Additionally, `Reply` provides access to the context of the request:

```js
fastify.get('/', {config: {foo: 'bar'}}, function (request, reply) {
  reply.send('handler config.foo = ' + reply.context.config.foo)
})
```

### .code(statusCode)
<a id="code"></a>

If not set via `reply.code`, the resulting `statusCode` will be `200`.

### .statusCode
<a id="statusCode"></a>

This property reads and sets the HTTP status code. It is an alias for
`reply.code()` when used as a setter.
```js
if (reply.statusCode >= 299) {
  reply.statusCode = 500
}
```

### .server
<a id="server"></a>

The Fastify server instance, scoped to the current [encapsulation
context](./Encapsulation.md).

```js
fastify.decorate('util', function util () {
  return 'foo'
})

fastify.get('/', async function (req, rep) {
  return rep.server.util() // foo
})
```

### .header(key, value)
<a id="header"></a>

Sets a response header. If the value is omitted or undefined, it is coerced to
`''`.

> Note: the header's value must be properly encoded using [`encodeURI`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURI) or similar modules such as [`encodeurl`](https://www.npmjs.com/package/encodeurl). Invalid characters will result in a 500 `TypeError` response.

For more information, see
[`http.ServerResponse#setHeader`](https://nodejs.org/dist/latest-v14.x/docs/api/http.html#http_response_setheader_name_value).

- ### set-cookie
  <a id="set-cookie"></a>

    - When sending different values as a cookie with `set-cookie` as the key,
      every value will be sent as a cookie instead of replacing the previous
      value.

    ```js
    reply.header('set-cookie', 'foo');
    reply.header('set-cookie', 'bar');
    ```
  - The browser will only consider the latest reference of a key for the
    `set-cookie` header. This is done to avoid parsing the `set-cookie` header
    when added to a reply and speeds up the serialization of the reply.

  - To reset the `set-cookie` header, you need to make an explicit call to
    `reply.removeHeader('set-cookie')`, read more about `.removeHeader(key)`
    [here](#removeheaderkey).



### .headers(object)
<a id="headers"></a>

Sets all the keys of the object as response headers.
[`.header`](#headerkey-value) will be called under the hood.
```js
reply.headers({
  'x-foo': 'foo',
  'x-bar': 'bar'
})
```

### .getHeader(key)
<a id="getHeader"></a>

Retrieves the value of a previously set header.
```js
reply.header('x-foo', 'foo') // setHeader: key, value
reply.getHeader('x-foo') // 'foo'
```

### .getHeaders()
<a id="getHeaders"></a>

Gets a shallow copy of all current response headers, including those set via the
raw `http.ServerResponse`. Note that headers set via Fastify take precedence
over those set via `http.ServerResponse`.

```js
reply.header('x-foo', 'foo')
reply.header('x-bar', 'bar')
reply.raw.setHeader('x-foo', 'foo2')
reply.getHeaders() // { 'x-foo': 'foo', 'x-bar': 'bar' }
```

### .removeHeader(key)
<a id="getHeader"></a>

Remove the value of a previously set header.
```js
reply.header('x-foo', 'foo')
reply.removeHeader('x-foo')
reply.getHeader('x-foo') // undefined
```

### .hasHeader(key)
<a id="hasHeader"></a>

Returns a boolean indicating if the specified header has been set.

### .redirect([code ,] dest)
<a id="redirect"></a>

Redirects a request to the specified URL, the status code is optional, default
to `302` (if status code is not already set by calling `code`).

> Note: the input URL must be properly encoded using [`encodeURI`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURI) or similar modules such as [`encodeurl`](https://www.npmjs.com/package/encodeurl). Invalid URLs will result in a 500 `TypeError` response.

Example (no `reply.code()` call) sets status code to `302` and redirects to
`/home`
```js
reply.redirect('/home')
```

Example (no `reply.code()` call) sets status code to `303` and redirects to
`/home`
```js
reply.redirect(303, '/home')
```

Example (`reply.code()` call) sets status code to `303` and redirects to `/home`
```js
reply.code(303).redirect('/home')
```

Example (`reply.code()` call) sets status code to `302` and redirects to `/home`
```js
reply.code(303).redirect(302, '/home')
```

### .callNotFound()
<a id="call-not-found"></a>

Invokes the custom not found handler. Note that it will only call `preHandler`
hook specified in [`setNotFoundHandler`](./Server.md#set-not-found-handler).

```js
reply.callNotFound()
```

### .getResponseTime()
<a id="getResponseTime"></a>

Invokes the custom response time getter to calculate the amount of time passed
since the request was started.

Note that unless this function is called in the [`onResponse`
hook](./Hooks.md#onresponse) it will always return `0`.

```js
const milliseconds = reply.getResponseTime()
```

### .type(contentType)
<a id="type"></a>

Sets the content type for the response. This is a shortcut for
`reply.header('Content-Type', 'the/type')`.

```js
reply.type('text/html')
```
If the `Content-Type` has a JSON subtype, and the charset parameter is not set, `utf-8` will be used as the charset by default.

### .serializer(func)
<a id="serializer"></a>

`.send()` will by default JSON-serialize any value that is not one of: `Buffer`,
`stream`, `string`, `undefined`, `Error`. If you need to replace the default
serializer with a custom serializer for a particular request, you can do so with
the `.serializer()` utility. Be aware that if you are using a custom serializer,
you must set a custom `'Content-Type'` header.

```js
reply
  .header('Content-Type', 'application/x-protobuf')
  .serializer(protoBuf.serialize)
```

Note that you don't need to use this utility inside a `handler` because Buffers,
streams, and strings (unless a serializer is set) are considered to already be
serialized.

```js
reply
  .header('Content-Type', 'application/x-protobuf')
  .send(protoBuf.serialize(data))
```

See [`.send()`](#send) for more information on sending different types of
values.

### .raw
<a id="raw"></a>

This is the
[`http.ServerResponse`](https://nodejs.org/dist/latest-v14.x/docs/api/http.html#http_class_http_serverresponse)
from Node core. Whilst you are using the Fastify `Reply` object, the use of
`Reply.raw` functions is at your own risk as you are skipping all the Fastify
logic of handling the HTTP response. e.g.:

```js
app.get('/cookie-2', (req, reply) => {
  reply.setCookie('session', 'value', { secure: false }) // this will not be used

  // in this case we are using only the nodejs http server response object
  reply.raw.writeHead(200, { 'Content-Type': 'text/plain' })
  reply.raw.write('ok')
  reply.raw.end()
})
```
Another example of the misuse of `Reply.raw` is explained in
[Reply](#getheaders).

### .sent
<a id="sent"></a>

As the name suggests, `.sent` is a property to indicate if a response has been
sent via `reply.send()`.
It will also be `true` in case `reply.hijack()` was used.

In case a route handler is defined as an async function or it returns a promise,
it is possible to call `reply.hijack()` to indicate that the automatic
invocation of `reply.send()` once the handler promise resolve should be skipped.
By calling `reply.hijack()`, an application claims full responsibility for
the low-level request and response. Moreover, hooks will not be invoked.

*Modifying the `.sent` property directly is deprecated. Please use the aformentioned
`.hijack()` method to achieve the same effect.*

<a name="hijack"></a>
### .hijack()
Sometimes you might need to halt the execution of the normal request lifecycle and
handle sending the response manually.

To achieve this, Fastify provides the `reply.hijack()` method that can be called
during the request lifecycle (At any point before `reply.send()` is called), and
allows you to prevent Fastify from sending the response, and from running the
remaining hooks (and user handler if the reply was hijacked before).

```js
app.get('/', (req, reply) => {
  reply.hijack()
  reply.raw.end('hello world')

  return Promise.resolve('this will be skipped')
})
```

If `reply.raw` is used to send a response back to the user, the `onResponse`
hooks will still be executed.

### .send(data)
<a id="send"></a>

As the name suggests, `.send()` is the function that sends the payload to the
end user.

#### Objects
<a id="send-object"></a>

As noted above, if you are sending JSON objects, `send` will serialize the
object with
[fast-json-stringify](https://www.npmjs.com/package/fast-json-stringify) if you
set an output schema, otherwise, `JSON.stringify()` will be used.
```js
fastify.get('/json', options, function (request, reply) {
  reply.send({ hello: 'world' })
})
```

#### Strings
<a id="send-string"></a>

If you pass a string to `send` without a `Content-Type`, it will be sent as
`text/plain; charset=utf-8`. If you set the `Content-Type` header and pass a
string to `send`, it will be serialized with the custom serializer if one is
set, otherwise, it will be sent unmodified (unless the `Content-Type` header is
set to `application/json; charset=utf-8`, in which case it will be
JSON-serialized like an object — see the section above).
```js
fastify.get('/json', options, function (request, reply) {
  reply.send('plain string')
})
```

#### Streams
<a id="send-streams"></a>

*send* can also handle streams out of the box. If you are sending a stream and
you have not set a `'Content-Type'` header, *send* will set it at
`'application/octet-stream'`.
```js
fastify.get('/streams', function (request, reply) {
  const fs = require('fs')
  const stream = fs.createReadStream('some-file', 'utf8')
  reply.send(stream)
})
```

#### Buffers
<a id="send-buffers"></a>

If you are sending a buffer and you have not set a `'Content-Type'` header,
*send* will set it to `'application/octet-stream'`.
```js
const fs = require('fs')
fastify.get('/streams', function (request, reply) {
  fs.readFile('some-file', (err, fileBuffer) => {
    reply.send(err || fileBuffer)
  })
})
```

#### Errors
<a id="errors"></a>

If you pass to *send* an object that is an instance of *Error*, Fastify will
automatically create an error structured as the following:

```js
{
  error: String        // the HTTP error message
  code: String         // the Fastify error code
  message: String      // the user error message
  statusCode: Number   // the HTTP status code
}
```

You can add custom properties to the Error object, such as `headers`, that will
be used to enhance the HTTP response.

*Note: If you are passing an error to `send` and the statusCode is less than
400, Fastify will automatically set it at 500.*

Tip: you can simplify errors by using the
[`http-errors`](https://npm.im/http-errors) module or
[`fastify-sensible`](https://github.com/fastify/fastify-sensible) plugin to
generate errors:

```js
fastify.get('/', function (request, reply) {
  reply.send(httpErrors.Gone())
})
```

To customize the JSON error output you can do it by:

- setting a response JSON schema for the status code you need
- add the additional properties to the `Error` instance

Notice that if the returned status code is not in the response schema list, the
default behaviour will be applied.

```js
fastify.get('/', {
  schema: {
    response: {
      501: {
        type: 'object',
        properties: {
          statusCode: { type: 'number' },
          code: { type: 'string' },
          error: { type: 'string' },
          message: { type: 'string' },
          time: { type: 'string' }
        }
      }
    }
  }
}, function (request, reply) {
  const error = new Error('This endpoint has not been implemented')
  error.time = 'it will be implemented in two weeks'
  reply.code(501).send(error)
})
```

If you want to customize error handling, check out
[`setErrorHandler`](./Server.md#seterrorhandler) API.

*Note: you are responsible for logging when customizing the error handler*

API:

```js
fastify.setErrorHandler(function (error, request, reply) {
  request.log.warn(error)
  var statusCode = error.statusCode >= 400 ? error.statusCode : 500
  reply
    .code(statusCode)
    .type('text/plain')
    .send(statusCode >= 500 ? 'Internal server error' : error.message)
})
```

The not found errors generated by the router will use the
[`setNotFoundHandler`](./Server.md#setnotfoundhandler)

API:

```js
fastify.setNotFoundHandler(function (request, reply) {
  reply
    .code(404)
    .type('text/plain')
    .send('a custom not found')
})
```

#### Type of the final payload
<a id="payload-type"></a>

The type of the sent payload (after serialization and going through any
[`onSend` hooks](./Hooks.md#onsend)) must be one of the following
types, otherwise, an error will be thrown:

- `string`
- `Buffer`
- `stream`
- `undefined`
- `null`

#### Async-Await and Promises
<a id="async-await-promise"></a>

Fastify natively handles promises and supports async-await.

*Note that in the following examples we are not using reply.send.*
```js
const delay = promisify(setTimeout)

fastify.get('/promises', options, function (request, reply) {
 return delay(200).then(() => { return { hello: 'world' }})
})

fastify.get('/async-await', options, async function (request, reply) {
  await delay(200)
  return { hello: 'world' }
})
```

Rejected promises default to a `500` HTTP status code. Reject the promise, or
`throw` in an `async function`, with an object that has `statusCode` (or
`status`) and `message` properties to modify the reply.

```js
fastify.get('/teapot', async function (request, reply) {
  const err = new Error()
  err.statusCode = 418
  err.message = 'short and stout'
  throw err
})

fastify.get('/botnet', async function (request, reply) {
  throw { statusCode: 418, message: 'short and stout' }
  // will return to the client the same json
})
```

If you want to know more please review
[Routes#async-await](./Routes.md#async-await).

### .then(fulfilled, rejected)
<a id="then"></a>

As the name suggests, a `Reply` object can be awaited upon, i.e. `await reply`
will wait until the reply is sent. The `await` syntax calls the `reply.then()`.

`reply.then(fulfilled, rejected)` accepts two parameters:

- `fulfilled` will be called when a response has been fully sent,
- `rejected` will be called if the underlying stream had an error, e.g. the
  socket has been destroyed.

For more details, see:

- https://github.com/fastify/fastify/issues/1864 for the discussion about this
  feature
- https://promisesaplus.com/ for the definition of thenables
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then
  for the signature
