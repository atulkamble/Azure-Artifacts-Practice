## 11.2 Create the Project

```bash
git clone https://github.com/atulkamble/azure-artifacts-practice.git
cd azure-artifacts-practice
```

---

## 11.3 Python Application

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

## 11.4 Configuration File

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

## 11.5 Deployment Script

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

## 11.6 `.artifactignore`

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

# 12. Install Azure DevOps CLI Extension

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

---

# 13. Publish Universal Package Using CLI

## 13.1 Variables

```bash
ORGANIZATION="https://dev.azure.com/cloudnautic"
PROJECT="project"
FEED="cloudnautic-feed"
PACKAGE_NAME="cloudnautic-tools"
PACKAGE_VERSION="1.0.4"
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

## test feed access 
```
az artifacts feed show \
  --organization "https://dev.azure.com/cloudnautic" \
  --project "project" \
  --feed "cloudnautic-feed"
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
Feed: cloudnautic-feed
```

Open:

```text
Azure DevOps
    -> Artifacts
    -> cloudnautic-feed
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
  --project "artifacts-project" \
  --scope project \
  --feed "cloudnautic-feed" \
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
python downloaded-package/app.py
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
  --feed "cloudnautic-feed" \
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
  --feed "cloudnautic-feed" \
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
  --feed "cloudnautic-feed" \
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
  --feed "cloudnautic-feed" \
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
  --feed "cloudnautic-feed" \
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
  --feed "cloudnautic-feed" \
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
  --feed "cloudnautic-feed" \
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
FEED="cloudnautic-feed"
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
