# Quality Assurance

Quality assurance for TERRA UGLA is split between implementation testing in the private code repository and public documentation checks in this repository.

## Implementation QA

The implementation test suite should cover:

- page smoke tests for forecast and validation interfaces;
- AOI and fixed-geometry API contracts;
- latest-observation extraction endpoint behavior;
- forecast endpoint validation;
- retrospective validation slicing, ranking, and error handling;
- result download and artifact-serving behavior.

## Documentation QA

The public documentation repository should be checked for:

- no committed credentials or private runtime configuration;
- no service source code or generated model/data artifacts;
- valid MkDocs navigation;
- working links to OpenAPI and technical route pages;
- clear separation between public documentation and private implementation details.

## Release Checklist

Before publishing a documentation version:

1. confirm `docs/mkdocs.yml` uses the correct `site_url` and `repo_url`;
2. run a local MkDocs build where dependencies are available;
3. scan the tree for code, credentials, generated data, and model artifacts;
4. publish with the on-demand workflow and a clear version label.
