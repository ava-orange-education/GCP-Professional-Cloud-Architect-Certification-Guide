# Scenario: Granting Temporary Access to a User
To provide a user with temporary read-only access to Cloud Storage objects:
1. Create the iam Role
```bash
gcloud iam roles create tempAccess --project=my-project \
    --title="Temporary Access" \
    --permissions="storage.objects.list,storage.objects.get" \
    --stage=ALPHA
```
2. Attach the IAM Role
```bash
gcloud projects add-iam-policy-binding my-project \
    --member=user:example@gmail.com \
    --role=projects/my-project/roles/tempAccess
```

# Scenario: Restricting Service Account Permissions
To limit a service account’s access to Cloud Storage:

1. Create the IAM Role
```bash
gcloud iam roles create storageLimitedRole --project=my-project \
    --title="Limited Storage Access" \
    --permissions="storage.objects.get,storage.objects.list"
```
2. Add the IAM role to a service account
```bash
gcloud iam service-accounts add-iam-policy-binding my-sa@my-project.iam.gserviceaccount.com \
    --role=projects/my-project/roles/storageLimitedRole \
    --member=serviceAccount:my-sa@my-project.iam.gserviceaccount.com
```