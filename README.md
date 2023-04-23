# Ethereum Relay API Specifications

![CI][ci]

The Relay API consists of two categories of APIs:

* Builder API for builders to submit blocks.
* Data API for anyone to query information about the relay.
  * Things like registrations, bids, and payloads.

For interacting with validators, a relay should follow the [builder-specs][builder-specs].

View the Relay API specifications here:
* https://flashbots.github.io/relay-specs/

## Contributing

The API specification is checked for lint errors before merging pull requests.

To run the linter locally, install it with:
```console
npm install -g @stoplight/spectral-cli@6.2.1
```
and then run it:
```console
spectral lint relay-oapi.yaml
```

### Render API Specification

To render spec in browser, you will simply need an HTTP server to load the
`index.html` file in root of the repo.

For example:
```console
python -m http.server 8080
```

The spec will render at [http://localhost:8080](http://localhost:8080).

### Usage

Local changes will be observable if "dev" is selected in the "Select a
definition" drop-down in the web UI.

It may be necessary to tick the "Disable Cache" box in their browser's
developer tools to see changes after modifying the source.

## Releasing

1. Create and push tag:
    - Ensure `info.version` in `relay-oapi.yaml` file is updated before tagging.
    - CI will create a GitHub release and upload bundled spec file.
2. Add release entrypoint in `index.html`

In `SwaggerUIBundle` configuration (inside `index.html` file), add another
entry in `urls` field. Entry should be in following format (replace `<tag>`
with real tag name from step 1.):

```javascript
{url: "https://github.com/flashbots/relay-specs/releases/download/<tag>/relay-oapi.yaml", name: "<tag>"},
```

[builder-specs]: https://github.com/ethereum/builder-specs
[ci]: https://github.com/flashbots/relay-specs/workflows/CI/badge.svg
