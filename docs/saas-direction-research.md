# SaaS Direction Research: AI-Driven Deployment Tool

Date: 2026-07-22

## Question

What should this project become if the goal is a workable SaaS, given the current codebase already analyzes GitHub repositories, generates deployment files, pushes to GitHub, monitors GitHub Actions, and can deploy images to local Minikube?

## Current Project Reality

The current project is strongest as an AI DevOps preparation engine:

- Clone a submitted GitHub repository.
- Detect language, framework, services, and databases.
- Generate `Dockerfile`, `.dockerignore`, `docker-compose.yml`, `.env.example`, and GitHub Actions workflow files.
- Scan for secrets before pushing generated files.
- Push generated files back to GitHub when the user has connected GitHub through Clerk.
- Monitor GitHub Actions workflow runs and logs.
- After GitHub Actions succeeds, deploy the resulting GHCR image to local Minikube.

The Minikube part is useful for demos and local developer testing, but it is not a SaaS hosting model. A cloud SaaS cannot assume its customers have Docker Desktop, `kubectl`, and Minikube running on the SaaS server.

## What Existing Platforms Already Do

### Render

Render already lets users link a GitHub/GitLab/Bitbucket repository and automatically rebuild/redeploy when changes land on a configured branch. Render also supports Docker services by either building from a Dockerfile in the repository or running a prebuilt image from a registry such as GitHub Container Registry (GHCR).

Sources:

- https://render.com/docs/deploys
- https://render.com/docs/docker
- https://render.com/docs/deploying-an-image

Implication: competing with Render as a general app host is hard. It already owns build, run, logs, auto-deploy, zero-downtime deploys, and public URLs.

### Railway

Railway services can deploy from GitHub, a local repository, or a Docker image. If a Dockerfile is present in the source repository, Railway can use it to build the service image. Railway describes a deployment as building and packaging the service, then starting it with a detected or configured start command.

Sources:

- https://docs.railway.com/services
- https://docs.railway.com/deployments/reference
- https://docs.railway.com/quick-start

Implication: Railway already solves "connect repo and run app." This project should not try to win only by being another generic deploy button.

### Koyeb

Koyeb supports deployment from GitHub, a local project directory, or prebuilt Docker images. It supports buildpacks, Dockerfile builds, private registries including GHCR, environment variables, build logs, and service types such as web services and workers.

Sources:

- https://www.koyeb.com/docs/build-and-deploy
- https://www.koyeb.com/docs/reference/services

Implication: Koyeb already covers the exact source-to-container and image-to-service deployment paths this project currently approximates locally.

### Vercel

Vercel is especially strong for frontend and framework deployments. It supports deployments through Git, CLI, Deploy Hooks, and REST API. Pushing to a connected Git repository can automatically trigger builds and deployments. Vercel also provides Preview and Production environments, deployment URLs, build customization, and environment variables.

Sources:

- https://vercel.com/docs/deployments/overview
- https://vercel.com/docs/builds
- https://vercel.com/docs/rest-api
- https://vercel.com/docs/cli/deploy

Implication: for frontend-heavy repositories, the SaaS should recommend or integrate with Vercel instead of forcing Docker/Minikube.

### Fly.io

Fly.io deploys applications as images onto Fly Machines. It supports Dockerfile, buildpack, and image-based deployment paths.

Sources:

- https://fly.io/docs/reference/builders/
- https://fly.io/docs/launch/deploy/

Implication: Fly is a realistic target integration for container-native backends, but it requires provider-specific configuration and account credentials.

### Qovery

Qovery is closest to a managed Kubernetes deployment platform. It supports deployment from Git repositories, container registries, and Helm charts. Its documented deployment flow includes pulling code, building an image, pushing to a registry, creating Kubernetes resources, configuring networking, setting domains/TLS, and tracking deployment status/logs.

Sources:

- https://www.qovery.com/docs/getting-started/guides/getting-started/deploy-your-first-application
- https://www.qovery.com/docs/configuration/deployment/overview
- https://www.qovery.com/docs/configuration/application

Implication: if this project wants "real Kubernetes SaaS deployment," Qovery is the shape of the mature product. Rebuilding that directly is a large infrastructure product.

### GitHub Actions and GHCR

GitHub Actions workflow runs can be listed, inspected, re-run, canceled, and have logs retrieved through the GitHub REST API. The current project already uses this model to detect whether a workflow is still queued/running or completed successfully/failed.

Source:

- https://docs.github.com/en/rest/actions/workflow-runs

GitHub Actions can authenticate to GitHub Container Registry with repository permissions and push Docker images to `ghcr.io`.

Sources:

- https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry
- https://docs.github.com/en/actions/publishing-packages/publishing-docker-images

Implication: the current GitHub Actions/GHCR architecture is valid. The problem is not GHCR. The problem is what happens after the image exists.

## Strategic Gap

Existing platforms already do this:

```text
repo -> build -> deploy -> public URL -> logs
```

This project should focus on the part many student and junior-developer projects struggle with before reaching those platforms:

```text
repo -> understand stack -> detect deploy blockers -> generate correct config -> explain choices -> open PR -> monitor build failures -> suggest fixes
```

That is the more defensible product.

## Recommended SaaS Positioning

Build an **AI Deployment Readiness and Fixing SaaS**, not a cloud hosting platform first.

Short version:

```text
Paste GitHub repo.
The SaaS tells you what it is, whether it is deployable, what is missing, and creates the deployment files or PR needed to deploy it.
```

The product should help users get from "my repo is messy" to "this repo is ready for Vercel/Render/Railway/Koyeb/Fly."

## Recommended MVP

### MVP Scope

1. GitHub connection and repository selection.
2. Repo analysis:
   - language/framework
   - frontend/backend split
   - monorepo detection
   - database detection
   - env var detection
   - existing Dockerfile/workflow detection
3. Deployment readiness report:
   - "Can deploy as-is"
   - "Needs Dockerfile"
   - "Needs start command"
   - "Needs env vars"
   - "Needs database"
   - "Frontend should go to Vercel"
   - "Backend should go to Render/Railway/Koyeb"
4. Generate deployment files:
   - `Dockerfile`
   - `.dockerignore`
   - `.env.example`
   - GitHub Actions workflow
   - optional `render.yaml`, `railway.json`, `vercel.json`, `fly.toml`, or Koyeb instructions
5. Open a PR instead of directly committing to `main`.
6. Monitor GitHub Actions.
7. Explain build failures and propose patch actions.

### MVP Non-Goals

Do not try to host arbitrary user apps on this SaaS in v1.

Avoid these in v1:

- Running untrusted user code on your own servers.
- Managing customer containers long-term.
- Multi-tenant runtime isolation.
- Public domains and TLS for user apps.
- Billing for compute usage.
- Kubernetes cluster management.
- Automatic database provisioning.

These are real platform problems and would pull the project away from its current strengths.

## Recommended Product Flow

```text
User connects GitHub
-> User selects/pastes repo
-> App clones and analyzes repo
-> App shows Deployment Readiness Report
-> User chooses target: Vercel / Render / Railway / Koyeb / Docker / Local Minikube
-> App generates target-specific files
-> App opens PR with generated files
-> GitHub Actions validates build
-> App monitors workflow result
-> If failed, app diagnoses logs and suggests PR patch
-> If passed, app gives exact next deploy button/instructions or calls provider API in later versions
```

## What To Do With Minikube

Keep Minikube, but reposition it:

```text
Local Kubernetes Demo Mode
```

It should not be the primary SaaS deployment story.

Use it for:

- classroom demo
- local proof that generated Docker image can run
- Kubernetes learning mode
- capstone/expo visual workflow

Do not present it as cloud hosting.

## Future Versions

### V1: Deployment Readiness SaaS

Goal: prepare repos for deployment and open PRs.

Core deliverable: accurate report + generated files + GitHub PR.

### V2: Provider-Assisted Deployment

Goal: integrate with providers rather than hosting apps yourself.

Possible integrations:

- Vercel for frontend projects.
- Render/Koyeb/Railway/Fly for backends and Docker images.
- MongoDB Atlas/Supabase/Neon guidance for databases.

The SaaS can either:

- generate provider config files, or
- call provider APIs after the user connects provider credentials.

### V3: Managed Runtime

Goal: actually run user apps.

This requires:

- sandboxing
- container orchestration
- runtime secrets
- metrics/log retention
- limits and billing
- domain/TLS management
- abuse controls
- cleanup jobs

This is a separate infrastructure company, not just a feature.

## Immediate Engineering Recommendations

1. Rename the product mental model from "Auto Deployment Platform" to "AI Deployment Readiness and Fix Assistant."
2. Add a first-class `DeploymentReadinessReport` backend schema.
3. Stop direct pushes to `main`; create branches and PRs.
4. Separate "generation" from "deployment":
   - `analyze`
   - `generate`
   - `open_pr`
   - `monitor_ci`
   - `deploy_local_minikube`
   - later: `deploy_provider`
5. Add target-specific recommendations:
   - frontend/static -> Vercel
   - Docker backend -> Render/Koyeb/Railway/Fly
   - Kubernetes learning/demo -> Minikube
6. Treat database/env-var detection as a core feature, not a side feature.
7. Make Minikube an optional local demo mode.
8. Remove committed `__pycache__` files and ensure `.env` files are not tracked.

## Final Recommendation

The workable SaaS is not "we run any GitHub repo for free."

The workable SaaS is:

```text
An AI DevOps assistant that turns an undeployable GitHub repo into a deployable repo, then guides or automates deployment to existing platforms.
```

This uses the strongest parts of the current codebase and avoids competing directly with mature hosting platforms before the product is ready.
