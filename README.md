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
