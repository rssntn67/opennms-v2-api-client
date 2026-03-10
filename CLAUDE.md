# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Java REST API client library for OpenNMS (network monitoring system), auto-generated from two OpenAPI 3.0 specifications using OpenAPI Generator. It is packaged as an OSGi bundle.

- **GroupId:** `it.arsinfo`
- **ArtifactId:** `opennms-v2-api-client`
- **Package roots:** `it.arsinfo.clients.opennms` and `it.arsinfo.clients.opennms.v2`

## Build Commands

```bash
mvn clean install          # Full build + install to local Maven repo
mvn clean package          # Build without installing
mvn clean generate-sources # Regenerate code from OpenAPI specs only
mvn test                   # Run tests (no tests currently exist)
```

## Architecture

The project follows a code-generation pattern with two specs:

1. **Source of truth:** `src/build/resources/openapi-opennms.json` and `src/build/resources/openapi-opennms-v2.json` (OpenNMS 35.0.3)
2. **Codegen:** OpenAPI Generator Maven plugin (v7.9.0) runs two executions, generating sources into separate subdirectories under `target/generated-sources/`
3. **Build helper plugin** adds both generated source directories to the compile path
4. **OSGi bundle plugin** exports `it.arsinfo.clients.opennms.*` and `it.arsinfo.clients.opennms.v2.*`

### Generated Package Structure

| Package | Contents | Source spec |
|---|---|---|
| `it.arsinfo.clients.opennms.handler` | Shared `ApiClient` (Jersey2), `Configuration`, `ApiException`, JSON/auth utilities | `openapi-opennms.json` |
| `it.arsinfo.clients.opennms.api` | API clients | `openapi-opennms.json` |
| `it.arsinfo.clients.opennms.model` | Model classes | `openapi-opennms.json` |
| `it.arsinfo.clients.opennms.v2.api` | API clients (OpenNMS 35.0.3) | `openapi-opennms-v2.json` |
| `it.arsinfo.clients.opennms.v2.model` | Model classes (OpenNMS 35.0.3) | `openapi-opennms-v2.json` |

The handler package is shared: the v2 execution sets `generateSupportingFiles=false` and points its `invokerPackage` to `it.arsinfo.clients.opennms.handler`.

### Key Design Points

- **Jersey2 library:** HTTP client uses JAX-RS 2.1 / Jersey 2.42
- **`openApiNullable=false`:** Disables `JsonNullable` wrappers (avoids `jackson-databind-nullable` dependency)
- **`skipValidateSpec=true`:** The OpenNMS 35.0.3 spec has undeclared path parameters that would block generation
- **Do not manually edit generated sources** — they are overwritten on every `mvn generate-sources`. Make changes via the OpenAPI spec or plugin configuration in `pom.xml`
- **`src/main/java/it/arsinfo/clients/opennms/v2/model/MapStringTimeTrackingMonitor.java`** is a manually-written stub for a type the generator references but does not produce (generator bug)

## Modifying the Client

To add/change API endpoints or models, edit the relevant spec in `src/build/resources/`, then run `mvn clean generate-sources` to regenerate. Plugin configuration (package names, library, options) is in the `openapi-generator-maven-plugin` section of `pom.xml`.