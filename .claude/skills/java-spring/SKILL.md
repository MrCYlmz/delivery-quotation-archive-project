---
name: java-spring
description: Java 21, Spring Boot 3, spring-jdbc on SQLite, and Maven conventions.
---

## Versions

Use the versions in `pom.xml`. Much of the Spring code models learned from is Spring Boot 2, so these older forms keep coming back:

- `jakarta.*` imports, never `javax.*`.
- `RestClient` or an HTTP service interface, never `RestTemplate`.

## Java

- Use records for request and response types and value objects. They are immutable and need no boilerplate.
- Use sealed types and switch pattern matching where they replace `instanceof` chains.
- Build file paths with `java.nio.file.Path`, never by joining strings, because the app runs on Windows and Linux.

## Spring

- Inject through the constructor into `final` fields, never `@Autowired` on fields, so a class can be built in a test without Spring.
- Bind new configuration to `@ConfigurationProperties` records instead of scattered `@Value` fields, so it is typed and validated at startup. Every setting gets a `DEW_*` environment variable with a default in `application.yml`.
- Validate request bodies with Jakarta Bean Validation and `@Valid`.

## Persistence

- No JPA, Hibernate or Spring Data repositories: Spring Data has no SQLite dialect (playbook decisions).
- Read and write dates and timestamps through `repo/Columns`: SQLite has no date type, so they are stored as ISO-8601 text.
- Compare `BigDecimal` with `compareTo`, never `equals`: SQLite drops trailing zeros, so 4210.00 comes back as 4210.
- Store enums by name, never by ordinal, so reordering an enum cannot corrupt rows.

## Maven

- Add dependencies that Spring Boot manages without a version, so they stay compatible with each other.
- There is no wrapper: build with `mvn verify`, or `mvn -Dskip.frontend=true verify` when only Java changed.
