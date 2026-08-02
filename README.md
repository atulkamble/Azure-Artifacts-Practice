# Azure Artifacts Universal Packages - Complete Hands-on Guide

This guide walks through creating, publishing, downloading, versioning, and troubleshooting Azure Artifacts Universal Packages. It also includes a verified workaround for the Azure CLI wrapper authentication issue discovered during hands-on testing.

## Document Structure

1. Project Setup
2. Sample Application
3. Azure DevOps CLI Setup
4. Authentication
5. Publishing Packages
6. Downloading Packages
7. Versioning Best Practices
8. Practice Labs
9. Troubleshooting
10. Known Issues

---

## 1. Create the Project

```bash
git clone https://github.com/atulkamble/Azure-Artifacts-Practice.git
cd Azure-Artifacts-Practice
```

---

## 2. Python Application

File:

```text
package/app.py
```

Code:

```python
def hello() -> None:
    print("Hello from the Azure Artifacts package")


if __name__ == "__main__":
    hello()
```

Run:

```bash
python package/app.py
```

Expected output:

```text
Hello from the Azure Artifacts package
```

---

## 3. Configuration File

File:

```text
package/config.json
```

Code:

```json
{
  "application": "cloudnautic-app",
  "environment": "training",
  "version": "1.0.0"
}
```

---

## 4. Deployment Script

File:

```text
package/deploy.sh
```

Code:

```bash
#!/bin/bash

echo "Starting application deployment"
echo "Package version: 1.0.0"
python app.py
echo "Deployment completed"
```

Give execute permission:

```bash
chmod +x package/deploy.sh
```

---

## 5. `.artifactignore`

File:

```text
.artifactignore
```

Code:

```gitignore
.git
.gitignore
azure-pipelines.yml
README.md
*.log
__pycache__/
.env
```

`.artifactignore` controls files excluded when publishing Universal Packages and pipeline artifacts.

---

# 6. Install Azure DevOps CLI Extension

Check Azure CLI:

```bash
az version
```

Install the Azure DevOps extension:

```bash
az extension add --name azure-devops
```

Update it when already installed:

```bash
az extension update --name azure-devops
```

Sign in:

```bash
az login
```

Sign in to Azure DevOps:
```
az devops login
```
if want to logout existing 

check details 
```
az version
az account show
env | grep AZURE_DEVOPS
```
```
az devops configure --list
```
and 
```
az config get extension.use_dynamic_install
```
```
az devops logout
unset AZURE_DEVOPS_EXT_PAT
unset AZURE_DEVOPS_EXT_ARTIFACTTOOL_PATVAR
rm -f ~/.azure/azuredevops/cli/config
```

## set new
```
export AZURE_DEVOPS_EXT_PAT="PASTE_NEW_PAT_HERE"
```

Create and Update Token form Azure DevOps Settings (PAT)
Add Token: 

Configure the default organization:

```bash
az devops configure --defaults organization=https://dev.azure.com/cloudnautic
```

Configure the default project:

```bash
az devops configure --defaults project=project

https://dev.azure.com/cloudnautic/project

az devops configure \
  --defaults project=artifacts-project
```








33 Verify if OAT is set
```
 if [ -n "$AZURE_DEVOPS_EXT_PAT" ]; then
  echo "PAT is configured"
else
  echo "PAT is missing"
fi
```
---

# 13. Publish Universal Package Using CLI

## 13.1 Variables

```bash
ORGANIZATION="https://dev.azure.com/cloudnautic"
PROJECT="project"
FEED="newfeed"
PACKAGE_NAME="cloudnautic-tools"
PACKAGE_VERSION="1.0.9"
PACKAGE_PATH="./package"
```

## Verify 
```
echo "$ORGANIZATION"
echo "$PROJECT"
echo "$FEED"
echo "$PACKAGE_NAME"
echo "$PACKAGE_VERSION"
echo "$PACKAGE_PATH"
```

## show project 
```
az devops project show \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project"
```

## Check whether newfeed exists 
```
az devops invoke \
  --organization "$ORGANIZATION" \
  --area packaging \
  --resource feeds \
  --route-parameters project="$PROJECT" \
  --api-version "7.1" \
  --query "value[?name=='$FEED'].{Name:name,ID:id,Project:project.name}" \
  --output table
```
## Test feed access

> `az artifacts feed show` is not a valid Azure DevOps CLI command. Use `az devops invoke` instead.

```bash
az devops invoke \
  --organization "$ORGANIZATION" \
  --area packaging \
  --resource feeds \
  --route-parameters project="$PROJECT" \
  --api-version "7.1" \
  --query "value[?name=='$FEED'].{Name:name,ID:id,Project:project.name}" \
  --output table
```
---

## verify the feed using the Azure DevOps REST API
```
az devops invoke \
  --organization "https://dev.azure.com/cloudnautic" \
  --area packaging \
  --resource feeds \
  --route-parameters project="project" \
  --api-version "7.1-preview" \
  --output json
```

## 13.2 Publish to Project-Scoped Feed

```bash

az artifacts universal publish \
  --organization "$ORGANIZATION" \
  --project "$PROJECT" \
  --scope project \
  --feed "$FEED" \
  --name "$PACKAGE_NAME" \
  --version "$PACKAGE_VERSION" \
  --description "Cloudnautic training package" \
  --path "$PACKAGE_PATH"
```

Universal Package names must follow Azure Artifacts naming rules and should be lowercase.

---

## 13.3 Publish to Organization-Scoped Feed

```bash
az artifacts universal publish \
  --organization "$ORGANIZATION" \
  --scope organization \
  --feed "$FEED" \
  --name "$PACKAGE_NAME" \
  --version "$PACKAGE_VERSION" \
  --description "Cloudnautic organization package" \
  --path "$PACKAGE_PATH"
```

---

## 13.4 Expected Result

```text
Package Name: cloudnautic-tools
Version: 1.0.0
Feed: newfeed
```

Open:

```text
Azure DevOps
    -> Artifacts
    -> newfeed
    -> cloudnautic-tools
```

---

# 14. Download Universal Package Using CLI

Create a download directory:

```bash
mkdir downloaded-package
```

Download:

```bash
az artifacts universal download \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project" \
  --scope project \
  --feed "newfeed" \
  --name "cloudnautic-tools" \
  --version "1.0.0" \
  --path "./downloaded-package"
```

Verify:

```bash
tree downloaded-package
```

Run the application:

```bash
cd downloaded-package
python3 app.py

chmod +x deploy.sh
./deploy.sh

cd ..
```

---

Practice Azure Artifacts Universal Packages by updating the files, changing the version, publishing a new package version, downloading it, and running it.

## Version practice plan

Use versions like this:

```text
1.0.0  Initial package
1.0.1  Message update
1.0.2  Configuration update
1.1.0  New feature
2.0.0  Major breaking change
```

Version format:

```text
MAJOR.MINOR.PATCH
```

* **PATCH:** Small fix: `1.0.0 → 1.0.1`
* **MINOR:** New feature: `1.0.1 → 1.1.0`
* **MAJOR:** Major change: `1.1.0 → 2.0.0`

Universal Package versions are immutable. Never publish the same version twice.

---

# Practice 1: Publish version 1.0.4

Your `1.0.3` package already exists. Update the files to `1.0.4`.

## 1. Update `app.py`

```bash
cat > package/app.py <<'EOF'
def hello() -> None:
    print("Hello from Azure Artifacts Universal Package")
    print("Application version: 1.0.4")


if __name__ == "__main__":
    hello()
EOF
```

## 2. Update `config.json`

```bash
cat > package/config.json <<'EOF'
{
  "application": "cloudnautic-app",
  "environment": "training",
  "version": "1.0.4"
}
EOF
```

## 3. Update `deploy.sh`

```bash
cat > package/deploy.sh <<'EOF'
#!/bin/bash

echo "Starting application deployment"
echo "Package version: 1.0.4"
python3 app.py
echo "Deployment completed"
EOF
```

Make it executable:

```bash
chmod +x package/deploy.sh
```

## 4. Test locally

```bash
cd package
./deploy.sh
cd ..
```

Expected output:

```text
Starting application deployment
Package version: 1.0.4
Hello from Azure Artifacts Universal Package
Application version: 1.0.4
Deployment completed
```

## 5. Publish version 1.0.4

```bash
az artifacts universal publish \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project" \
  --scope project \
  --feed "newfeed" \
  --name "cloudnautic-tools" \
  --version "1.0.4" \
  --description "Cloudnautic training package version 1.0.4" \
  --path "./package"
```

---

# Practice 2: Download and run version 1.0.4

## 1. Create download directory

```bash
rm -rf downloaded-package
mkdir downloaded-package
```

## 2. Download the package

```bash
az artifacts universal download \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project" \
  --scope project \
  --feed "newfeed" \
  --name "cloudnautic-tools" \
  --version "1.0.4" \
  --path "./downloaded-package"
```

## 3. Check downloaded files

```bash
tree downloaded-package
```

Expected:

```text
downloaded-package
├── app.py
├── config.json
└── deploy.sh
```

## 4. Run downloaded package

```bash
cd downloaded-package
chmod +x deploy.sh
./deploy.sh
cd ..
```

---

# Practice 3: Create version 1.0.5

Make a small patch update.

## Update `app.py`

```bash
cat > package/app.py <<'EOF'
def hello() -> None:
    print("Welcome to Cloudnautic Azure Artifacts Training")
    print("Application version: 1.0.5")


if __name__ == "__main__":
    hello()
EOF
```

## Update version in `config.json`

```bash
cat > package/config.json <<'EOF'
{
  "application": "cloudnautic-app",
  "environment": "training",
  "version": "1.0.5"
}
EOF
```

## Update `deploy.sh`

```bash
cat > package/deploy.sh <<'EOF'
#!/bin/bash

echo "Starting application deployment"
echo "Package version: 1.0.5"
python3 app.py
echo "Deployment completed successfully"
EOF

chmod +x package/deploy.sh
```

## Test

```bash
cd package
./deploy.sh
cd ..
```

## Publish version 1.0.5

```bash
az artifacts universal publish \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project" \
  --scope project \
  --feed "newfeed" \
  --name "cloudnautic-tools" \
  --version "1.0.5" \
  --description "Updated welcome message and deployment output" \
  --path "./package"
```

---

# Practice 4: Add a feature in version 1.1.0

Add a health-check script.

```bash
cat > package/health-check.sh <<'EOF'
#!/bin/bash

echo "Checking application health..."
python3 app.py

if [ $? -eq 0 ]; then
  echo "Application status: Healthy"
else
  echo "Application status: Unhealthy"
  exit 1
fi
EOF

chmod +x package/health-check.sh
```

Update `config.json`:

```bash
cat > package/config.json <<'EOF'
{
  "application": "cloudnautic-app",
  "environment": "training",
  "version": "1.1.0",
  "healthCheckEnabled": true
}
EOF
```

Update `app.py`:

```bash
cat > package/app.py <<'EOF'
def hello() -> None:
    print("Welcome to Cloudnautic Azure Artifacts Training")
    print("Application version: 1.1.0")
    print("Health-check feature is enabled")


if __name__ == "__main__":
    hello()
EOF
```

Update `deploy.sh`:

```bash
cat > package/deploy.sh <<'EOF'
#!/bin/bash

echo "Starting application deployment"
echo "Package version: 1.1.0"

python3 app.py
./health-check.sh

echo "Deployment completed successfully"
EOF

chmod +x package/deploy.sh
```

Test:

```bash
cd package
./deploy.sh
cd ..
```

Publish:

```bash
az artifacts universal publish \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project" \
  --scope project \
  --feed "newfeed" \
  --name "cloudnautic-tools" \
  --version "1.1.0" \
  --description "Added application health-check feature" \
  --path "./package"
```

---

# Download a specific version

Download `1.0.4`:

```bash
az artifacts universal download \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project" \
  --scope project \
  --feed "newfeed" \
  --name "cloudnautic-tools" \
  --version "1.0.4" \
  --path "./downloads/1.0.4"
```

Download `1.0.5`:

```bash
az artifacts universal download \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project" \
  --scope project \
  --feed "newfeed" \
  --name "cloudnautic-tools" \
  --version "1.0.5" \
  --path "./downloads/1.0.5"
```

Download `1.1.0`:

```bash
az artifacts universal download \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project" \
  --scope project \
  --feed "newfeed" \
  --name "cloudnautic-tools" \
  --version "1.1.0" \
  --path "./downloads/1.1.0"
```

Compare versions:

```bash
tree downloads
```

---

# Recommended practice cycle

```text
1. Modify package files
2. Update version inside config.json
3. Update version inside deploy.sh
4. Test locally
5. Publish with a new package version
6. Download the package
7. Run the downloaded package
8. Compare old and new versions
```

## Compact publish variables

```bash
ORGANIZATION="https://dev.azure.com/cloudnautic"
PROJECT="project"
FEED="newfeed"
PACKAGE_NAME="cloudnautic-tools"
PACKAGE_VERSION="1.1.1"
PACKAGE_PATH="./package"
```

Publish:

```bash
az artifacts universal publish \
  --organization "$ORGANIZATION" \
  --project "$PROJECT" \
  --scope project \
  --feed "$FEED" \
  --name "$PACKAGE_NAME" \
  --version "$PACKAGE_VERSION" \
  --description "Cloudnautic package $PACKAGE_VERSION" \
  --path "$PACKAGE_PATH"
```

For each new practice, change only:

```bash
PACKAGE_VERSION="1.1.2"
```

and update the version inside `config.json`, `app.py`, and `deploy.sh`.


---

# Appendix: Verified ArtifactTool Workflow

During testing, the Azure CLI wrapper command:

```bash
az artifacts universal publish ...
```

failed with:

```text
TF400813: The user '<GUID>' is not authorized to access this resource.
```

The PAT, project, feed, and permissions were valid. Directly invoking ArtifactTool successfully published and downloaded the package.

## 1. Set variables

Run these commands from the repository root:

```bash
cd Azure-Artifacts-Practice

ORGANIZATION="https://dev.azure.com/cloudnautic"
PROJECT="project"
FEED="newfeed"
PACKAGE_NAME="cloudnautic-tools"
PACKAGE_VERSION="1.0.9"
PACKAGE_PATH="./package"
DOWNLOAD_PATH="./downloaded-package"
```

## 2. Prepare the PAT for ArtifactTool

```bash
export PATVAR="$AZURE_DEVOPS_EXT_PAT"

echo "PAT length: ${#PATVAR}"
```

Expected in this environment:

```text
PAT length: 84
```

Do not print the PAT value itself.

## 3. Locate ArtifactTool

```bash
ARTIFACT_TOOL=$(find ~/.azure/azuredevops/cli/tools/artifacttool \
  -type f \
  -name artifacttool \
  | head -1)

echo "$ARTIFACT_TOOL"
```

Example path:

```text
/Users/atul/.azure/azuredevops/cli/tools/artifacttool/ArtifactTool_osx-x64_0.2.565/artifacttool
```

## 4. Publish version 1.0.9

```bash
"$ARTIFACT_TOOL" \
  universal publish \
  --service "$ORGANIZATION" \
  --patvar PATVAR \
  --feed "$FEED" \
  --package-name "$PACKAGE_NAME" \
  --package-version "$PACKAGE_VERSION" \
  --path "$PACKAGE_PATH" \
  --project "$PROJECT" \
  --description "Cloudnautic training package $PACKAGE_VERSION"
```

Verified success output included:

```text
Package does not yet exist
Processed 4 files from ./package successfully.
Content pushed
Added package
Version: 1.0.9
Success
```

## 5. Download version 1.0.9

```bash
rm -rf "$DOWNLOAD_PATH"
mkdir -p "$DOWNLOAD_PATH"

"$ARTIFACT_TOOL" \
  universal download \
  --service "$ORGANIZATION" \
  --patvar PATVAR \
  --feed "$FEED" \
  --package-name "$PACKAGE_NAME" \
  --package-version "$PACKAGE_VERSION" \
  --path "$DOWNLOAD_PATH" \
  --project "$PROJECT"
```

Verified success output included:

```text
Obtained package metadata
Download completed.
Version: 1.0.9
Success
```

## 6. Verify the downloaded files

```bash
tree downloaded-package
```

Expected:

```text
downloaded-package
├── app.py
├── config.json
└── deploy.sh
```

## 7. Run the downloaded package

```bash
cd downloaded-package

python3 app.py

chmod +x deploy.sh
./deploy.sh

cd ..
```

The `chmod +x deploy.sh` command is required because the executable permission may not be preserved after package download.

## 8. Publish the next version correctly

Before publishing, return to the repository root. Otherwise, `PACKAGE_PATH="./package"` points to a non-existent directory.

```bash
cd ~/path/to/Azure-Artifacts-Practice
```

Confirm the package path:

```bash
pwd
ls -la "$PACKAGE_PATH"
```

Update the package files to version `1.0.10`, then set:

```bash
PACKAGE_VERSION="1.0.10"
```

Publish:

```bash
"$ARTIFACT_TOOL" \
  universal publish \
  --service "$ORGANIZATION" \
  --patvar PATVAR \
  --feed "$FEED" \
  --package-name "$PACKAGE_NAME" \
  --package-version "$PACKAGE_VERSION" \
  --path "$PACKAGE_PATH" \
  --project "$PROJECT" \
  --description "Cloudnautic training package $PACKAGE_VERSION"
```

## Why `The path provided is invalid` occurred

The command was run from:

```text
Azure-Artifacts-Practice/downloaded-package
```

while the variable still contained:

```bash
PACKAGE_PATH="./package"
```

From inside `downloaded-package`, `./package` does not exist.

Use either:

```bash
cd ..
```

before publishing, or set an absolute package path:

```bash
PACKAGE_PATH="$(cd package && pwd)"
```

The recommended approach is to run publish commands from the repository root.

## Immutable versions

Universal Package versions cannot be overwritten.

If version `1.0.9` already exists, publish a new version:

```bash
PACKAGE_VERSION="1.0.10"
```

## Security cleanup

After finishing:

```bash
unset PATVAR
unset AZURE_DEVOPS_EXT_PAT
unset AZURE_DEVOPS_EXT_ARTIFACTTOOL_PATVAR
```
---

# Recommended Troubleshooting Workflow

Follow these checks **in order** whenever publishing fails.

## 1. Verify Azure CLI

```bash
az version
az extension update --name azure-devops
```

## 2. Verify Azure DevOps Login

```bash
az account show
az devops configure --list
```

## 3. Verify PAT

```bash
echo ${#AZURE_DEVOPS_EXT_PAT}
```

## 4. Verify Feed Exists

```bash
az devops invoke   --organization "$ORGANIZATION"   --area packaging   --resource feeds   --route-parameters project="$PROJECT"   --api-version "7.1"
```

## 5. Verify Feed Access

```bash
curl -L -u ":$AZURE_DEVOPS_EXT_PAT" "https://feeds.dev.azure.com/cloudnautic/project/_apis/packaging/Feeds/<feed-id>?api-version=7.1-preview.1"
```

HTTP 200 confirms authentication and feed visibility.

## 6. If TF400813 Still Occurs

Use ArtifactTool directly:

```bash
export PATVAR="$AZURE_DEVOPS_EXT_PAT"

ARTIFACT_TOOL=$(find ~/.azure/azuredevops/cli/tools/artifacttool -type f -name artifacttool | head -1)

"$ARTIFACT_TOOL" \
  universal publish \
  --service "$ORGANIZATION" \
  --patvar PATVAR \
  --feed "$FEED" \
  --package-name "$PACKAGE_NAME" \
  --package-version "$PACKAGE_VERSION" \
  --path "$PACKAGE_PATH" \
  --project "$PROJECT"
```

This workflow was verified successfully during testing.
