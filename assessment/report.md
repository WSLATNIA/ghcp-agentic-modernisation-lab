# Order Service modernization assessment

## Purpose and provenance

This is the committed assessment handoff and source of truth for a separate planning session. It consolidates the completed assessment; it is not an implementation plan and does not authorize application changes. Preserve the target and recommendations below when planning.

- Application root: `app/Java - Spring Boot/Order Service`.
- Assessment run: `20261008124219` (2026-10-08 UTC).
- Assessment branch: `assess-order-service-java-25`.
- Original evidence commit: `56e232ca7cc572df640361cc70556d9be3eb5381`.
- Status: `success`; artifact validation: `passed`; language: `java`; planning supported: `true`.
- Domains: `java-upgrade`, `security`. Coverage: `issue-only` (source: `default`).
- All seven security tasks completed; no failed or partial tasks.
- Application source and build manifests were unchanged by the assessment.

## Current state

- Framework: Spring Boot 2.7.18.
- Java source/target: 8; observed assessment runtime: Microsoft OpenJDK 17.0.18.
- Database: H2 2.1.214, in-memory.
- Maven backend with Spring Web, Data JPA and Bean Validation; React/Vite frontend.

## Target state and validation boundary

- Target runtime and compiler release: **Java 25**.
- Recommended framework line: **Spring Boot 3.5.x**, using a supported compatible patch selected and rescanned at implementation time.
- Compatibility documentation observed during assessment: Spring Boot 3.5.16, https://docs.spring.io/spring-boot/3.5/system-requirements.html.
- Official system requirements observed during assessment state Java 17 minimum and compatibility through Java 25. Select and rescan the supported current patch at implementation time; this is not a guarantee of indefinite support.
- Compatibility of the target framework line was checked against official documentation. The migrated application has NOT been built or run on Java 25. Verification passed for assessment artifacts, not for a migrated target implementation.
- The user requested preservation of recorded recommendations for planning. No exact final patch, cloud deployment target, architecture redesign or production security policy was approved by this assessment.

## Recorded recommendations and migration evidence

### build-java25

**Observation:** Java version and compiler source/target are fixed to 8 in both properties and plugin configuration.

**Recommendation to preserve:** Install/select JDK 25 for Maven and runtime. Use a consistent release 25 compiler configuration and remove conflicting Java 8 overrides. Confirm plugin and test tooling compatibility on JDK 25.

**Evidence (application-relative):** `pom.xml:20-22`, `pom.xml:85-86`.

### jakarta-migration

**Observation:** JPA and Bean Validation imports use javax.persistence and javax.validation.

**Recommendation to preserve:** Move the affected imports to jakarta.persistence and jakarta.validation with the Spring Boot 3 framework/BOM migration. Do not indiscriminately replace unrelated javax JDK namespaces.

**Evidence (application-relative):** `src/main/java/com/contoso/demo/orderservice/model/Order.java:3-14`, `src/main/java/com/contoso/demo/orderservice/web/OrderController.java:17`.

### spring-mvc-api

**Observation:** Configuration extends deprecated WebMvcConfigurerAdapter.

**Recommendation to preserve:** Implement WebMvcConfigurer and preserve the API route/method behavior. Review wildcard CORS separately before production exposure.

**Evidence (application-relative):** `src/main/java/com/contoso/demo/orderservice/config/WebConfig.java:5-19`.

### framework-behavior

**Observation:** The upgrade changes Spring Framework, Hibernate/JPA, validation and test dependencies.

**Recommendation to preserve:** Upgrade through a controlled Java 17/Spring Boot 3 transition, then validate on Java 25. Preserve POST validation, status reset to PENDING, PATCH enum validation, 404 responses, BigDecimal totals and date serialization against existing tests.

**Evidence (application-relative):** `pom.xml:7-12`, `src/main/java/com/contoso/demo/orderservice/model/Order.java`, `src/main/java/com/contoso/demo/orderservice/repository/OrderRepository.java`, `src/test/java`.

### dependency-remediation

**Observation:** Direct Maven advisory queries identified vulnerable Log4j Core, Commons Text and H2 versions.

**Recommendation to preserve:** Remove unused explicit Log4j Core and Commons Text dependencies after confirming no use, or upgrade to supported patched releases. The direct-scope advisory set requires Log4j Core at least 2.25.4, Commons Text at least 1.10.0 and H2 at least 2.2.220; these are advisory minima, not recommended final patch pins. Prefer compatible Boot BOM management and repeat a full transitive advisory scan.

**Evidence (application-relative):** `pom.xml:44-59`, `.github/modernize/assessment/engines/security/github-advisories.json`.

## Security findings

The verification receipt tracks **12 findings**: 5 high, 6 medium and 1 low under its normalized mapping. The original advisory severities and CWE classifications below remain authoritative for their respective sources; normalized counts do not mean Log4Shell lacks critical advisory severity.

Direct Maven dependency scanning returned 9 CVE findings. The six CWE categories assessed 59 rules: 3 FOUND and 56 NOT_FOUND. NOT_FOUND is bounded source-review evidence, not a security guarantee.

### Dependency advisories

#### CVE-2022-45868: Password exposure in H2 Database

- Assessment classification: `mandatory`.
- Evidence: `pom.xml:41`.
- https://github.com/advisories/GHSA-22wj-vf5f-wrvj; severity=high; com.h2database:h2:2.1.214; advisory ranges and first patches: >= 1.4.198, < 2.2.220 => 2.2.220; dependency presence verified, application exploitability not proven. Scope: direct Maven dependencies only.

#### CVE-2026-34480: Apache Log4j Core: Silent log event loss in XmlLayout due to unescaped XML 1.0 forbidden characters

- Assessment classification: `optional`.
- Evidence: `pom.xml:49`.
- https://github.com/advisories/GHSA-3pxv-7cmr-fjr4; severity=medium; org.apache.logging.log4j:log4j-core:2.14.1; advisory ranges and first patches: >= 2.0-alpha1, < 2.25.4 => 2.25.4 | >= 3.0.0-alpha1, <= 3.0.0-beta3 => ; dependency presence verified, application exploitability not proven. Scope: direct Maven dependencies only.

#### CVE-2022-42889: Arbitrary code execution in Apache Commons Text

- Assessment classification: `mandatory`.
- Evidence: `pom.xml:55`.
- https://github.com/advisories/GHSA-599f-7c49-w659; severity=critical; org.apache.commons:commons-text:1.9; advisory ranges and first patches: >= 1.5, < 1.10.0 => 1.10.0; dependency presence verified, application exploitability not proven. Scope: direct Maven dependencies only.

#### CVE-2026-34477: Apache Log4j Core: `verifyHostName` attribute silently ignored in TLS configuration

- Assessment classification: `optional`.
- Evidence: `pom.xml:49`.
- https://github.com/advisories/GHSA-6hg6-v5c8-fphq; severity=medium; org.apache.logging.log4j:log4j-core:2.14.1; advisory ranges and first patches: >= 2.12.0, < 2.25.4 => 2.25.4 | >= 3.0.0-alpha1, <= 3.0.0-beta3 => ; dependency presence verified, application exploitability not proven. Scope: direct Maven dependencies only.

#### CVE-2021-45046: Incomplete fix for Apache Log4j vulnerability

- Assessment classification: `mandatory`.
- Evidence: `pom.xml:49`.
- https://github.com/advisories/GHSA-7rjr-3q55-vv33; severity=critical; org.apache.logging.log4j:log4j-core:2.14.1; advisory ranges and first patches: >= 2.13.0, < 2.16.0 => 2.16.0 | >= 2.4.0, < 2.12.2 => 2.12.2 | < 2.3.1 => 2.3.1; dependency presence verified, application exploitability not proven. Scope: direct Maven dependencies only.

#### CVE-2021-44832: Improper Input Validation and Injection in Apache Log4j2

- Assessment classification: `optional`.
- Evidence: `pom.xml:49`.
- https://github.com/advisories/GHSA-8489-44mv-ggj8; severity=medium; org.apache.logging.log4j:log4j-core:2.14.1; advisory ranges and first patches: >= 2.0-beta7, < 2.3.2 => 2.3.2 | >= 2.4, < 2.12.4 => 2.12.4 | >= 2.13.0, < 2.17.1 => 2.17.1; dependency presence verified, application exploitability not proven. Scope: direct Maven dependencies only.

#### CVE-2021-44228: Remote code injection in Log4j

- Assessment classification: `mandatory`.
- Evidence: `pom.xml:49`.
- https://github.com/advisories/GHSA-jfh8-c2jp-5v3q; severity=critical; org.apache.logging.log4j:log4j-core:2.14.1; advisory ranges and first patches: >= 2.13.0, < 2.15.0 => 2.15.0 | >= 2.4, < 2.12.2 => 2.12.2 | >= 2.0-beta9, < 2.3.1 => 2.3.1; dependency presence verified, application exploitability not proven. Scope: direct Maven dependencies only.

#### CVE-2021-45105: Apache Log4j2 vulnerable to Improper Input Validation and Uncontrolled Recursion

- Assessment classification: `mandatory`.
- Evidence: `pom.xml:49`.
- https://github.com/advisories/GHSA-p6xc-xr62-6r2g; severity=high; org.apache.logging.log4j:log4j-core:2.14.1; advisory ranges and first patches: >= 2.4.0, < 2.12.3 => 2.12.3 | >= 2.13.0, < 2.17.0 => 2.17.0 | < 2.3.1 => 2.3.1; dependency presence verified, application exploitability not proven. Scope: direct Maven dependencies only.

#### CVE-2025-68161: Apache Log4j does not verify the TLS hostname in its Socket Appender

- Assessment classification: `optional`.
- Evidence: `pom.xml:49`.
- https://github.com/advisories/GHSA-vc5p-v9hr-52mj; severity=medium; org.apache.logging.log4j:log4j-core:2.14.1; advisory ranges and first patches: >= 2.0-beta9, < 2.25.3 => 2.25.3; dependency presence verified, application exploitability not proven. Scope: direct Maven dependencies only.

### Application-source findings

#### CWE-477: Use of Obsolete Function

- Classification: `optional`; category: Code Quality.
- Evidence: `src/main/java/com/contoso/demo/orderservice/config/WebConfig.java`.
- WebConfig imports deprecated WebMvcConfigurerAdapter at line 5 and extends it at line 13. This obsolete Spring MVC API is a concrete framework upgrade blocker, not evidence of a remotely exploitable vulnerability.

#### CWE-778: Insufficient Logging

- Classification: `potential`; category: Credentials & Secrets.
- Evidence: `src/main/java/com/contoso/demo/orderservice/service/OrderService.java`, `src/main/resources/application.properties`.
- OrderService.create (lines 35-38) and updateStatus (lines 40-49) persist order creation and status mutations without recording an application audit event, actor, order identifier or old/new status. The SQL logging setting at application.properties line 7 is not an attributable business/security audit trail. Impact concerns investigation of unauthorized order mutations if the demo is deployed.

#### CWE-798: Use of Hard-coded Credentials

- Classification: `optional`; category: Credentials & Secrets.
- Evidence: `src/main/resources/application.properties`.
- application.properties lines 1-4 explicitly configure the in-memory H2 database with the fixed sa account and an empty password. The H2 console is enabled at line 8. This is a local demo credential configuration, not a leaked production secret; server.address=127.0.0.1 and web-allow-others=false at lines 9-10 constrain exposure.

### Automated recommendation caveat

The verifier recorded: "Address top vulnerability: Password exposure in H2 Database". This is an automated report recommendation, not an approved implementation order. Preserve its H2 evidence, but account for reachability and the critical Log4j advisory severities when prioritizing remediation. H2's CLI password exposure scenario was not used by the application.

## Existing behavior to preserve

- POST validates required customer/amount fields and amount >= 0.01; creation resets status to PENDING.
- PATCH validates OrderStatus enum values and retains expected missing-order 404 responses.
- GET list/detail and customer totals retain their API contracts; totals use BigDecimal.
- Preserve date serialization, seeded demo data behavior and frontend API integration unless a separately approved change requires otherwise.
- Existing test evidence earlier in this session: `mvn clean test` passed on Java 17 with 20 tests, zero failures/errors/skips. This is a baseline only, not Java 25 validation.

## Scope, risk and limitations

- AppCAT openjdk25 source rules reported no incidents. This does not establish whole-framework or dependency compatibility, and AppCAT reported no dependency output.
- Dependency advisory assessment covers only the seven direct Maven dependencies. Transitive dependencies and frontend npm packages were not included in the CVE scan.
- No JDK 25 build, migration or vulnerability exploitation was executed. Dependency presence is not proof that the vulnerable functionality is reachable. The current source uses no Commons Text interpolation and no direct Log4j Core API; default Spring Boot web logging uses Logback.
- H2 CVE-2022-45868 concerns a CLI admin password argument not used by this application; its affected dependency is nevertheless present.
- H2 console and fixed blank-password sa configuration are local demo settings. The backend binds to 127.0.0.1; no production exposure was established.
- The plugin CWE catalog is bounded and is not a complete authentication, authorization or penetration audit.
- Framework migration changes Spring, Hibernate/JPA, validation and test tooling together; API/data compatibility must not be inferred from the zero AppCAT incident count.
- Local Windows PATH/JAVA_HOME corrections and AppCAT installation are machine setup, not portable repository configuration.

## Committed evidence locations

All paths below are relative to the application root. Absolute Windows paths in original receipts identify the assessment execution host and must not be required by another session.

- Public report: `.github/modernize/assessment/reports/report-20261008124219/report.json`.
- Verification receipt (including complete completionEvidence): `.github/modernize/assessment/reports/report-20261008124219/verification.json`.
- Recorded target, recommendations and limitations: `.github/modernize/assessment/reports/report-20261008124219/migration-review.json`.
- Interactive report: `.github/modernize/reports/20261008124219-assess.html`.
- AppCAT source results: `.github/modernize/.memory/runs/20261008124219/appcat/report.json`.
- Normalized assessment: `.github/modernize/.memory/runs/20261008124219/normalized-assessment.json`.
- Original GitHub advisory evidence: `.github/modernize/assessment/engines/security/github-advisories.json`.
- Complete security outputs: `.github/modernize/assessment/engines/security/incoming/`.

## Separate planning-session contract

Read this committed artifact before creating the plan. Keep all planning in the new repository session in Plan mode with Auto model selection. Produce a prioritized, executable modernization plan that preserves the recorded target, recommendations, behavior and qualifications. Include dependencies, risk, scope and measurable validation criteria for every task. Resolve unvalidated assumptions explicitly rather than labeling them validated. Do not implement modernization in this assessment session.
