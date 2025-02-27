# Google Cloud Run

[Configuring Workload Identity Federation for GitHub actions and Terraform Cloud](https://cloud.google.com/blog/products/identity-security/secure-your-use-of-third-party-tools-with-identity-federation)

[Deploy to Coud Run with GitHub Actions](https://cloud.google.com/blog/products/devops-sre/deploy-to-cloud-run-with-github-actions/)

## Create a workload identity pool

```
gcloud iam workload-identity-pools create github-actions-pool --location="global" --description="The pool to authenticate GitHub actions." --display-name="GitHub Actions Pool"
```

## Create a workload identity pool provider

```
gcloud iam workload-identity-pools providers create-oidc github-actions-oidc --workload-identity-pool="github-actions-pool" --issuer-uri="https://token.actions.GitHubusercontent.com/" --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository,attribute.repository_owner=assertion.repository_owner,attribute.branch=assertion.sub.extract('/heads/{branch}/')" --location=global --attribute-condition="assertion.repository_owner=='sdelamo'"
```

## Create a service account for each repository and assign them appropriate IAM permissions

```
gcloud iam service-accounts create mn-starter-app-sa --display-name="Micronaut Starter Application Service Account" --description="manages the application resources"
```

```
gcloud iam service-accounts create mn-starter-networking-sa --display-name="Micronaut Starter Networking Service Account" --description="manages the networking resources"
```

## Add IAM bindings for the workload pool

```
gcloud iam service-accounts add-iam-policy-binding mn-starter-networking-sa@micronaut-guides.iam.gserviceaccount.com --role="roles/iam.workloadIdentityUser" --member="principalSet://iam.googleapis.com/projects/290477064382/locations/global/workloadIdentityPools/github-actions-pool/attribute.repository/sdelamo/micronaut-starter"
```

```
gcloud iam service-accounts add-iam-policy-binding mn-starter-app-sa@micronaut-guides.iam.gserviceaccount.com --role="roles/iam.workloadIdentityUser" --member="principal://iam.googleapis.com/projects/290477064382/locations/global/workloadIdentityPools/github-actions-pool/subject/repo:sdelamo/micronaut-starter:ref:refs/heads/main"
```


### GitHub Action Secrets 

Name: `GOOGLE_APPLICATION_CREDENTIALS`

Value: 
```
projects/290477064382/locations/global/workloadIdentityPools/github-actions-pool/providers/github-actions-oidc
```

Name: `GCLOUD_SERVICE_ACCOUNT_EMAIL`

Value: `mn-starter-app-sa@micronaut-guides.iam.gserviceaccount.com`
