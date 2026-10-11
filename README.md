# CI/CD Pipeline Demo (GitHub Actions)

A complete, production-grade CI/CD pipeline template with automated build validation, unit testing, security scanning, container publishing (GitHub Container Registry), and automated deployment.

---

## 📁 Repository Structure

```
.
├── app.py                      # Flask REST application
├── requirements.txt            # Application and CI tool dependencies
├── test_app.py                 # Pytest test suite
├── Dockerfile                  # Container definition (python:3.12-slim & gunicorn)
├── .gitignore                  # Git ignore rules for Python & CI artifacts
├── README.md                   # Project documentation & configuration guide
└── .github/
    └── workflows/
        └── ci-cd.yml           # GitHub Actions workflow
```

---

## 🔄 Pipeline Workflow Stages

| Stage | Trigger | Description |
|---|---|---|
| **Build Validation** | PR & Push | Compiles Python code to catch syntax issues (`compileall`). |
| **Automated Tests** | PR & Push | Runs `pytest` and exports JUnit XML test reports as pipeline artifacts. |
| **Security Scanning** | PR & Push | AST security analysis via `bandit` and dependency CVE audit via `pip-audit`. |
| **Docker Build & Push** | Push to `main` | Builds multi-platform container and publishes to GitHub Container Registry (`ghcr.io`). |
| **Deployment** | Push to `main` | Connects via SSH to target host, pulls latest container, and executes blue/green or rolling restart. |
| **Pipeline Report** | Always | Generates step summary report directly within GitHub Actions UI. |

---

## 💻 Local Development & Testing

### 1. Set Up Virtual Environment

```bash
# Create virtual environment
python -m venv .venv

# Activate environment
# On Windows (PowerShell):
.venv\Scripts\Activate.ps1
# On Linux / macOS:
source .venv/bin/activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

### 2. Run Tests & Security Scans

```bash
# Run unit tests
pytest -v

# Run Bandit security scan
bandit -r app.py

# Run pip-audit for dependency vulnerabilities
pip-audit
```

### 3. Run Locally

```bash
python app.py
```
Endpoints:
- Application: [http://localhost:5000/](http://localhost:5000/)
- Health Check: [http://localhost:5000/health](http://localhost:5000/health)

---

## 🔐 Required GitHub Secrets (For Deployment)

In your GitHub repository, navigate to **Settings → Secrets and variables → Actions → New repository secret**:

| Secret Name | Description | Example |
|---|---|---|
| `VPS_HOST` | Remote server IP address or hostname | `203.0.113.10` or `app.example.com` |
| `VPS_USER` | SSH username | `deployer` / `ubuntu` |
| `VPS_SSH_KEY` | Private SSH key (ed25519 or RSA) | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `VPS_KNOWN_HOSTS` | Output from `ssh-keyscan -H <YOUR_HOST>` | `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5...` |

---

## 🚀 Deployment Platforms

### Option A: Linux VPS (Default in `ci-cd.yml`)
The workflow connects via SSH, pulls `ghcr.io/<owner>/<repo>:<sha>`, and runs the container with Docker restart policies.

### Option B: Render
To deploy automatically to Render:
1. Create a **Web Service** on Render connected to this repository (or pointing to your GHCR image).
2. Grab the **Deploy Hook URL** in Render settings.
3. Replace the `deploy` job with a simple webhook curl:
   ```yaml
   deploy-render:
     needs: docker-publish
     runs-on: ubuntu-latest
     steps:
       - name: Trigger Render Deploy Hook
         run: curl -X POST "${{ secrets.RENDER_DEPLOY_HOOK_URL }}"
   ```

### Option C: AWS EC2 / ECS / App Runner
- For **AWS EC2**: Use AWS SSM Agent (`aws-actions/aws-ssm-send-command`) or SSH keys in GitHub Secrets.
- For **AWS App Runner / ECS**: Use `aws-actions/amazon-ecr-login` and update the service definition.

### Option D: Docker Only (No deployment)
If you only need testing, security scans, and published images on GHCR, remove or comment out the `deploy` job in `.github/workflows/ci-cd.yml`.
