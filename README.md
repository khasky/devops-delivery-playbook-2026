# DevOps Delivery Playbook

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
  - [Rollback strategy](#rollback-strategy)
    - [Two rollback modes worth supporting](#two-rollback-modes-worth-supporting)
    - [What I would document in every deploy guide](#what-i-would-document-in-every-deploy-guide)
  - [Secret scanning](#secret-scanning)
    - [Why secret scanning belongs in the playbook](#why-secret-scanning-belongs-in-the-playbook)
    - [Baseline rule](#baseline-rule)
  - [A practical workflow model](#a-practical-workflow-model)
  - [Example GitHub Actions layout](#example-github-actions-layout)
  - [Node.js test runners and monorepos](#nodejs-test-runners-and-monorepos)
  - [Things I would avoid](#things-i-would-avoid)
  - [References and inspiration](#references-and-inspiration)
    - [Official and high-signal references](#official-and-high-signal-references)
    - [Tooling references](#tooling-references)
    - [Similar or adjacent GitHub repositories](#similar-or-adjacent-github-repositories)
  - [License](#license)

---

## Companion playbooks

These repositories form one playbook suite:

- [Auth & Identity Playbook](https://github.com/khasky/auth-identity-playbook) — sessions, tokens, OAuth, and identity boundaries across the stack
- [Backend Architecture Playbook](https://github.com/khasky/backend-architecture-playbook) — APIs, boundaries, OpenAPI, persistence, and errors
- [Best of JavaScript](https://github.com/khasky/best-of-javascript) — curated JS/TS tooling and stack defaults
- [Caching Playbook](https://github.com/khasky/caching-playbook) — HTTP, CDN, and application caches; consistency and invalidation
- [Code Review Playbook](https://github.com/khasky/code-review-playbook) — PR quality, ownership, and review culture
- **DevOps Delivery Playbook** — CI/CD, environments, rollout safety, and observability
- [Engineering Lead Playbook](https://github.com/khasky/engineering-lead-playbook) — standards, RFCs, and technical leadership habits
- [Frontend Architecture Playbook](https://github.com/khasky/frontend-architecture-playbook) — React structure, performance, and consuming API contracts
- [Marketing and SEO Playbook](https://github.com/khasky/marketing-and-seo-playbook) — growth, SEO, experimentation, and marketing surfaces
- [Monorepo Architecture Playbook](https://github.com/khasky/monorepo-architecture-playbook) — workspaces, package boundaries, and shared code at scale
- [Node.js Runtime & Performance Playbook](https://github.com/khasky/nodejs-runtime-performance-playbook) — event loop, streams, memory, and production Node performance
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

---

## Node.js test runners and monorepos

- **Vitest** is the default this playbook assumes for **new** React (Vite) and Node + TypeScript packages (`vitest run`, or `pnpm exec vitest run`). **Jest** remains valid for large legacy repos; use **`npm test -- --runInBand`** (or your existing script) only when that is what the package already defines.
- In a **pnpm workspace** or **npm workspaces** monorepo, prefer **`pnpm turbo run test`** / **`nx test`** (or equivalent) so API, web, and `packages/*` run in dependency order; add a **`openapi:generate`** (or codegen) task to that graph when the UI imports generated types (see the [backend](https://github.com/khasky/backend-architecture-playbook) and [frontend](https://github.com/khasky/frontend-architecture-playbook) playbooks).
- Keep **one** documented unit-test entry point per package so `fast-checks` jobs stay boring to copy across repos.

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
