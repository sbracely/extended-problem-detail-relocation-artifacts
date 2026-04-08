# Copilot Instructions

## Build and test commands

This repository is a Maven multi-module project made entirely of relocation POMs.

- Full validation: `mvn test`
- Validate a single module: `mvn -pl extended-problem-detail-webmvc-autoconfigure test`
- Package relocation POMs without running a deploy: `mvn package`

There are currently no `src/main` or `src/test` trees in the repository, so there is no class-level single-test command to run today. The smallest meaningful test scope is a single Maven module via `-pl`.

## High-level architecture

The root `pom.xml` is an aggregator for four legacy artifact coordinates:

| Old artifact | Relocates to |
| --- | --- |
| `extended-problem-detail-webmvc-autoconfigure` | `extended-problem-detail-boot4-webmvc-autoconfigure` |
| `extended-problem-detail-webflux-autoconfigure` | `extended-problem-detail-boot4-webflux-autoconfigure` |
| `extended-problem-detail-webmvc-spring-boot-starter` | `extended-problem-detail-boot4-webmvc-spring-boot-starter` |
| `extended-problem-detail-webflux-spring-boot-starter` | `extended-problem-detail-boot4-webflux-spring-boot-starter` |

Each child module is `packaging=pom` and exists only to publish a relocation notice through `<distributionManagement><relocation>...</relocation></distributionManagement>`. There is no runtime code in this repository; the actual implementation lives in the main `extended-problem-detail` project, but the POM source and publishing metadata in this repository point here.

The root aggregator is not published (`maven.deploy.skip=true`), but it wires release-time plugins used when publishing the child artifacts:

- `maven-gpg-plugin` signs artifacts during `verify`
- `central-publishing-maven-plugin` handles Maven Central publishing while excluding the root aggregator artifact

## Key conventions

- Treat this as metadata-only infrastructure. Most valid changes are edits to POM metadata, module lists, relocation targets, signing, or publishing configuration.
- Every child module follows the same pattern: legacy artifact coordinates stay in the module's own `groupId` and `artifactId`, while the replacement coordinates are declared only inside `<relocation>`.
- Because the main library split into Boot 3 and Boot 4 lines, relocation messages should explicitly tell users that Boot 4 is the default target and that Boot 3 users should switch to the corresponding `boot3` artifact manually.
- The child modules explicitly set `maven.deploy.skip=false` so they can publish relocation POMs even though the root project does not deploy.
- Build output is signed `.pom` files under each module's `target` directory; there are no jars to inspect and no Spring auto-configuration classes in this repository despite the artifact names.
- If you change versions or artifact names, update the root aggregator and all affected child relocation blocks together so the published compatibility coordinates stay aligned.
