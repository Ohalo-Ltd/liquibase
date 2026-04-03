# Liquibase (Ohalo Fork)

> **This is Ohalo's fork of Liquibase with SHA256 checksum algorithm support for FIPS compliance.**

## Ohalo Fork Changes

This fork adds support for configurable checksum algorithms via the `LIQUIBASE_CHECKSUM_ALGORITHM` environment variable:

| Value | Description |
|-------|-------------|
| `MD5` | Default, backward compatible with existing deployments |
| `SHA256` | FIPS 140-2/140-3 compliant, required for FIPS-enabled environments |

Based on [liquibase/liquibase#6431](https://github.com/liquibase/liquibase/pull/6431).

### Usage

```bash
# For FIPS-compliant deployments
export LIQUIBASE_CHECKSUM_ALGORITHM=SHA256

# For standard deployments (default)
export LIQUIBASE_CHECKSUM_ALGORITHM=MD5
```

**Important**: The checksum algorithm cannot be changed after initial deployment. Liquibase stores checksums in the `DATABASECHANGELOG` table, and changing the algorithm will cause validation failures.

## Release Process

### Prerequisites
- Java 17+
- GitHub CLI (`gh`)

### Steps

1. **Trigger a build** on the `ohalo/v5.0.2` branch:
   ```bash
   gh workflow run ohalo-build.yml --ref ohalo/v5.0.2 --repo Ohalo-Ltd/liquibase
   ```

2. **Wait for the build to complete** and note the run ID from the output or GitHub Actions UI.

3. **Download the build artifacts**:
   ```bash
   mkdir -p /tmp/liquibase-release
   cd /tmp/liquibase-release
   gh run download <RUN_ID> --repo Ohalo-Ltd/liquibase -n liquibase-artifacts -D artifacts
   ```

4. **Rename artifacts** to match re-version.sh expectations:
   ```bash
   mv artifacts/liquibase-0-SNAPSHOT.tar.gz artifacts/liquibase-ohalo_v5.0.2-SNAPSHOT.tar.gz
   mv artifacts/liquibase-0-SNAPSHOT.zip artifacts/liquibase-ohalo_v5.0.2-SNAPSHOT.zip
   ```

5. **Run re-version.sh** to update version strings (will fail on missing source JARs, but processes the core JAR):
   ```bash
   /path/to/liquibase/.github/util/re-version.sh artifacts 5.0.2 ohalo_v5.0.2
   ```

6. **Repack the distribution** with the versioned JAR:
   ```bash
   mkdir -p dist
   tar -xzf artifacts/liquibase-ohalo_v5.0.2-SNAPSHOT.tar.gz -C dist
   cp re-version/out/liquibase-core-5.0.2.jar dist/internal/lib/liquibase-core.jar
   (cd dist && tar -czf ../liquibase-5.0.2-ohalo.tar.gz *)
   (cd dist && zip -qr ../liquibase-5.0.2-ohalo.zip *)
   ```

7. **Verify the version** in the manifest:
   ```bash
   unzip -p dist/internal/lib/liquibase-core.jar META-INF/MANIFEST.MF | grep -E "Bundle-Version|Liquibase-Version"
   # Should show: Liquibase-Version: 5.0.2 and Bundle-Version: 5.0.2
   ```

8. **Update the GitHub release**:
   ```bash
   # Delete old assets
   gh release delete-asset v5.0.2-ohalo liquibase-5.0.2-ohalo.tar.gz --repo Ohalo-Ltd/liquibase -y
   gh release delete-asset v5.0.2-ohalo liquibase-5.0.2-ohalo.zip --repo Ohalo-Ltd/liquibase -y
   
   # Upload new assets
   gh release upload v5.0.2-ohalo liquibase-5.0.2-ohalo.tar.gz liquibase-5.0.2-ohalo.zip --repo Ohalo-Ltd/liquibase
   ```

### Why Re-versioning is Required

The Maven build uses `0-SNAPSHOT` as the version. Without re-versioning, security scanners like Trivy will detect the version as `0.0.0.SNAPSHOT` and incorrectly flag CVEs that were fixed in earlier versions.

The `re-version.sh` script updates:
- `MANIFEST.MF`: `Liquibase-Version` and `Bundle-Version`
- `liquibase.build.properties`: `build.version`
- `pom.properties`: Maven artifact version

---

# Liquibase [![Build and Test](https://github.com/liquibase/liquibase/actions/workflows/run-tests.yml/badge.svg)](https://github.com/liquibase/liquibase/actions/workflows/run-tests.yml) [![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=liquibase&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=liquibase)
<p align="center"><img src="https://github.com/liquibase/liquibase/blob/master/Liquibase.png" width="30%" height="30%"></p>

Liquibase helps millions of developers track, version, and deploy database schema changes. It will help you to:
- Control database schema changes for specific versions
- Eliminate errors and delays when releasing databases
- Automatically order scripts for deployment
- Easily rollback changes
- Collaborate with tools you already use

This repository contains the main source code for Liquibase Community. For more information about the product, see the [Liquibase website](https://www.liquibase.com/).

## Liquibase Automation and Integrations

Liquibase Community has built-in support for a variety of databases. Databases that are not part of Liquibase Community require extensions that you can download for free. Here is the full list of [supported databases](https://www.liquibase.com/supported-databases).

Liquibase can be integrated with Maven, Ant, Gradle, Spring Boot, and other CI/CD tools. For a full list, see [Liquibase Tools & Integrations](https://docs.liquibase.com/tools-integrations/home.html). You can use Liquibase with [GitHub Actions](https://github.com/liquibase/liquibase-github-action-example), [Spinnaker](https://github.com/liquibase/liquibase-spinnaker-plugin), and many different [workflows](https://docs.liquibase.com/workflows/home.html).


## Install and Run Liquibase

### System Requirements
Liquibase system requirements can be found on the [Download Liquibase](https://www.liquibase.com/download) page.

### An H2 in-memory database example for CLI
1. [Download and run the appropriate installer](https://www.liquibase.com/download). 
2. Make sure to add Liquibase to your PATH.
3. Copy the included `examples` directory to the needed location.
4. Open your CLI and navigate to your `examples/sql` or `examples/xml` directory.
5. Start the included H2 database with the `liquibase init start-h2` command.
6. Run the `liquibase update` command.
7. Run the `liquibase history` command to see what has executed!

See also how to [get started with Liquibase in minutes](https://docs.liquibase.com/start/home.html) or refer to our [Installing Liquibase](https://docs.liquibase.com/start/install/home.html) documentation page for more details.

## Documentation

Visit the [Liquibase Documentation](https://docs.liquibase.com/home.html) website to find the information on how Liquibase works.

## Courses

Learn all about Liquibase by taking our free online courses at [Liquibase University](https://learn.liquibase.com/).

## Want to help?

Want to file a bug or improve documentation? Excellent! Read up on our guidelines for [contributing](https://contribute.liquibase.com/)!

### Contribute code 

Use our [step-by-step instructions](https://contribute.liquibase.com/code/) for contributing code to the Liquibase project. 

### Join the Liquibase Community

Earn points for your achievements and contributions, collect and show off your badges, add accreditations to your LinkedIn. [Learn more about the pathway to Legend and benefits](https://www.liquibase.com/community/liquibase-legends). Enjoy being part of the community!

## Liquibase Extensions

[Provide more database support and features for Liquibase](https://contribute.liquibase.com/extensions-integrations/directory/).

## License

Liquibase Community is [licensed under the Functional Source License (FSL)](https://fsl.software/FSL-1.1-ALv2.template.md).

[Liquibase Secure](https://www.liquibase.com/liquibase-secure) has additional features and support and is commercially licensed.

LIQUIBASE is a registered trademark of [Liquibase Inc.](https://www.liquibase.com/company)

## [Contact us](https://www.liquibase.com/contact)

[Liquibase Forum](https://forum.liquibase.org/) 

[Liquibase Blog](https://www.liquibase.com/blog)

[Get Support & Advanced Features](https://www.liquibase.com/pricing)

## Publish Release Manual Trigger to Sonatype 

1. When a PO (Product Owner) or a Team Leader navigates to Publish a release from here -> https://github.com/liquibase/liquibase/releases/, the workflow from /workflow/release-published.yml job is triggered. 
2. When a release is triggered, the workflow file will stop after `Setup` step and an email will be sent out to the list of `approvers` mentioned in job `manual_trigger_deployment`. You can click on the link and perform anyone of the options mentioned in description. 
3. A minimum of 2 approvers are needed in order for the other jobs such as `deploy_maven`, `deploy_javadocs`, `publish_to_github_packages`, etc to be executed.
4. When you view the GitHub PR, make sure to verify the version which is being published. It should say something like `Deploying v4.20.0 to sonatype`



