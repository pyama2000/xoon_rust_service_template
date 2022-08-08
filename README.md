# Service name

> **Note**
Add GCP Workload Identity

> ref: https://dev.classmethod.jp/articles/google-cloud-auth-with-workload-identity/

environment variables:

| Name | Description |
|:-|:-|
| PROJECT_ID | GCP project id |
| SERVICE_ACCOUNT_NAME | e.g.,xoon-rust-service-template |
| PROVIDER_NAME | e.g., xoon-rust-service-template-provider |
| WORKLOAD_IDENTITY_POOL_NAME | GCP Workload Identity pool name |
| WORKLOAD_IDENTITY_POOL_ID | GCP Workload Identity pool id |

```shell
# Create serviceaccount
gcloud iam service-accounts create "$SERVICE_ACCOUNT_NAME" \
    --project "$PROJECT_ID"

# Create Providers
gcloud iam workload-identity-pools providers create-oidc "$PROVIDER_NAME" \
    --project "$PROJECT_ID" \
    --location global \
    --workload-identity-pool "$WORKLOAD_IDENTITY_POOL_NAME" \
    --display-name "$PROVIDER_NAME" \
    --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
    --issuer-uri="https://token.actions.githubusercontent.com"

# Describe workload identity pool id
gcloud iam workload-identity-pools providers describe "$PROVIDER_NAME" \
    --project "$PROJECT_ID" \
    --location global \
    --workload-identity-pool "$WORKLOAD_IDENTITY_POOL_NAME" \
    --format="value(name)"

# Add role to service account
gcloud iam service-accounts add-iam-policy-binding "${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" \
    --project "$PROJECT_ID" \
    --role "roles/iam.workloadIdentityUser" \
    --member "principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPOSITORY_NAME}"

# Describe provider name
gcloud iam workload-identity-pools providers describe "$PROVIDER_NAME" \
    --project "$PROJECT_ID" \
    --location global \
    --workload-identity-pool "$WORKLOAD_IDENTITY_POOL_NAME" \
    --format="value(name)"
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
