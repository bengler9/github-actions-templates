# Feature Comparison: GitLab CI vs GitHub Actions Templates

This document compares the features available in GitLab CI templates vs GitHub Actions templates to identify gaps.

## Feature Parity Status

### ✅ Fully Implemented in Both

| Feature                | GitLab CI                         | GitHub Actions                          | Status |
| ---------------------- | --------------------------------- | --------------------------------------- | ------ |
| **Quality Checks**     |
| Flake8 linting         | `.default-flake8`                 | `quality-checks.yml` (flake8)           | ✅     |
| Mypy type checking     | `.default-mypy`                   | `quality-checks.yml` (mypy)             | ✅     |
| CloudFormation linting | N/A                               | `quality-checks.yml` (cfn-lint)         | ✅     |
| Terraform linting      | `.default-tf_lint`                | `terraform-workflow.yml` (fmt/validate) | ✅     |
| **Security Scans**     |
| Bandit security scan   | `.default-bandit`                 | `security-scans.yml` (bandit)           | ✅     |
| Checkov IaC scan       | `.default-checkov`                | `security-scans.yml` (checkov)          | ✅     |
| Radon complexity       | `.default-radon`                  | `security-scans.yml` (radon)            | ✅     |
| **Testing**            |
| Python tests (pytest)  | `.default-test`                   | `python-test.yml`                       | ✅     |
| **Documentation**      |
| Verify docs            | `.default-verify_docs`            | `verify-docs.yml`                       | ✅     |
| **Code Analysis**      |
| Code archive to S3     | `.default-code_analyzer`          | `code-analyzer.yml`                     | ✅     |
| **Deployment**         |
| Lambda deployment      | `.default-deploy`                 | `lambda-deploy.yml`                     | ✅     |
| Terraform plan/apply   | `.default-plan`, `.default-apply` | `terraform-workflow.yml`                | ✅     |

### ❌ Missing in GitHub Actions

| Feature                       | GitLab CI                        | GitHub Actions | Priority    | Notes                              |
| ----------------------------- | -------------------------------- | -------------- | ----------- | ---------------------------------- |
| **Versioning & Releases**     |
| Auto-tagging                  | `.default-auto_tag`              | ❌ Missing     | 🔴 **HIGH** | Semantic versioning, creates tags  |
| Auto-release                  | `.default-auto_tag`              | ❌ Missing     | 🔴 **HIGH** | Creates GitHub releases with notes |
| **Infrastructure Management** |
| Detect drift                  | `.default-detect_drift`          | ❌ Missing     | 🟡 Medium   | Terraform state drift detection    |
| Verify infrastructure         | `.default-verify_infrastructure` | ❌ Missing     | 🟡 Medium   | Post-deploy verification           |
| **Branch Management**         |
| Reset env branches            | `.default-reset_env_branches`    | ❌ Missing     | 🟡 Medium   | Reset env branches to match main   |
| **Build & Push**              |
| Build and push Docker         | `.default-build_and_push`        | ❌ Missing     | 🟡 Medium   | Docker image build/push            |
| **Vue.js Specific**           |
| Vue lint                      | `.default-vue_lint`              | ❌ Missing     | 🟢 Low      | Vue.js specific                    |
| Vue test                      | `.default-vue_test`              | ❌ Missing     | 🟢 Low      | Vue.js specific                    |
| Vue build                     | `.default-vue_build`             | ❌ Missing     | 🟢 Low      | Vue.js specific                    |
| Vue deploy                    | `.default-vue_deploy`            | ❌ Missing     | 🟢 Low      | Vue.js specific                    |

## Detailed Feature Gaps

### 1. Auto-Tagging & Auto-Release (🔴 HIGH PRIORITY)

**GitLab Implementation:**

- Semantic versioning (v1.2.3)
- Branch-based versioning:
  - `main` → patch bump (v1.2.3 → v1.2.4)
  - `env/dev` → minor bump with beta suffix (v1.2.3 → v1.3.0-beta)
  - `feature/*` → minor bump with alpha suffix
  - `hotfix/*` → patch bump with hotfix suffix
- Commit message overrides: "INCREASE MINOR VERSION", "INCREASE MAJOR VERSION"
- Creates GitLab releases with MR descriptions as release notes

**GitHub Equivalent Needed:**

- Create GitHub Actions workflow: `auto-tag-release.yml`
- Use GitHub API to create releases
- Extract PR descriptions for release notes
- Support semantic versioning with same logic

**Impact:** High - This is a core feature for version management and release tracking.

### 2. Branch Reset (🟡 MEDIUM PRIORITY)

**GitLab Implementation:**

- `.default-reset_env_branches` job
- Resets environment branches (env/dev, env/qat, env/stg) to match main branch
- Runs automatically after merges to main (post-merge mode)
- Can also run on schedule (weekly) or manually
- Checks if branches have diverged from main
- Deletes and recreates branches from main to keep them in sync
- Uses GitLab API or Git commands

**GitHub Equivalent Needed:**

- Create `reset-env-branches.yml` workflow
- Use GitHub API to check branch divergence
- Delete and recreate environment branches from main
- Support multiple modes: post-merge, scheduled, manual
- Run on push to main (post-merge) or on schedule

**Impact:** Medium - Important for keeping environment branches in sync with main, but can be done manually if needed.

### 3. Infrastructure Drift Detection (🟡 MEDIUM PRIORITY)

**GitLab Implementation:**

- `.default-detect_drift` job
- Compares Terraform state with actual infrastructure
- Creates issues when drift detected

**GitHub Equivalent Needed:**

- Add drift detection job to `terraform-workflow.yml`
- Use GitHub Issues API to create issues
- Run on schedule or manual trigger

**Impact:** Medium - Useful for infrastructure codeitems but not critical for Lambda functions.

### 4. Infrastructure Verification (🟡 MEDIUM PRIORITY)

**GitLab Implementation:**

- `.default-verify_infrastructure` job
- Post-deployment verification checks
- Multiple variants for different infrastructure types

**GitHub Equivalent Needed:**

- Add verification step to `terraform-workflow.yml` or `lambda-deploy.yml`
- Customizable verification scripts

**Impact:** Medium - Good practice but can be added later.

### 5. Docker Build & Push (🟡 MEDIUM PRIORITY)

**GitLab Implementation:**

- `.default-build_and_push` job
- Builds Docker images
- Pushes to container registry

**GitHub Equivalent Needed:**

- Create `docker-build-push.yml` workflow
- Support ECR, Docker Hub, GitHub Container Registry

**Impact:** Medium - Needed for containerized applications but not for Lambda functions.

## Recommendation

### Option A: Integrate Now, Add Features Later (Recommended)

**Pros:**

- ✅ Get teams using GitHub Actions immediately
- ✅ Establish patterns and best practices
- ✅ Learn from real-world usage
- ✅ Can add features incrementally based on actual needs
- ✅ Current feature set covers 90% of use cases

**Cons:**

- ⚠️ Some codeitems will need to wait for auto-tag/release
- ⚠️ May need to add features later anyway

**Best for:** Teams that want to start using GitHub Actions now and can live without auto-tagging temporarily.

### Option B: Add Features First, Then Integrate

**Pros:**

- ✅ Feature parity from day one
- ✅ No need to go back and add features later
- ✅ More complete solution

**Cons:**

- ⚠️ Delays integration by 1-2 weeks
- ⚠️ Features may not be needed by all codeitems
- ⚠️ May over-engineer before understanding real needs

**Best for:** Teams that absolutely need auto-tagging/release before they can use GitHub Actions.

## Recommended Approach

**Start with Option A (Integrate Now):**

1. **Phase 1: Integrate Current Templates** (Now)

   - Use existing templates for new codeitems
   - Establish patterns and workflows
   - Collect feedback from teams

2. **Phase 2: Add High-Priority Features** (1-2 weeks)

   - Implement `auto-tag-release.yml` workflow
   - Add to existing codeitems that need it
   - Document usage

3. **Phase 3: Add Medium-Priority Features** (As needed)
   - Branch reset workflow (for keeping env branches in sync)
   - Infrastructure drift detection (for Terraform codeitems)
   - Docker build/push (for containerized apps)
   - Infrastructure verification (as needed)

**Rationale:**

- Current templates cover the core CI/CD needs (quality, security, testing, deployment)
- Auto-tagging/release is nice-to-have but not blocking
- Can add features incrementally based on actual demand
- Faster time-to-value for teams

## Implementation Priority

1. **Now:** Integrate current templates into codeitems
2. **Week 1-2:** Add auto-tag-release workflow
3. **As needed:** Add branch reset, drift detection, Docker build, etc.

## Notes

- Most Lambda codeitems don't need Docker build/push
- Most codeitems don't need infrastructure drift detection
- Auto-tagging/release is the most commonly requested missing feature
- Branch reset is important for maintaining environment branches but can be done manually
- Can always add features later without breaking existing workflows
