# Jenkins to GitHub Actions Migration Report

## Summary

Migrated all Jenkins pipeline definitions in this repository to GitHub Actions workflows and archived the original Jenkinsfiles under `.github/ci-archive/`.

## Source pipelines analyzed

| Original path | Pipeline type | Migrated workflow | Notes |
| --- | --- | --- | --- |
| `Jenkinsfile` | Scripted Jenkins pipeline | `.github/workflows/maven-build-and-test.yml` | Maven config-file setup, JDK 11, compile/test, Surefire and site report artifacts. |
| `ksansibledeployment/Jenkinsfile` | Scripted Jenkins pipeline | `.github/workflows/ansible-kubernetes-deployment.yml` | Manual deployment workflow with Ansible, SSH, Nexus artifact download, and remote installation stages. |
| `kspluginparalleltesting/Jenkinsfile` | Scripted Jenkins pipeline with parallel branches and shared library calls | `.github/workflows/plugin-parallel-testing.yml` | Matrix-based test split, JDK 17 build, JaCoCo artifact aggregation, and placeholder hooks for shared-library incrementals scripts. |

## Created workflows

### `.github/workflows/maven-build-and-test.yml`

- Replaces Jenkins `configFileProvider` usage with optional repository secrets.
- Sets up JDK 11 and Maven cache.
- Recreates environment setup, `mvn clean compile`, and `mvn test` stages.
- Uploads Surefire XML reports and Maven site reports as workflow artifacts.

### `.github/workflows/ansible-kubernetes-deployment.yml`

- Uses `workflow_dispatch` inputs for deployment-specific values that were previously expected as Jenkins environment variables.
- Installs Ansible, `sshpass`, and `rsync` on the runner.
- Recreates inventory, hosts, playbook, remote preparation, `init-install.sh`, `k8s-install.sh`, and `skipper.sh` stages.
- Avoids printing generated files containing passwords to workflow logs.

### `.github/workflows/plugin-parallel-testing.yml`

- Converts Jenkins parallel branches to a GitHub Actions matrix with two `kind` test shards.
- Recreates JDK 17 build with a three-attempt shell retry.
- Uploads class files, JaCoCo execution data, test reports, kind logs, and merged JaCoCo report artifacts.
- Jenkins shared library calls `infra.prepareToPublishIncrementals()` and `infra.maybePublishIncrementals()` had no source implementation in this repository, so the workflow calls repository scripts if future equivalents are added at `scripts/prepare-to-publish-incrementals.sh` and `scripts/maybe-publish-incrementals.sh`.

## Secrets and variables required

| Name | Type | Used by | Jenkins equivalent |
| --- | --- | --- | --- |
| `MAVEN_SETTINGS_XML` | Secret | `maven-build-and-test.yml` | Config file provider `global-settings`. |
| `APP_CONFIG_PROPERTIES` | Secret | `maven-build-and-test.yml` | Config file provider `app-config`. |
| `LINUX_USER` | Secret | `ansible-kubernetes-deployment.yml` | `$LINUX_USER`. |
| `LINUX_PASS` | Secret | `ansible-kubernetes-deployment.yml` | `$LINUX_PASS`. |
| `NEXUS_USER` | Secret | `ansible-kubernetes-deployment.yml` | `$NEXUS_USER`. |
| `NEXUS_PASS` | Secret | `ansible-kubernetes-deployment.yml` | `$NEXUS_PASS`. |
| `SSH_PRIVATE_KEY` | Secret | `ansible-kubernetes-deployment.yml` | `/home/ubuntu/.ssh/id_rsa` and `.pub` material used by Jenkins. |
| `ip_address` | Workflow input | `ansible-kubernetes-deployment.yml` | `$IP_ADDRESS`. |
| `nexus_url` | Workflow input | `ansible-kubernetes-deployment.yml` | `$NEXUS_URL`. |
| `virtual_ipaddress` | Workflow input | `ansible-kubernetes-deployment.yml` | `$VIRTUAL_IPADDRESS`. |
| `metallb_ipaddress` | Workflow input | `ansible-kubernetes-deployment.yml` | `$METALLLB_IPADDRESS`. |
| `virtual_router_id` | Workflow input | `ansible-kubernetes-deployment.yml` | `$VIRTUAL_ROUTER_ID`. |
| `storage_class` | Workflow input | `ansible-kubernetes-deployment.yml` | `$STORAGE_CLASS`. |
| `local_path_provisioner_path` | Workflow input | `ansible-kubernetes-deployment.yml` | `$LOCAL_PATH_PROVISIONER_PATH`. |

## Archived originals

Original Jenkinsfiles were moved to:

- `.github/ci-archive/root/Jenkinsfile`
- `.github/ci-archive/ksansibledeployment/Jenkinsfile`
- `.github/ci-archive/kspluginparalleltesting/Jenkinsfile`

## Validation

- GitHub Actions use official GitHub-created actions pinned to immutable tag SHAs.
- Workflows are intentionally manual (`workflow_dispatch`) because this repository contains pipeline examples and does not include the Maven/Ansible project files needed for automatic push or pull request execution.
- Action syntax was validated with `actionlint` during migration.

## Follow-up notes

- Configure the required repository secrets before running the migrated workflows.
- Add repository scripts for the Jenkins shared library incrementals calls if equivalent publish behavior is required.
- Review the remote block-device wipe step in `ansible-kubernetes-deployment.yml` before production use; it preserves the Jenkins behavior but remains intentionally manual.
