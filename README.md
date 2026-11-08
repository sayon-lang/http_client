# http_client

A small HTTP client for talking to web APIs from [Sayon](https://github.com/sayon-lang/sayon), written in Sayon on top of `standard/network.sayon`. You give a client the API's base address and the headers every request needs, like a token. It sends and reads JSON, builds query strings and forms, and tries again when the server is busy.

## Using it

Add it to your project's `sayon.toml` and run `sayon install`, which also fetches [json_parser](https://github.com/sayon-lang/json_parser):

```toml
[dependencies]
http_client = "0.1"
```

Then import it through the dependency:

```sayon
json = import("standard/json.sayon")
time = import("standard/time.sayon")
http = import("dependencies/http_client/src/client.sayon")

api = http.new("https://api.example.com/v1")
api.bearer("the token")
api.retry(3, time.milliseconds(250))

user = api.get_json("/users", ["name" = "ada"])
city = api.get_at("/users/1", "/address/city")
made = api.post_json("/notes", json.parse("{{\"text\": \"buy tea\"}}"))
```

## What it has

| Module | What it does |
| --- | --- |
| `src/client.sayon` | `new` makes a `Client`, whose methods `header`, `bearer`, `basic_auth`, `timeout`, and `retry` set what its requests share. `send`, `get`, `delete`, and `post_form` give the `network.Response`; `get_json`, `post_json`, `put_json`, `patch_json` (a JSON Patch), and `send_json` give the JSON answer, and `get_at` the value at a JSON Pointer in it. An answer that is no success, or no JSON, raises an error with its status and the start of what the server said. |
| `src/urls.sayon` | `encode` and `decode` (percent-encoding, RFC 3986), `query` and `form` (a query string, an HTML form's body), and `join` (a base address and a path). |
| `src/base64.sayon` | `encode`, `encode_text`, and `decode` (RFC 4648). |

A client only retries requests that are safe to repeat: GET, HEAD, PUT, DELETE, and OPTIONS. It retries when the connection fails or the server answers 429, 502, 503, or 504, waiting twice as long each time, or as long as the server's `Retry-After` asks, up to a minute.

## Examples

`sayon run` in this folder takes a tour against a small notes API that the example serves itself (`examples/notes.sayon`), so it doesn't need the network. `sayon run examples/registry.sayon -- json` searches a Sayon package registry: the one on your own computer, unless you put another API's address after the words.

## Tests

`sayon test` runs the tests in `src` and in `tests`, where `client.sayon` makes requests to a server of its own.

## License

[Apache License 2.0](LICENSE).
