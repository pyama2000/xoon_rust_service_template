# Service name

> **Note**
Add GCP Workload Identity

> ref: https://dev.classmethod.jp/articles/google-cloud-auth-with-workload-identity/

shell variables:

| Name | Description |
|:-|:-|
| github_repository_name | e.g., xoon_rust_service_template |
| project_id | GCP project id |
| service_account_name | e.g.,xoon-rust-service-template |
| provider_name | e.g., xoon-rust-service-template-provider |
| pool_name | GCP Workload Identity pool name |
| repository_name | e.g., "pyama2000/${github_repository_name}" |
| docker_repository_name | Docker repository name |

```shell
#!/usr/bin/env bash

set -eux

github_repository_name=
project_id=
service_account_name=
provider_name=
pool_name=
repository_name=
docker_repository_name=

if [[ ${#service_account_name} -gt 32 ]]; then
  echo "${service_account_name} length over 32"
  exit 1
fi
if [[ ${#provider_name} -gt 32 ]]; then
  echo "${provider_name} length over 32"
  exit 1
fi

# Create serviceaccount
gcloud iam service-accounts create "$service_account_name" \
    --project "$project_id"

# Create Providers
gcloud iam workload-identity-pools providers create-oidc "$provider_name" \
    --project "$project_id" \
    --location global \
    --workload-identity-pool "$pool_name" \
    --display-name "$provider_name" \
    --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
    --issuer-uri="https://token.actions.githubusercontent.com"

# Add role for push Docker image to Artifact Registry
gcloud projects add-iam-policy-binding "$project_id" \
    --role="roles/artifactregistry.writer" \
    --member="serviceAccount:${service_account_name}@${project_id}.iam.gserviceaccount.com"
# Add role for deploy Cloud Run
gcloud projects add-iam-policy-binding "$project_id" \
    --role="roles/run.admin" \
    --member="serviceAccount:${service_account_name}@${project_id}.iam.gserviceaccount.com"
gcloud projects add-iam-policy-binding "$project_id" \
    --role="roles/iam.serviceAccountUser" \
    --member="serviceAccount:${service_account_name}@${project_id}.iam.gserviceaccount.com"

# Describe workload identity pool id
pool_id=$(gcloud iam workload-identity-pools describe "$pool_name" \
    --project "$project_id" \
    --location global \
    --format="value(name)")

# Add role to service account
gcloud iam service-accounts add-iam-policy-binding "${service_account_name}@${project_id}.iam.gserviceaccount.com" \
    --project "$project_id" \
    --role "roles/iam.workloadIdentityUser" \
    --member "principalSet://iam.googleapis.com/${pool_id}/attribute.repository/${repository_name}"

# Describe provider name
provider_id=$(gcloud iam workload-identity-pools providers describe "$provider_name" \
    --project "$project_id" \
    --location global \
    --workload-identity-pool "$pool_name" \
    --format="value(name)")

echo "set GitHub Actions secrets"
echo "GCP_ARTIFACT_REGISTRY_URL=asia-northeast1-docker.pkg.dev/${project_id}/${docker_repository_name}"
echo "GCP_PROJECT_ID=${project_id}"
echo "GCP_PROVIDER_NAME=${provider_id}"
echo "GCP_SERVICE_ACCOUNT_NAME=${service_account_name}"
echo "GCP_SERVICE_NAME=${github_repository_name}"

```

***FIXME***

- README.md
  - Top level heading
  - Prerequisites
- Cargo.toml
  - package.name

## Prerequisites

> **Note**
>
> - Rust 1.62

## Setup

Apply git hooks

```shell
git config core.hooksPath ./scripts/githooks
```
