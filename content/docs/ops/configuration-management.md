---
menu:
  docs:
    parent: policies
layout: ops
title: Configuration management
---

<!-- This page is important for FedRAMP compliance. See the CM family of controls, including CM-9. Code Climate is part of SA-11 (1), SI-3, and RA-5. -->

This document describes how the cloud.gov team approaches configuration management of the core platform. Before configuration changes go into production, they need to pass our [significant change rubric]({{< relref "docs/ops/continuous-monitoring.md#appendix-significant-change-rubric" >}}), as described in our [Feature Lifecycle](https://github.com/18F/cg-product/blob/master/FeatureLifecycle.md) and [Story Lifecycle](https://github.com/18F/cg-product/blob/master/StoryLifecycle.md).

## What goes into configuration management?
In short, everything needed to run and operate the platform that is not a _secret_. (See [Secret Key Management]({{< relref "docs/ops/secrets.md" >}}) for that.)

Here are some examples that should be in configuration management:

- CI pipelines (Concourse)
- Infrastructure/network configuration (Terraform)
- VM setup and quantity (BOSH)
- Software configuration (BOSH)
- 18F-developed code

*Special cases:* For changing settings that we currently cannot manage as configuration files in version control, such as GitHub repository settings and Nessus scan settings, you must first get agreement from another cloud.gov team member that the change should be made (such as over Slack or Hangouts).

## Where should all this configuration go?
All configuration must be stored in GitHub using the following "Change Workflow" unless it is a _secret_.

## How do we test these changes?
If possible, first test the changes locally. After that, upload them to a development environment where either manual or automated testing needs to be run.
Security tests need to be executed in the development environment where changes are applied.

## Change workflow

1. All configuration changes must flow through a git repository, centrally managed through GitHub, unless they contain sensitive information. In these cases, sensitive information should be stored in an S3 bucket with a proper security policy and encryption, versioned such that changes can be easily rolled back.
1. A change is initiated and discussed, following the steps in our [Story Lifecycle](https://github.com/18F/cg-product/blob/master/StoryLifecycle.md).
1. In the appropriate GitHub repository for the component, a pull request (PR) against the `master` branch is created that addresses the change.
1. If the repository contains 18F-developed code, the PR must have an automated [Code Climate](https://codeclimate.com) check, which must pass before the PR can be merged.
1. The PR is reviewed by someone other than the committer. Pairing via screen-sharing
is encouraged and qualifies as a review. Review should include assessment of architectural design, DRY principles, security and code quality.
    The reviewer approves the PR via GitHub.
1. The reviewer merges the approved PR.
    The committer may merge an approved PR if the changes made are time-sensitive.
1. A continuous integration (CI) server handles automated tests and continuous deployment (CD) of the merged changes.
    - All changes are deployed to a testing environment, such as development.
    - Any and all automated tests are run.
    - If all tests pass, changes can be promoted for deployment to production in the pipeline.

1. The CI/CD tool uses GitHub repositories and S3-stored sensitive content as the canonical source of truth for what the platform should look like. If there are manual changes, it will reset the state of all systems to match.

## Checklist for new repositories

Before we put a new repository into production:

* Give it a name starting with `cg-` or `cf-` (to make clear that it's part of our work, unless we have a good reason to name it something else).
* [Add `LICENSE`, `CONTRIBUTING`, and `README` files](https://github.com/18F/open-source-policy/blob/master/practice.md#how-to-license-18f-repos) (to support open source reuse of our work).
* Set up [Code Climate](https://codeclimate.com/) for pull requests (SI-3).
  * Go to Code Climate and log in using your GitHub auth.
  * Give Code Climate permissions to access your public repositories including the 18F org.
  * Go to [the open source dashboard](https://codeclimate.com/oss/dashboard) and [add the new repository](https://codeclimate.com/github/repos/new).
  * In Code Climate, go to the repository settings, go to Integrations, and enable Pull Request checking.
  * In the repo, add a [Code Climate configuration file](https://docs.codeclimate.com/docs/advanced-configuration) (`.codeclimate.yml`) to [enable relevant plugins](https://docs.codeclimate.com/docs/list-of-engines).
* Configure a [protected master branch](https://help.github.com/articles/about-protected-branches/) (CM-9).
  * Enable "Require pull request reviews before merging"
  * Enable "Dismiss stale pull request approvals when new commits are pushed"
  * Enable "Require status checks to pass before merging"
  * Enable "Require branches to be up to date before merging"
  * Enable "Include administrators"
* Configure permissions (CM-3):
  * If it's a platform configuration repo, restrict permissions to Cloud Ops, as follows:
     * `Read` for [18F](https://github.com/orgs/18F/teams/18f/members).
     * `Read` for [cloud-gov](https://github.com/orgs/18F/teams/cloud-gov/members).
     * `Admin` for [cloud-gov-ops](https://github.com/orgs/18F/teams/cloud-gov-ops/members).
  * If it's not a platform configuration repo, configure as follows:
     * `Read` for [18F](https://github.com/orgs/18F/teams/18f/members).
     * `Admin` for [cloud-gov](https://github.com/orgs/18F/teams/cloud-gov/members).
     * `Admin` for [cloud-gov-ops](https://github.com/orgs/18F/teams/cloud-gov-ops/members).
* Set up CI/CD for changes (CM-3).
* Open a PR to add it to the [repos list]({{< relref "docs/ops/repos.md#repositories" >}}) (to help us keep track of our repos and support open source reuse of our work).
* Open a PR to add it to the [repos list for pre-merge checks](https://github.com/18F/cg-pipeline-tasks/blob/master/security-considerations/repos.txt)

## What if a configuration changed and it is not in Configuration Management?
If possible, Configuration Management tools need to be set up to always roll back to a known state. Other than that, these tools need to be able to "recreate" all settings from the known configurations.

## Roles and responsibilities

* **All team members**
  * Follow the configuration management plan.
  * Make suggestions (such as in PRs) if you have ideas for improving the plan.
* **Cloud Ops** (Platform squad)
  * Ensure Concourse, Terraform, BOSH, GitHub, AWS, and other resources are correctly set up to implement the technical aspects of the plan.
  * Review the plan in our quarterly [Security Policy and Account Review meetings](https://drive.google.com/drive/folders/0B1nrtBDSrHKHSnByWklWc0V4Q1E).
* **Program Manager**
  * Ensure the team follows the Feature Lifecycle, Story Lifecycle, and other operational aspects of the plan.
* **System Owner**
  * Ensure that team members uphold their responsibilities.
  * Approve any major changes to the plan, and coordinate with JAB representatives as necessary.

## GitHub contribution guidelines

<!-- As long as these GitHub guidelines comply with the FedRAMP-required policies above and our 18F GitHub requirements, we can adjust them at will, according to our team preferences. -->

Because cloud.gov is built and maintained by 18F, we follow [the 18F requirements for using GitHub](https://handbook.18f.gov/github/). These are our team practices within those requirements.

### Forking vs. branching

Both forking and branching are welcome in our repositories. Contributors inside 18F can use forking or branching according to their personal preferences, and contributors outside 18F can fork repositories.

The team often practices branching. The rationale for branching within a team is
that paired collaboration on a single branch avoids certain types of friction:

- Having to create multiple forked PRs in order to contribute to the branch
- Having to add new users to forked repositories as collaborators in order to
  have people directly contribute on short-lived forked branches

When contributing directly on a branch, we're able to modify work-in-progress (WIP) pull requests and encourage collaboration across the Cloud Operations team.

### Squashing commits

[Squashing commits](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History#Squashing-Commits) is allowed but discouraged, except in rare instances.

### Rebase or merge

The team prefers [rebasing over merging](https://www.atlassian.com/git/tutorials/merging-vs-rebasing/).

### When should a PR be created?

Work-in-progress PRs are encouraged. When a PR is ready for review, it should be tagged in GitHub
with the `review-needed` label. If you create a work-in-progress PR, you might also make it plain in the PR name with a `[WIP]` prefix.

### Should PRs be assigned?

PRs are typically not assigned in GitHub, unless someone specifically needs to sign off on the change.

You can request a review using GitHub's built-in tools, mention someone in the PR with the `@` notation, or contact them outside the GitHub context to request a review.

### When reviewing a PR, should the change be tested locally?

Whenever possible, the proposed changes should be tested locally. Because of the nature of many of the cloud.gov repositories and deployment environments, local testing is not always possible or practical. Visual code review, however, is always required.
