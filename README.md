# jwkcache

JWK Set caching backed by [httprc](https://github.com/lestrrat-go/httprc).

This package was extracted from the `jwk` package in [github.com/lestrrat-go/jwx/v4](https://github.com/lestrrat-go/jwx) to keep the core jwx/v4 module free of the httprc dependency.

## Install

```
go get github.com/jwx-go/jwkcache/v4
```

Requires `GOEXPERIMENT=jsonv2`.

## Usage

```go
import _ "github.com/jwx-go/jwkcache/v4"
```

### Register Options

| Option | Description |
|--------|-------------|
| `WithConstantInterval(d)` | Use a fixed refresh interval |
| `WithMinInterval(d)` | Set the minimum refresh interval |
| `WithMaxInterval(d)` | Set the maximum refresh interval |
| `WithHTTPClient(c)` | Override the HTTP client for this resource |
| `WithWaitReady(bool)` | Whether `Register` blocks until the first fetch completes (default: true) |
| `WithMaxFetchBodySize(n)` | Override the max response body size (default: 10 MB) |

### CachedSet

`Cache.CachedSet(url)` returns a read-only `jwk.Set` that always reflects the latest cached data. All mutation methods return errors.

### Fetcher

`NewFetcher(cache)` wraps a `Cache` as a `jwk.Fetcher`, suitable for passing to APIs that accept one.

## Security

jwkcache does not validate URLs itself. URL policy lives on the
`httprc.Client` you pass to `NewCache`, and `httprc.NewClient` defaults
to `httprc.InsecureWhitelist{}` — every URL is allowed. That default is
fine when the URLs you register are hard-coded or come from trusted
configuration, but it is **not** safe when a URL can be influenced by
untrusted input — most commonly a `jku` header copied out of an
untrusted JWS.

For untrusted URLs, construct the `httprc.Client` with an explicit
whitelist:

- `httprc.NewMapWhitelist()` for an exact set of URLs,
- `httprc.NewRegexpWhitelist()` with patterns anchored as
  `^https://your\.host/`,
- or a custom implementation of `httprc.Whitelist`.

For additional defense against redirect-to-private-IP and DNS-rebinding
attacks, pass an `http.Client` via `httprc.WithHTTPClient` whose
`Transport.DialContext` validates resolved addresses.
