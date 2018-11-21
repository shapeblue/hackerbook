# CloudStack API Development

## Skeleton Implementation

API plugin skeleton + maven project TODO.

## Types of CloudStack APIs

- Synchronous: implements `BaseCmd`, executes in a blocking way until the
  response is returned.
- Asynchronous: implements `BaseAsyncCmd` or `BaseAsyncCreateCmd`, creates an
  async job and returns a pollable job ID via `queryAsyncJobResult` API.

## API handling in CloudStack

TODO: how API is intercepted and handled.

## API implementation

TODO: Implementing a general API + exporting it.

A class implementing `PluggableService` interface exports API classes via
`getCommands` method.

## Challenges

- Attempt and fix CloudStack API issue(s): https://github.com/apache/cloudstack/issues?q=is%3Aissue+is%3Aopen+label%3Aapi
