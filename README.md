# CI/CD with Helm Deployment Lab

A Flask service, Docker image, Helm chart, and GitHub Actions pipeline. Pushes to `main` run tests and chart validation, publish a commit-tagged image to GHCR, deploy it to a temporary kind cluster, and smoke-test the deployed version. Pull requests run tests and chart validation only.

## Run locally on Windows

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pytest -q
```

Build and run the container:

```powershell
docker build -t lab-app:0.1.0 .
docker run --rm -p 8080:8080 lab-app:0.1.0
```

In another terminal, check `http://localhost:8080/healthz`.

## Validate the Helm chart

```powershell
helm lint chart
helm template demo chart
helm template demo chart --set replicaCount=5
helm template demo chart -f chart/values-dev.yaml
helm template demo chart -f chart/values-prod.yaml
```

For the manual kind deployment in the lab guide, build the image first, create a cluster named `lab`, load `lab-app:0.1.0` into it, then install with `--set image.pullPolicy=Never`.

## Enable GitHub Actions

1. Create a GitHub repository named `cicd-helm-lab` under `Raghu0703` and push this folder to its `main` branch.
2. In repository Settings, ensure Actions can use the `GITHUB_TOKEN` with package write access if your organization or repository policy restricts token permissions.
3. The workflow publishes `ghcr.io/raghu0703/cicd-helm-lab:<7-character-commit-sha>`. GHCR packages are private by default. After the first image is published, change the package visibility to Public so the temporary kind cluster can pull it, then rerun the workflow. Subsequent deployments use `helm upgrade --install --atomic --wait` and roll back automatically if rollout readiness fails.

A push requires GitHub authentication and cannot be completed from this workspace without your GitHub session. The workflow derives the image owner from the repository, lowercases it for GHCR, and does not require a personal access token.
