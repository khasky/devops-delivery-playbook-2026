# DevOps Delivery Playbook

[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE) [![Web Reactions](https://api.webreactions.app/badge/github/khasky/devops-delivery-playbook.svg)](https://webreactions.app/?utm_source=github&utm_channel=repository&utm_medium=devops-delivery-playbook)

Practical DevOps delivery guide for CI/CD, environments, rollout safety, observability, and release workflows.

> *If I were defining DevOps defaults for a product team today, I would start with a boring rule: every change should earn trust in layers — branch, pull request, staging, canary, full rollout.*

---

## Table of Contents

- [DevOps Delivery Playbook](#devops-delivery-playbook)
  - [Table of Contents](#table-of-contents)
  - [Companion playbooks](#companion-playbooks)
  - [The defaults I'd reach for first](#the-defaults-id-reach-for-first)
  - [Branch and pull request flow](#branch-and-pull-request-flow)
    - [The baseline I would publish](#the-baseline-i-would-publish)
    - [Why this matters](#why-this-matters)
    - [The operating rule](#the-operating-rule)
  - [CI test lanes](#ci-test-lanes)
    - [Lane 1: fast checks on feature-branch push](#lane-1-fast-checks-on-feature-branch-push)
    - [Lane 2: smoke tests on feature branches](#lane-2-smoke-tests-on-feature-branches)
    - [Lane 3: full suite on main](#lane-3-full-suite-on-main)
    - [Lane 4: on-demand full suite for risky branches](#lane-4-on-demand-full-suite-for-risky-branches)
  - [Staging validation](#staging-validation)
    - [What "tested on staging" should actually mean](#what-tested-on-staging-should-actually-mean)
    - [What I would require before production rollout](#what-i-would-require-before-production-rollout)
  - [Canary releases](#canary-releases)
    - [The default model](#the-default-model)
    - [What I would watch during a canary](#what-i-would-watch-during-a-canary)
    - [A practical canary sequence](#a-practical-canary-sequence)
  - [Feature flags and progressive delivery](#feature-flags-and-progressive-delivery)
    - [Why flags earn their place](#why-flags-earn-their-place)
    - [The lifecycle rule](#the-lifecycle-rule)
    - [One performance caveat](#one-performance-caveat)
  - [Rollback strategy](#rollback-strategy)
    - [Two rollback modes worth supporting](#two-rollback-modes-worth-supporting)
    - [What I would document in every deploy guide](#what-i-would-document-in-every-deploy-guide)
  - [Secret scanning](#secret-scanning)
    - [Why secret scanning belongs in the playbook](#why-secret-scanning-belongs-in-the-playbook)
    - [Baseline rule](#baseline-rule)
  - [Supply chain integrity](#supply-chain-integrity)
    - [Baseline rules](#baseline-rules)
    - [The trust rule](#the-trust-rule)
  - [A practical workflow model](#a-practical-workflow-model)
  - [Example GitHub Actions layout](#example-github-actions-layout)
  - [OIDC to the cloud, not long-lived secrets](#oidc-to-the-cloud-not-long-lived-secrets)
    - [The federation default](#the-federation-default)
    - [For secrets that must exist](#for-secrets-that-must-exist)
  - [Node.js test runners and monorepos](#nodejs-test-runners-and-monorepos)
  - [Infrastructure as code, minimally](#infrastructure-as-code-minimally)
    - [The provisioning baseline](#the-provisioning-baseline)
    - [Start small](#start-small)
  - [Things I would avoid](#things-i-would-avoid)
  - [References and inspiration](#references-and-inspiration)
    - [Official and high-signal references](#official-and-high-signal-references)
    - [Tooling references](#tooling-references)
    - [Similar or adjacent GitHub repositories](#similar-or-adjacent-github-repositories)
  - [License](#license)

---

## Companion playbooks

These repositories form one playbook suite:

- [AI-Assisted Engineering Playbook](https://github.com/khasky/ai-assisted-engineering-playbook) — agent workflows, guardrails, and quality control for AI-heavy teams
- [API Design Playbook](https://github.com/khasky/api-design-playbook) — versioning, pagination, idempotency, error contracts, and webhooks
- [Auth & Identity Playbook](https://github.com/khasky/auth-identity-playbook) — sessions, tokens, OAuth, and identity boundaries across the stack
- [Backend Architecture Playbook](https://github.com/khasky/backend-architecture-playbook) — APIs, boundaries, OpenAPI, persistence, and errors
- [Best of JavaScript](https://github.com/khasky/best-of-javascript) — curated JS/TS tooling and stack defaults
- [Caching Playbook](https://github.com/khasky/caching-playbook) — HTTP, CDN, and application caches; consistency and invalidation
- [Code Review Playbook](https://github.com/khasky/code-review-playbook) — PR quality, ownership, and review culture
- [CTO Playbook](https://github.com/khasky/cto-playbook) — org design, hiring, technical strategy, budgets, and due diligence
- **DevOps Delivery Playbook** — CI/CD, environments, rollout safety, and observability
- [Engineering Lead Playbook](https://github.com/khasky/engineering-lead-playbook) — standards, RFCs, and technical leadership habits
- [Frontend Architecture Playbook](https://github.com/khasky/frontend-architecture-playbook) — React structure, performance, and consuming API contracts
- [Git Collaboration Playbook](https://github.com/khasky/git-collaboration-playbook) — branching, stacked PRs, merge queues, and CI collaboration at scale
- [Marketing and SEO Playbook](https://github.com/khasky/marketing-and-seo-playbook) — growth, SEO, experimentation, and marketing surfaces
- [Messaging & Async Playbook](https://github.com/khasky/messaging-and-async-playbook) — queues, events, outbox, idempotent consumers, and retries
- [Monorepo Architecture Playbook](https://github.com/khasky/monorepo-architecture-playbook) — workspaces, package boundaries, and shared code at scale
- [Node.js Runtime & Performance Playbook](https://github.com/khasky/nodejs-runtime-performance-playbook) — event loop, streams, memory, and production Node performance
- [Observability Playbook](https://github.com/khasky/observability-playbook) — logs, traces, metrics, SLOs, and production visibility
- [React Cross-Platform Playbook](https://github.com/khasky/react-cross-platform-playbook) — shared React UI and logic across web and native with TypeScript
- [Software Design Playbook](https://github.com/khasky/software-design-playbook) — separation of concerns, composition, and module boundaries
- [State Management Playbook](https://github.com/khasky/state-management-playbook) — client state architecture, MobX, and choosing a state layer
- [Styling Architecture Playbook](https://github.com/khasky/styling-architecture-playbook) — type-safe styling, design tokens, and theming at scale
- [Sysadmin Operations Playbook](https://github.com/khasky/sysadmin-operations-playbook) — servers, backups, DNS, TLS, identity, and the self-hosted ops stack
- [Testing Strategy Playbook](https://github.com/khasky/testing-strategy-playbook) — unit, integration, contract, E2E, and CI-friendly test layers

---

## The defaults I'd reach for first

If I were setting release rules for a team today, I would usually start here:

- **Feature branch push:** lint, unit tests, and fast smoke coverage
- **Pull request:** reviewable diff, status checks, staging validation path
- **Main branch merge:** full test suite
- **Manual dispatch:** ability to run the full suite on a feature branch when risk is high
- **Secrets:** scan the repository for exposed keys and tokens on every meaningful path
- **Deploy:** stage first, then canary, then full rollout
- **Rollback:** automatic where signals are clear, manual where judgment is needed
- **Visibility:** error rate, latency, throughput, and business metrics visible during rollout

The goal is not "more pipelines" The goal is progressive confidence.

---

## Branch and pull request flow

A healthy DevOps flow starts before deployment.

### The baseline I would publish

- every change starts on a branch;
- pull requests are the collaboration and review boundary;
- checks run on PRs before merge;
- merged work is what earns heavier validation and deployment rights.

### Why this matters

A branch gives isolation. A pull request gives review. Status checks give a gate. That combination is simple, scalable, and easy to explain to a team.

### The operating rule

Do not wait until `main` to discover something your feature branch could have told you in minutes.

---

## CI test lanes

The source notes contain a very good layered model. I would keep it almost exactly, but make it explicit.

### Lane 1: fast checks on feature-branch push

Run the things that should almost never be skipped:

- linting;
- unit tests;
- fast static checks;
- lightweight build validation.

For **Node + TypeScript** frontend and API repositories, this is where **Vitest** (`vitest run` or `pnpm exec vitest run`) or, in legacy setups, **Jest** should run on every push. Pick one primary runner per package and document it in `package.json` so CI stays copy-pasteable.

### Lane 2: smoke tests on feature branches

Smoke tests are not the whole e2e catalog. They are the minimum critical path that tells you whether the branch is fundamentally broken.

Use them for:

- app boots;
- login or auth shell works;
- the most critical happy-path flows do not immediately fail.

### Lane 3: full suite on main

Once a branch is merged to the main branch, run the expensive confidence layer:

- broader e2e coverage;
- integration suites;
- slower contract checks (including **OpenAPI codegen** or contract tests when the web app depends on generated types — see the [backend](https://github.com/khasky/backend-architecture-playbook) and [frontend](https://github.com/khasky/frontend-architecture-playbook) playbooks);
- deployment packaging if appropriate.

### Lane 4: on-demand full suite for risky branches

Sometimes you know a branch is large, risky, or hard to reason about. That is when manual full-suite execution on a branch is worth the time.

This is a very healthy capability. It gives teams a way to buy extra certainty without making every single push unbearably slow.

---

## Staging validation

One of the strongest lines in the source notes is also one of the most operationally useful:

> All PRs should be tested on staging using feature branches

That is exactly the kind of sentence a repository guide should contain.

### What "tested on staging" should actually mean

- the deployable artifact from the branch can run in a realistic environment;
- downstream dependencies are present or acceptably simulated;
- the team can verify critical flows before production traffic touches the build.

### What I would require before production rollout

- branch checks passed;
- staging deploy is healthy;
- critical smoke or acceptance path is validated;
- rollback path is understood.

---

## Canary releases

Canary deployment is one of the best ways to reduce release risk without freezing delivery.

### The default model

- expose the new version to a small percentage of traffic first;
- compare the canary against the stable version;
- expand only if health stays good;
- rollback automatically when clear alarm thresholds are crossed.

### What I would watch during a canary

- error rate;
- latency;
- throughput;
- resource saturation;
- business outcomes if the change can affect them.

### A practical canary sequence

1. deploy to staging;
2. validate readiness and health checks;
3. release to a small traffic slice;
4. watch alarms and dashboards during the evaluation window;
5. expand traffic if healthy;
6. rollback if the canary degrades.

A canary is not just "deploy to 5%" It is "deploy to 5% with enough observability and authority to stop".

### The case that defines the stakes

Knight Capital, 1 August 2012. New code was rolled out to eight production servers; one was missed and kept running an old code path that a reused flag now switched on. The firm lost about $440 million in 45 minutes and was gone within days.

Every element of this section exists because of that shape of failure: a rollout that did not reach every host, a flag reused instead of retired, and no authority to stop while the market was open.

---

## Feature flags and progressive delivery

A canary controls which build receives traffic. A flag controls which behavior runs inside it. Teams get into trouble when they treat these as the same lever.

### Why flags earn their place

- deploy and release become separate decisions; code can ship dark and turn on later;
- risky paths get a kill switch that works in seconds, not a redeploy;
- a change can ramp by user segment while the canary ramps by traffic slice.

Flags and canaries are complementary, not competing. The canary protects the deployment. The flag protects the feature. The signals you watch during a ramp are the same ones covered in the [Observability Playbook](https://github.com/khasky/observability-playbook).

### The lifecycle rule

Every flag follows the same arc: create, ramp, clean up.

A flag that shipped at 100% weeks ago is not a feature flag anymore. It is dead code with a runtime cost. I would track flag age and treat stale flags as debt with an owner and a removal date.

### One performance caveat

Keep flag evaluation out of hot inner loops. Resolve the flag once per request or unit of work, then branch on a local value. A flag SDK call inside a tight loop is a self-inflicted latency problem.

---

## Rollback strategy

Rollback is not a note you add because it sounds mature. It is part of the release design.

### Two rollback modes worth supporting

- **automatic rollback**
  - when health checks, error rates, or alarm thresholds fail clearly;
- **manual rollback**
  - when the issue is subtle, business-specific, or not captured by simple thresholds.

### What I would document in every deploy guide

- who can execute rollback;
- which signals trigger it;
- where the rollback command or workflow lives;
- how to verify that rollback actually restored health.

A team that cannot explain rollback in one minute does not yet have a finished deploy process.

---

## Secret scanning

The source material explicitly mentions Gitleaks. That is a good choice.

### Why secret scanning belongs in the playbook

Credential leaks are rarely "interesting" incidents. They are expensive, preventable, and embarrassing.

### Baseline rule

Scan for hardcoded secrets in:

- commits;
- pull requests;
- repositories;
- local pre-commit or CI paths where possible.

Gitleaks is a strong default because it is easy to run in CI and directly targets passwords, API keys, tokens, and similar credential patterns.

---

## Supply chain integrity

Secret scanning protects what leaks out of the repository. Supply chain integrity protects what flows in — and what you ship out.

### Baseline rules

- generate an SBOM in CI for every release artifact, so "what exactly is in this build" is a query, not an investigation;
- attest and sign build artifacts: npm packages with `npm publish --provenance`, container images with sigstore/cosign — SLSA is the useful framing for deciding how far to go;
- pin dependencies through a lockfile and audit that lockfile in CI; a build that resolves different versions on different days is not reproducible;
- pin third-party GitHub Actions to full commit SHAs, not tags — a tag can be moved to malicious code after you reviewed it.

### Why the SHA pin is not paranoia

In March 2025 an attacker compromised `tj-actions/changed-files` and rewrote its tags — `v1` through `v45.0.7` — to point at a malicious commit that dumped CI secrets from the runner's memory into the build log. Roughly 23,000 repositories referenced the action (CVE-2025-30066, CISA advisory). Nothing about the workflow files changed; a tag that had already been reviewed simply started resolving to different code.

Pinning to a full commit SHA was the only configuration that did not move. Public build logs made the leaked secrets readable by anyone, so every affected repository owed a rotation, not just an upgrade.

### The trust rule

Anything that runs in CI with access to your secrets or your artifacts is part of your product. Review it, pin it, and know when it changes.

---

## A practical workflow model

This is the model I would share with a team:

```txt
Feature branch push
  -> lint + unit tests
  -> smoke tests
  -> optional secret scan

Pull request
  -> review
  -> status checks
  -> staging validation path

Merge to main
  -> full suite
  -> build release artifact
  -> deploy to staging or pre-prod
  -> canary rollout
  -> monitor
  -> full rollout or rollback
```

That sequence is simple enough to remember and strict enough to protect production.

---

## Example GitHub Actions layout

This is only an example, but it reflects the intended flow:

```yaml
name: ci

on:
  push:
    branches-ignore:
      - main
  pull_request:
  workflow_dispatch:

jobs:
  fast-checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install
        run: npm ci
      - name: Lint
        run: npm run lint
      - name: Unit tests
        # Vitest (typical for new Vite/React + Node TS repos):
        run: npx vitest run
        # Jest (legacy): npm test -- --runInBand

  smoke-tests:
    if: github.event_name != 'workflow_dispatch' || github.ref != 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run smoke tests
        run: npm run test:smoke

  full-suite:
    if: github.ref == 'refs/heads/main' || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run full test suite
        run: npm run test:full

  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Gitleaks
        run: gitleaks dir .
```

The exact toolchain can change. The layered confidence model should not.

### The trigger to be careful with

`pull_request` runs the fork's workflow with no secrets and a read-only token — that is why the layout above is safe for outside contributions. `pull_request_target` is the dangerous sibling: it runs the *base branch* workflow with full secrets and write access, in the context of a pull request whose code you have not reviewed. Check out untrusted code under that trigger and the fork controls a job holding your credentials.

If a fork PR genuinely needs a secret, split it: a `pull_request` job that builds and uploads an artifact, and a separate privileged workflow triggered on `workflow_run` that consumes the artifact without ever checking out fork code.

---

## OIDC to the cloud, not long-lived secrets

The best cloud credential in a CI system is the one that does not exist between runs.

### The federation default

- CI jobs assume cloud roles through OIDC federation — GitHub Actions has first-class support for AWS, GCP, and Azure;
- no static cloud keys stored in repository secrets;
- tokens are short-lived, scoped to a specific repository and branch, and auditable on the cloud side.

A leaked static key works until someone notices and rotates it. A leaked OIDC token expires on its own in minutes.

### For secrets that must exist

Some credentials cannot be federated — third-party API keys, database passwords. Those live in a secrets manager with rotation, not in repository secrets that nobody remembers to rotate.

---

## Node.js test runners and monorepos

- **Vitest** is the default this playbook assumes for **new** React (Vite) and Node + TypeScript packages (`vitest run`, or `pnpm exec vitest run`). **Jest** remains valid for large legacy repos; use **`npm test -- --runInBand`** (or your existing script) only when that is what the package already defines.
- In a **pnpm workspace** or **npm workspaces** monorepo, prefer **`pnpm turbo run test`** / **`nx test`** (or equivalent) so API, web, and `packages/*` run in dependency order; add a **`openapi:generate`** (or codegen) task to that graph when the UI imports generated types (see the [backend](https://github.com/khasky/backend-architecture-playbook) and [frontend](https://github.com/khasky/frontend-architecture-playbook) playbooks).
- Keep **one** documented unit-test entry point per package so `fast-checks` jobs stay boring to copy across repos.

---

## Infrastructure as code, minimally

Whatever provisions your infrastructure lives in version control and goes through pull requests, like any other code.

### The provisioning baseline

- infra changes are diffs, reviewed like code;
- plan or preview runs in CI on the pull request; apply runs on merge;
- console clicking in production is drift, and drift is an outage waiting for the next redeploy to expose it.

### Start small

Start with the smallest tool that holds. A reviewed, versioned script beats clicking through a console. Terraform, Pulumi, or CDK earn their place when the surface grows — the non-negotiable part is not the tool, it is that provisioning is code with a review gate.

---

## Things I would avoid

- only running serious tests after merge;
- treating staging as ceremonial instead of useful;
- giant all-or-nothing rollouts by default;
- canaries without dashboards and alarms;
- rollback plans that live only in tribal knowledge;
- storing secrets in tracked files;
- making the slowest suite run on every tiny push when a layered model would work better.

---

## References and inspiration

### Official and high-signal references

- [GitHub Actions documentation](https://docs.github.com/actions)
- [Continuous integration with GitHub Actions](https://docs.github.com/en/actions/get-started/continuous-integration)
- [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow)
- [Amazon ECS canary deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/canary-deployment.html)

### Tooling references

- [Gitleaks](https://github.com/gitleaks/gitleaks)

### Similar or adjacent GitHub repositories

- [Awesome DevOps](https://github.com/wmariuss/awesome-devops)
- [Awesome DevOps Tools](https://github.com/Curated-Awesome-Lists/awesome-devops-tools)

---

## License

MIT is a sensible default for a playbook repository like this, but choose the license that fits your sharing goals.
