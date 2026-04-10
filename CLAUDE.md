# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

### Standard Maven Commands
```bash
# Build entire project
mvn clean install

# Build specific module with dependencies
mvn clean install -pl iacc-facade -am

# Run tests
mvn test

# Package application
mvn clean package

# Run Spring Boot application (from bootstrap module)
mvn spring-boot:run -pl bootstrap
```

### Deployment
```bash
# Deploy facade module (public API) and dependencies
./deploy.sh  # executes: mvn clean deploy -pl iacc-facade -am
```

## Architecture

IACC (Identity Authentication and Authority Control Center) is a Java 17 Spring Boot 3.1.5 multi-module project following Clean Architecture/Hexagonal patterns.

### Module Structure & Dependencies

- **`app/facade`** (`iacc-facade`): DTOs, API interfaces, validation. Depends on `rapidf-restful` and `rapidf-validation`.
- **`app/domain`** (`iacc-domain`): Core domain models and business logic. Uses `@DomainLayer` annotation from RapidF framework. Depends on facade, `rapidf-domain`, `rapidf-utils`, and `cell-spring-boot-starter`.
- **`app/application`** (`iacc-application`): Application services coordinating domain logic. Depends on domain and `iacc-infra-acl`.
- **`app/adapter`** (`iacc-adapter`): REST controllers (Spring Web). Depends on application.
- **`app/infrastructure`** (parent module):
  - **`dal`** (`iacc-infra-dal`): Data access with MyBatis, Druid connection pool, MySQL.
  - **`integration`** (`iacc-infra-integration`): External service integrations (currently empty).
  - **`acl`** (`iacc-infra-acl`): Anti-corruption layer isolating domain from infrastructure.
- **`bootstrap`** (`iacc-bootstrap`): Spring Boot entry point (`BootstrapApplication`). Scans `com.lazycece.iacc` package, loads MyBatis mappers from `com.lazycece.iacc.infra.dal.mapper`. Depends on adapter.
- **`test`** (`iacc-test`): Integration test module (tests not yet implemented). Depends on bootstrap and `spring-boot-starter-test`.

### Dependency Flow
```
bootstrap → adapter → application → domain → facade
                                  ↓
                           infra-acl → infra-dal
                                  ↓
                           infra-integration
```

### Technology Stack
- Java 17, Maven, Spring Boot 3.1.5
- MyBatis ORM, Druid connection pool, MySQL
- **RapidF Framework** (custom DDD framework) – provides `rapidf-*` modules
- **Cell Spring Boot Starter** (custom starter)

## Configuration

- Environment properties are loaded from `../conf/environment/*.properties` relative to the bootstrap module.
- Properties define app name, port, log path, and database connection details.
- Use `application-dev.properties` for development, `application-test.properties` for testing.
- Main configuration: `bootstrap/src/main/resources/application.properties` references these properties.

## Testing

- The dedicated `test` module depends on `bootstrap` and `spring-boot-starter-test`.
- Test structure mirrors main packages: `com.lazycece.iacc.test.app.*`, `com.lazycece.iacc.test.bootstrap`.
- HTTP client environment file: `test/src/test/java/com/lazycece/iacc/test/app/adapter/http-client.env.json`.
- Run all tests with `mvn test`.

## Development Notes

- Follow Clean Architecture: domain layer should not depend on infrastructure or adapter layers.
- The Anti-Corruption Layer (`infra/acl`) isolates domain models from infrastructure models.
- The facade module is the published API; deployment script only deploys facade.
- RapidF annotations (e.g., `@DomainLayer`) mark architectural boundaries.
- Use Maven module‑scoped commands (`-pl <module> -am`) to work on specific layers efficiently.