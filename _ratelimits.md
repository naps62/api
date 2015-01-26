# Rate Limits

The API applies rate limits based on the number of requests per a predefined interval (i.e. a time-window).

Currently, we do not differentiate between authenticated and unauthenticated requests. The global rate limit takes into account the remote client ip only.

Some sensitive endpoints have stricter rules and may additionally take into consideration the user account (email). For example, there can be 10 requests per 10 minutes per IP to `/password/forgot`, but multiple different IPs can only make 3 requests per 5 minutes per user account (e.g. `foo@bar.com`).

The following table indicates the current rate limits:

| Endpoint              | Requests (per ip) / window | Requests (per user) / window |
| ----------------------|---------------------------:|-----------------------------:|
| *Global*              | 300 / 5-min                | N/A                          |
| POST /oauth2/token    | 10 / 1-min window          | 10 / 1-min window            |
| POST /password/forgot | 10 / 10-min window         | 3 / 5-min window             |
| POST /users           | 10 / 10-min window         | N/A                          |

Notes:
  - *N/A* refers to *Not Applicable*.

## Response Headers

The current rate limit in effect is explained via custom HTTP headers as described in the table below. Additionally, the standard HTTP `Retry-After` header field will be appended when the rate limit is exhausted and indicates, in delta-seconds, how long until the rate limit window is reset.

| Header                | Description                                                                                                            |
|-----------------------|------------------------------------------------------------------------------------------------------------------------|
| X-RateLimit-Limit     | The total number of requests possible in the current window duration                                                   |
| X-RateLimit-Remaining | The number of requests remaining in the current window duration                                                        |
| X-RateLimit-Reset     | The time, in UTC [epoch seconds](http://en.wikipedia.org/wiki/Unix_time), until the end of the current window duration |
| Retry-After           | The time, in seconds, until the end of the current window duration                                                     |


> Example request:

```
curl -I -X GET "https://api.bitreserve.org/v0/ticker"
```

> Rate limit details on response headers:

```
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 299
X-RateLimit-Reset: 1422288284
```

When the API limit is reached, a status code of [429 Too Many Requests](http://tools.ietf.org/html/rfc6585#section-4) is returned with the aforementioned `Retry-After` header:

> Example response for a rate limited request:

```
HTTP/1.1 429 Too Many Requests

X-RateLimit-Limit: 300
X-RateLimit-Remaining: 299
X-RateLimit-Reset: 1422288284
Retry-After: 85
```

In this this example, the request could be retried in 1 minute and 25 seconds.

Alternatively, the `X-RateLimit-Reset` header could also be used to calculate when to retry the request:

> Parsing *X-RateLimit-Reset*:

```js
console.log(new Date(1422288199 * 1000));

// Mon Jan 26 2015 16:30:11 GMT+0000 (WET)
```

If you think you have a legitimate use-case for increased rate limits, please [contact us](/#support).
