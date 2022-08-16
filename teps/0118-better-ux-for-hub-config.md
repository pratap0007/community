---
status: proposed
title: Better UX for Hub Config
creation-date: '2022-07-27'
last-updated: '2022-07-27'
authors:
- '@PuneetPunamiya'
- '@piyush-garg'
- '@pratap0007'
---

# TEP-0118: Better UX for Hub Config

<!--
**Note:** Please remove comment blocks for sections you've filled in.
When your TEP is complete, all of these comment blocks should be removed.

To get started with this template:

- [ ] **Fill out this file as best you can.**
  At minimum, you should fill in the "Summary", and "Motivation" sections.
  These should be easy if you've preflighted the idea of the TEP with the
  appropriate Working Group.
- [ ] **Create a PR for this TEP.**
  Assign it to people in the Working Group that are sponsoring this process.
- [ ] **Merge early and iterate.**
  Avoid getting hung up on specific details and instead aim to get the goals of
  the TEP clarified and merged quickly. The best way to do this is to just
  start with the high-level sections and fill out details incrementally in
  subsequent PRs.

Just because a TEP is merged does not mean it is complete or approved. Any TEP
marked as a `proposed` is a working document and subject to change. You can
denote sections that are under active debate as follows:

```
<<[UNRESOLVED optional short context or usernames ]>>
Stuff that is being argued.
<<[/UNRESOLVED]>>
```

When editing TEPS, aim for tightly-scoped, single-topic PRs to keep discussions
focused. If you disagree with what is already in a document, open a new PR
with suggested changes.

If there are new details that belong in the TEP, edit the TEP. Once a
feature has become "implemented", major changes should get new TEPs.

The canonical place for the latest set of instructions (and the likely source
of this file) is [here](/teps/tools/tep-template.md.template).

-->

<!--
This is the title of your TEP. Keep it short, simple, and descriptive. A good
title can help communicate what the TEP is and should be considered as part of
any review.
-->

<!--
A table of contents is helpful for quickly jumping to sections of a TEP and for
highlighting any additional information provided beyond the standard TEP
template.

Ensure the TOC is wrapped with
  <code>&lt;!-- toc --&rt;&lt;!-- /toc --&rt;</code>
tags, and then generate with `hack/update-toc.sh`.
-->

<!-- toc -->
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
  - [Use Cases](#use-cases)
  - [Requirements](#requirements)
- [Proposal](#proposal)
  - [Notes and Caveats](#notes-and-caveats)
- [Design Details](#design-details)
- [Design Evaluation](#design-evaluation)
  - [Reusability](#reusability)
  - [Simplicity](#simplicity)
  - [Flexibility](#flexibility)
  - [User Experience](#user-experience)
  - [Performance](#performance)
  - [Risks and Mitigations](#risks-and-mitigations)
  - [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
- [Implementation Plan](#implementation-plan)
  - [Test Plan](#test-plan)
  - [Infrastructure Needed](#infrastructure-needed)
  - [Upgrade and Migration Strategy](#upgrade-and-migration-strategy)
  - [Implementation Pull Requests](#implementation-pull-requests)
- [References](#references)
<!-- /toc -->

## Summary

Tekton catalog is a collection of Tekton resources such as Tasks and Pipelines that provides reusable resources
which enables developers to integrate those in their workflows. Tekton Hub, a web based platform shows all these
resources with it's detailed information and yaml. The basic information required for Tekton Hub
i.e. categories, minimum catalog related details and the scopes required for user are provided through a [config.yaml](https://github.com/tektoncd/hub/blob/main/config.yaml)
Currently the `config.yaml` for hub needs to be hosted over some remote location and we need to provide the corresponding URL to the Hub API server via ConfigMap. Not only there is a dependency on hosting this file on remote location but also the maintenance of the file is not pretty scalable.

This is how the config.yaml looks like 
  ```yaml
  ---
  categories:
    - name: Automation
    - name: Build Tools
    - name: CLI
    - name: Cloud
    - name: Code Quality
    - name: Continuous Integration
    - name: Deployment
    - name: Developer Tools
  
  catalogs:
    - name: tekton
      org: tektoncd
      type: community
      provider: github
      url: https://github.com/tektoncd/catalog
      revision: main
  
  scopes:
    - name: agent:create
      users: [abc, xyz, qwe]
    - name: catalog:refresh
      users: [abc, xyz, qwe]
    - name: config:refresh
      users: [abc, xyz, qwe]
  
  default:
    scopes:
      - rating:read
      - rating:write
  ```

## Motivation

<!--
This section is for explicitly listing the motivation, goals and non-goals of
this TEP. Describe why the change is important and the benefits to users. The
motivation section can optionally provide links to [experience reports][experience reports]
to demonstrate the interest in a TEP within the wider Tekton community.

[experience reports]: https://github.com/golang/go/wiki/ExperienceReports
-->

### Goals

* Remove dependency of outside git file for config.yaml
* Have easy maintenance for config.yaml


### Non-Goals

<!--
Listing non-goals helps to focus discussion and make progress.
- What is out of scope for this TEP?
-->

### Use Cases

<!--
Describe the concrete improvement specific groups of users will see if the
Motivations in this doc result in a fix or feature.

Consider the user's:
- [role][role] - are they a Task author? Catalog Task user? Cluster Admin? e.t.c.
- experience - what workflows or actions are enhanced if this problem is solved?

[role]: https://github.com/tektoncd/community/blob/main/user-profiles.md
-->

### Requirements

<!--
Describe constraints on the solution that must be met, such as:
- which performance characteristics that must be met?
- which specific edge cases that must be handled?
- which user scenarios that will be affected and must be accommodated?
-->

## Proposal


Currently the config.yaml for hub needs to be hosted over some URL and we need to provide the URL for that to the Hub API deployment through configmap.

1. If the URL is not changing

   - Whenever you do some changes to it or you need to make some changes to it, if the URL is not changing, then you need to hit config refresh API manually to have the changes in Hub.


2. If the URL gets changed
   - If the URL gets changed based on some changes like you are using a particular commit etc then you need to update the configmap with the URL, have the new data from configmap populated in deployment and hit config refresh.


3. Current operator implementation
    - The config.yaml is present in Hub git repo, so whenever user wants scopes such as `agent:create`, `catalog:refresh`, etc other than default scopes user needs to update the `config.yaml` and push it in his git repository and update the URL in Hub CR by pointing to his git repo if user is deploying its own Hub instance

Present Structure of config.yaml in git repository
```yaml
---
categories:
  - name: Automation
  - name: Build Tools
  - name: CLI
  - name: Cloud
  - name: Code Quality
  - name: Continuous Integration
  - name: Deployment
  - name: Developer Tools
  - name: Image Build
  - name: Integration & Delivery
  - name: Git
  - name: Kubernetes
  - name: Messaging
  - name: Monitoring
  - name: Networking
  - name: Openshift
  - name: Publishing
  - name: Security
  - name: Storage
  - name: Testing

catalogs:
  - name: tekton
    org: tektoncd
    type: community
    provider: github
    url: https://github.com/tektoncd/catalog
    revision: main

scopes:
  - name: agent:create
    users: [abc, xyz, qwe]
  - name: catalog:refresh
    users: [abc, xyz, qwe]
  - name: config:refresh
    users: [abc, xyz, qwe]

default:
  scopes:
    - rating:read
    - rating:write
```


* Pros:-
  * Config.yaml can be centrally managed, admin has to just hit `config/refresh` api or even can set up a cron job at regular intervals. (Not the case if url is changing)

* Cons:-
  * Dependency on outside git file
    * To fetch the details from `config.yaml` Hub API makes an http call to get all the data. So to get all the categories, catalog details and initial users data in the db we need to depend on Git Repo i.e. if git is down we won’t get the data if we are spinning up Hub instance for the first time
  * The first config.yaml creation UX for its own hub instance is not good.
  * Maintenance cycle UX is not good, too many steps.
  * Operator UX is also not good, the user needs to move out of the cluster to change the config.

### Notes and Caveats

<!--
(optional)

Go in to as much detail as necessary here.
- What are the caveats to the proposal?
- What are some important details that didn't come across above?
- What are the core concepts and how do they relate?
-->


## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable. This may include API specs (though not always
required) or even code snippets. If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.

If it's helpful to include workflow diagrams or any other related images,
add them under "/teps/images/". It's upto the TEP author to choose the name
of the file, but general guidance is to include at least TEP number in the
file name, for example, "/teps/images/NNNN-workflow.jpg".
-->


## Design Evaluation
<!--
How does this proposal affect the api conventions, reusability, simplicity, flexibility
and conformance of Tekton, as described in [design principles](https://github.com/tektoncd/community/blob/master/design-principles.md)
-->

### Reusability

<!--
https://github.com/tektoncd/community/blob/main/design-principles.md#reusability

- Are there existing features related to the proposed features? Were the existing features reused?
- Is the problem being solved an authoring-time or runtime-concern? Is the proposed feature at the appropriate level
authoring or runtime?
-->

### Simplicity

<!--
https://github.com/tektoncd/community/blob/main/design-principles.md#simplicity

- How does this proposal affect the user experience?
- What’s the current user experience without the feature and how challenging is it?
- What will be the user experience with the feature? How would it have changed?
- Does this proposal contain the bare minimum change needed to solve for the use cases?
- Are there any implicit behaviors in the proposal? Would users expect these implicit behaviors or would they be
surprising? Are there security implications for these implicit behaviors?
-->

### Flexibility

<!--
https://github.com/tektoncd/community/blob/main/design-principles.md#flexibility

- Are there dependencies that need to be pulled in for this proposal to work? What support or maintenance would be
required for these dependencies?
- Are we coupling two or more Tekton projects in this proposal (e.g. coupling Pipelines to Chains)?
- Are we coupling Tekton and other projects (e.g. Knative, Sigstore) in this proposal?
- What is the impact of the coupling to operators e.g. maintenance & end-to-end testing?
- Are there opinionated choices being made in this proposal? If so, are they necessary and can users extend it with
their own choices?
-->

### Conformance

<!--
https://github.com/tektoncd/community/blob/main/design-principles.md#conformance

- Does this proposal require the user to understand how the Tekton API is implemented?
- Does this proposal introduce additional Kubernetes concepts into the API? If so, is this necessary?
- If the API is changing as a result of this proposal, what updates are needed to the
[API spec](https://github.com/tektoncd/pipeline/blob/main/docs/api-spec.md)?
-->

### User Experience

<!--
(optional)

Consideration about the user experience. Depending on the area of change,
users may be Task and Pipeline editors, they may trigger TaskRuns and
PipelineRuns or they may be responsible for monitoring the execution of runs,
via CLI, dashboard or a monitoring system.

Consider including folks that also work on CLI and dashboard.
-->

### Performance

<!--
(optional)

Consider which use cases are impacted by this change and what are their
performance requirements.
- What impact does this change have on the start-up time and execution time
of TaskRuns and PipelineRuns?
- What impact does it have on the resource footprint of Tekton controllers
as well as TaskRuns and PipelineRuns?
-->

### Risks and Mitigations

<!--
What are the risks of this proposal and how do we mitigate? Think broadly.
For example, consider both security and how this will impact the larger
Tekton ecosystem. Consider including folks that also work outside the WGs
or subproject.
- How will security be reviewed and by whom?
- How will UX be reviewed and by whom?
-->

### Drawbacks

<!--
Why should this TEP _not_ be implemented?
-->

## Alternatives

<!--
What other approaches did you consider and why did you rule them out? These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->


## Implementation Plan

<!--
What are the implementation phases or milestones? Taking an incremental approach
makes it easier to review and merge the implementation pull request.
-->


### Test Plan

<!--
Consider the following in developing a test plan for this enhancement:
- Will there be e2e and integration tests, in addition to unit tests?
- How will it be tested in isolation vs with other components?

No need to outline all the test cases, just the general strategy. Anything
that would count as tricky in the implementation and anything particularly
challenging to test should be called out.

All code is expected to have adequate tests (eventually with coverage
expectations).
-->

### Infrastructure Needed

<!--
(optional)

Use this section if you need things from the project or working group.
Examples include a new subproject, repos requested, GitHub details.
Listing these here allows a working group to get the process for these
resources started right away.
-->

### Upgrade and Migration Strategy

<!--
(optional)

Use this section to detail whether this feature needs an upgrade or
migration strategy. This is especially useful when we modify a
behavior or add a feature that may replace and deprecate a current one.
-->

### Implementation Pull Requests

<!--
Once the TEP is ready to be marked as implemented, list down all the GitHub
merged pull requests.

Note: This section is exclusively for merged pull requests for this TEP.
It will be a quick reference for those looking for implementation of this TEP.
-->

## References

<!--
(optional)

Use this section to add links to GitHub issues, other TEPs, design docs in Tekton
shared drive, examples, etc. This is useful to refer back to any other related links
to get more details.
-->
