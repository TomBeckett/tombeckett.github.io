---
title: "I've stopped using continuous delivery"
date: "2022-10-29T01:00:00"
disableShare: true
hideFooter: true
tags: ["technology", "github actions"]
categories: ["programming"]
---

Every project I have configured over the last five or so years has used [continuous deployment](https://en.wikipedia.org/wiki/Continuous_deployment).

Here is how [Atlassian](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment) describes it:

> Continuous deployment goes one step further than continuous delivery. With this practice, every change that passes all stages of your production pipeline is released to your customers. There's no human intervention, and only a failed test will prevent a new change to be deployed to production.
>
> Continuous deployment is an excellent way to accelerate the feedback loop with your customers and take pressure off the team as there isn't a "release day" anymore. Developers can focus on building software, and they see their work go live minutes after they've finished working on it.

It's fair to say I am _a fan_.

Continuous deployment has four ideas that greatly appeal to me:

- Deployment is **fast** (and therefore cheap).
- Automated testing is sufficient.
- The smallest change possible between deployments is also **safest**.
- If a bug is found, it is quick and easy to release a new version and understand which commit caused the issue _(hint - its probably the last one)_.

On its surface this seems elegantly simple but I've learnt it has its trade offs.

## No matter how fast, we don't always want a release

There are many times you merge a pull request but don't actually want or need a release. These can be new assets, documentation updates, code coverage configuration, changes to a development deployment pipeline, etc.

I've seen a few ways that developers respond to these unwanted builds:

- **Cancel unwanted builds**: Causing insignificant amount runner wastage across organizations even if you're fast.
- **Add various path ignores to the deployment trigger**: Introducing path globs across almost-duplicated workflows that only differ in ignored environment specific file paths.
- **Merge changes in a release PR**: Developers side step builds by to merging non-deployment commits into a single PR along with a change they **do** want to deploy. This breaks the _smaller changes between deployments_ goal.

We need to drop this assumption that _every_ merge is a deliverable that needs to be deployed. The reality is that there a lot of churn that takes place in a repository and its not easy to automate it away.

## Automated testing is not sufficient and smaller releases are not safer

Anyone who has used [dependabot](https://github.com/dependabot) on a JavaScript project has experienced the pain of almost [daily library updates](https://imgur.com/C9iUXNW.png).

I think is especially prevalent in the JavaScript ecosystem since many library developers are also website developers _where continuous deployment makes sense_.

However, libraries are not websites. A regular release cadence with manual testing is far more preferable to **merge -> automated test -> automated release**.

This is further compounded by **fixes can only move forward** where you (the library consumer) feel stuck within a **new feature release -> bug fix release** cycle.

## When to use it

Continuous delivery can work when:

- Your users are happy to manually test for you
- Releasing requires no action on the part of users.
- The churn of non-deployment code (docs, config, etc) is far lower than your features/fixes.

Should any of this change, I recommend **giving up some agility for stability**.

## What to use otherwise

I recommend [keeping change log](https://keepachangelog.com/en/1.0.0/) of unreleased changes and a [semantic version](https://semver.org/spec/v2.0.0.html) in a `package.json`, `pom.xml` type file.

When its time to release:

1. Merge in a pull request that moves the unreleased entries to a new version heading (which matches the semantic version)
2. [Manually trigger a workflow](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow) against the appropriate branch:
   - `git tag` using the release (the semver from the file),
   - Create a GitHub Release with entries from the `CHANGELOG.md` file
   - Perform the automated deployment.

This is not a radical approach and it's been used by large projects for a long time.

I've learnt that to go faster, you actually need to go slower sometimes.
