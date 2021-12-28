---
title: "Spring Boot: Docker builds"
date: "2021-12-28T01:00:00"
draft: true
disableShare: true
hideFooter: true
tags: ['java', 'spring', 'spring-boot', 'docker', 'github actions']
categories: ['cloud', 'automation']
---

### Setting up Spring Docker builds

We use [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html) and [Docker](https://www.docker.com/) for our production runtime. [GitHub Actions](https://github.com/features/actions) builds our containers and handles deployments.

When releasing a new version of our API the following steps are performed:

{{< mermaid align="left" theme="neutral" >}}
flowchart LR
    loginToECR(AWS Login to ECR) --> MavenBuild(Maven Build, Tag, Push) --> AddTaskId(Add new tag to Task Definition) --> DeployTask(Deploy new AWS Fargate Task)
{{< /mermaid >}}




- I'm going to be focusing on our [Rest API](https://en.wikipedia.org/wiki/Representational_state_transfer) which is accessible only via [Swagger UI](https://swagger.io/tools/swagger-ui/).
- We use [JSON Web Token](https://en.wikipedia.org/wiki/JSON_Web_Token) using [Firebase Authentication](https://firebase.google.com/docs/auth) when using our App or [Desktop](https://www.squaddy.app/squaddy-pro).



{{< detail-tag "CLICK ME" >}}
This text will be hidden

#### yes, even hidden code blocks! (seems like bold title dont work here too)

```python
print("hello world!")
```
{{< /detail-tag >}}