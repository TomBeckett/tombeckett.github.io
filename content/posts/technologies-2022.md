---
title: "Technologies Retrospective 2021"
date: "2022-01-04T01:00:00"
disableShare: true
hideFooter: true
tags: ['retrospective', 'learning']
categories: ['programming', 'meta']
---

Here's my reflection on what technologies I used in 2021 and what I'd like to use in 2022. I also detail my current preferred stack for this year.

## 2021

### The Rules

I've used a lot of different technologies in 2021.

To keep things sensible, I had to use the technology...

- In production with real customers - that means no 'I did the tutorial' opinions allowed.
- Either for the first time or after a break.

I'll keep my opinions short to prevent this article getting too long.

### VueJS

I'm a big React fan and switching to VueJS for a couple of months was eye opening. Overall I like the approach of mixing your HTML and JavaScript in a `<script>` tag and I think for simple use cases (which don't require a huge SPA app) VueJS is excellent.

However, I didn't like the lack of TypeScript support in the HTML and I actually think VueJS 3's [composition api](https://v3.vuejs.org/guide/composition-api-introduction.html) to be a step backwards.

**Verdict:** I'm open to working with VueJS more and may revisit VueJS3 again. But for now, I'll be sticking with React.

### Python 3

I've used Python a few times previously for short DevOps scripts (basically whenever Bash gets too much) but this year I had to create a few dashboards using Python. Nothing too complex but enough to make me realise that unless I needed a Python only library, I'd rather try R or Go more; it's not *bad* just not *good*.

Asking colleagues how to import files into other files, how best to get our artifact into a shippable form or use environment variables all could be done, but felt harder than it should be.

After thinking I was crazy, I spoke to a colleague who put it well:

> Python has an easy onboarding experience, but after a while you start to need deep knowledge to keep it performant and build something really big with it. 

On the other hand, I really enjoyed using [PyCharm](https://www.jetbrains.com/pycharm/) for a few months and having used [Rider](https://www.jetbrains.com/rider/) and [IntelliJ](https://www.jetbrains.com/idea) it really helped bridge the gap for someone so rusty.

**Verdict:** I'll be trying to learn an alternative in 2022.

### AWS SAM & CloudFormation

I did a lot of [Amazon Web Services (AWS)](https://aws.amazon.com/) this year. I've used a lot of Ansible, Azure Resource Templates and Terraform in the past and so it felt natural to use [AWS SAM](https://aws.amazon.com/serverless/sam/) and [CloudFormation](https://aws.amazon.com/cloudformation/).

It often did what I wanted to even if it took a few tries. That said...

- I **hated** the error messages both tools would give on failure. This got better over time due to SAM fixes and my own knowledge improving.
- Stack drift is still something I don't think AWS has resolved.
- If a stack fails during creation you then have to remove manually (rather than try again) is just time consuming.
- It's also far too easy to get yourself into a circular dependency chain with the only fix being to break your stacks up in unusual ways.
- AWS documentation is often contradictory with out-of-date blogs being a particular issue or tutorials that simply don't work.
- CloudFormation (and by extension SAM) often doesn't have support for new services. This goes against the advice AWS themselves give all users - automate all deployments, don't use the UI.

After all that you may think *"gee, I guess you're going to not use AWS then?"* but I will still be using AWS as my primary cloud. Unfortunately AWS is at the scale they can afford to have a bad developer experience. Also they're completing against GCP and Azure which are somehow worse.

**Verdict:** Please [CloudFlare](https://www.infoq.com/news/2021/10/cloudflare-r2-egress-aws/) or [Vercel](https://vercel.com/) save us all in 2022.

### AWS Lambda & DynamoDB

[Lambda](https://aws.amazon.com/lambda/) and [DynamoDB](https://aws.amazon.com/dynamodb/) are some of the oldest services provided by AWS. They have been far and away the better experiences mostly due to how basic they are. Updating both has been quick with good SAM/Cloudformation support. 

I would not hesitate to use both again but I did learn somethings I would insist on:

- DynamoDB should either be treated as either a hashmap/temporary storage OR a single table with many object types. Multi-table is a mistake.
- Lambda provisioned capacity is a trick and you're better of moving it to [ECS](https://aws.amazon.com/ecs/) as a dedicated always on service.
- Use [Step Functions](https://aws.amazon.com/step-functions/?step-functions.sort-by=item.additionalFields.postDateTime&step-functions.sort-order=desc) whenever possible.
- [HTTP API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api.html) works okay but I'd avoid hosting an *entire* API with only lambdas again. That's a post in itself but it was a real nightmare.

**Verdict:** Use it but for small services. If they get too big, move them to a container.

### AWS Elastic Container Service (ECS)

After the API Gateway nightmare I mentioned above, I moved to containers and this is how I'd host any backend in future. 

It's been fairly bulletproof and supports A/B or Blue/Green deployments, secrets, auto scaling and is well supported with infrastructure as code.

**Verdict:** I have no complaints.

### AWS S3

It took a while to get used to S3. It's extremely basic with often too many options. Configuring permissions (ACL/Policies) or CORS in particular is just a nightmare. I think the integration with CloudFront (often what you *should* be using) is flakey at best. 

That said, I have a good set of CloudFormation templates now (for IAM/CloudFront and S3) and I've mostly learn which parts to ignore. I would *not* say this is easy service to learn but as with most things AWS - you've not got a lot of choice right now.

**Verdict:** Use it, but only because I have to.

### Google CloudBuild & CloudRun

I use Firebase Website Hosting (which is CloudRun) for a few websites and also build/host our main Slackbot using GCP. It was fairly painless and I don't tend to go into GCP very often - only for Google Workspace/Domain/SAML settings. This leaves me feeling conflicted.

On the one hand, I find it easy to use to get going and it's nice to not be locked into AWS - both financially and in knowledge. On the other, I find the UX somehow even worse than AWS. It is so bad I can't actually describe it.

**Verdict:** If I was forced to switch to CloudRun I could use it but it's not my chosen tool.

### Firebase Auth

This is Firebase's killer product. Better than Hosting, Functions, Analytics, Crashlytics even Database. It's all the things AWS Cognito should have been.

**Verdict:** Used it on two projects, would use again.

### Cypress

It seems like this was the year Cypress arrived. It was everywhere in the JavaScript world and really came into its own in 2020/2021. It's the best way to functionally test a SPA application.

But it still needs improving and I've often found it to completely break (or slow to a crawl) between major versions. It doesn't feel mature as a project yet.

**Verdict:** My tool of choice but I do try to avoid updating it unless I need to.

### Sinon/Chai

I was recommended to switch our Jest tests to Sinon/Chai by a JavaScript guru. I'm very much of the opinion that testing frameworks are extremely similar and usually don't have any strong opinions. In this case, I absolutely prefer Sinon/Chai. 

- The syntax is much better 
- The tests are far faster.
- Mocking is closer to regular code than Jest mocking voodoo.

When paired with Cypress (which uses Chai syntax) it really unifies the testing experience. Thank you Chris!

**Verdict:** Must use on each project for any TypeScript/JavaScript unit testing.

### Spring JPA

Almost every item on this list was new to me in 2020 or 2021 but I've used Spring JPA for many years.

It makes the list because I actually took a break from it for a year or so and this year needed it for a major project. I dove deeper into Spring Data than I've ever had to before and really got a much better understanding of it and Hibernate as a result.

The more I learnt about Spring Data, JPA and Hibernate the more I liked it honestly.

**Verdict:** My tool of choice for any database work. I can't go back to a NodeJS runtime anymore.

## 2022

### My current stack

#### Hosting

- **Cloud Provider:** Amazon Web Services (AWS)
- **Infrastructure As Code:** AWS SAM, CloudFormation as backup
- **Auth:** Firebase Auth

#### Frontend

- **Frontend:** React for SPA, Hugo, Nunjucks or NextJS SSR
- **CDN/Hosting:** AWS CloudFront + S3, deployed with SAWS SAM
- **Styling:** SAAS
- **Testing:** Unit Testing - Sinon/Chai. E2E Testing - Cypress

#### Backend
- **Stack:** Java/Spring Boot
- **ORM:** Spring Data
- **Database:** DynamoDB/MySQL
- **Small Services:** AWS Lambda - TypeScript, deployed with AWS SAM
- **Hosting:** AWS ECS, deployed with AWS SAM
- **Testing:** JUnit5/Mockito

#### Utility
- **CI/CD:** GitHub Actions
- **Scripting:** Python
- **OS Configuration:** Ansible
- **Static Analysis:** Java/TypeScript - SonarQube
- **Payments:** Stripe Checkout or Stripe Connected Accounts.

### I'd like to learn more

I've done some tutorials in the following but I'd like to really deep dive them with a real project. These could potentially change my preferred stack.

- Tailwind - SAAS Replacement?
- Go - Python/Java Replacement?
- Flutter - Interested in more mobile development.
- AWS CDK - SAM/CloudFormation replacement?
- Rust/Kotlin - Java Replacement?
