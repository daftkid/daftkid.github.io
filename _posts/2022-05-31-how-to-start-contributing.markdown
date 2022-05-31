---
title:   "How to start contributing to Open Source"
date:    2022-05-31 18:30:00 +0300
tags:    personal open-source contribution
excerpt: ""
---

Hi!

During the last 2 or 3 years, I was really interested in open source projects and moreover about doing something to make
them better products. I'm using a lot of open source tools in my day-to-day work, some of them are already de-facto 
standards in the DevOps field.

Historically, one of the first open source tools I met and used was [Terraform][tf]. I started using Terraform in my day-to-day
tasks and activities 5 years ago being a Junior Systems Engineer when Terraform version `0.9.x` just was released.

I remember tons of excitement that overflew me. This tool was really magical - you're writing some code in a text file
following a particular style, run two or three commands in a terminal - and Voila! - everything is created in AWS. And
moreover, you can play with these resources, reconfigure them, and so on.

<figure>
<img src="https://www.datocms-assets.com/2885/1620155439-blog-library-product-terraform-aws-logomarks.jpg">
<figcaption>Credits to hashicorp.com</figcaption>
</figure>

One day I realized that Terraform is not able to create some very new resource for me (don't actually remember what it 
was) because this resource was released just a couple of days ago within AWS and Terraform did not have any idea of what 
to do with it.

I already knew that Terraform is an open source tool but did not quite understand what does it mean. Well, the code itself
is not a secret, it can be shared and used by anyone, I can see it in Github repo. I asked my more mature colleagues about 
possible ways to bypass this "limitation" in Terraform and they told me:

*"You know what? You can try to add this resource support by yourself!"*

It was a great idea actually, except for one significant thing - I did not have any clue about writing Go code.

That time I just gave up and waited a couple of weeks until someone else just did it for me... and for other guys who were
interested in this resource to be managed by Terraform. But the idea of contribution stuck deep in my mind and I just got
another tech-related dream.

I came back to this idea each half a year, however, I did not have either too much motivation or too much time for sitting 
down and studying, for example, GoLang or Terraform internals.

> **Tip 0**: Start from contribution to the tool that you are using on daily basis, loves and understands.

In my imagination, Terraform (including AWS Provider) was such a complicated software that I did not know how to start.
This complexity was killing my motivation. In addition, there were a lot of things not directly related to Terraform,
but to Software development in general in which I felt weak as I'm not a "native" Software Developer. I was using Python 
before for my DevOps-related tasks but Python is not a Go 🙃 and is deadly different to Go.

Also, I had a feeling that Terraform and its providers are developed by Gods 😇 😎, and I'm "too small" 😰 to participate 
in these tools.

> **Tip 1**: All tools are made by people. Each contribution is valuable as it moves the tool forward into a better
state. Do not feel like an impostor, just contribute!


# Obstacle 1: You have to know programming language

Yeah, that's true that you can contribute to some tool codebase without knowing the language - for example, you can fix
typos in the tool's documentation or try to copy/paste/tailor some code blocks. Unfortunately, It does not work like that.
Documentation fixes are 100% important, but they do not let you make the tool better.

That's why in my free time I tried to pass the GoLang online course - you know, some kind of "Learn Go in 3 hours".
Surprisingly, it turned out that Go is not so difficult.

Okay... I pulled the source code of Terraform Provider for AWS and tried to build all the stuff in my head. Failed on
the second file 😂. It's not enough to just know the syntax of a particular Programming Language, you have to have at 
least some basic understanding about what is going on under the hood of your favorite tool.

> **Tip 2**: Most popular programming languages for DevOps tools are **JS** or **Go**. They are dramatically different and
are used for completely different purposes so it's better to know in advance which tools you are going to contribute to.

<figure>
<img src="https://cdn-cejjk.nitrocdn.com/RqgODXEbgAKqRQayFqykFbKTSwlhEAKL/assets/static/optimized/rev-e623d2d/wp-content/uploads/2022/02/GOLang-Vs-NodeJS-Which-is-Best-For-Backend-Development.jpg">
<figcaption>Credits to hireindianprogrammers.com</figcaption>
</figure>

I have found really useful these online courses related to Go:
- [Master Go Programming at Udemy](https://www.udemy.com/course/master-go-programming-complete-golang-bootcamp/)
- [REST based microservices API evelopment in Go at Udemy](https://www.udemy.com/course/rest-based-microservices-api-development-in-go-lang/)
- [Design patterns in Go at Udemy](https://www.udemy.com/course/design-patterns-go/)


# Obstacle 2: It's not enough to have some hands-on experience in the tool's usage

I have pretty extensive knowledge and experience in Terraform (including `workspaces`, `remote state files`, and even code 
migration to newer terraform versions) but it was not enough (at least for me) to contribute yet.

I spent another couple of sleepless nights running through Terraform Provider Development guides. As a result, I realized 
that Terraform Provider internals are not related to Terraform code itself at all. What you write in `tf` files and in
`go` are extremely different things.

But I've got the main idea - the provider just copies SDK calls and is an implementation of a simple CRUD model. And it's
just needed to put some piece of logic (mostly related to types and structs conversion to some other forms) in the provider
code and extend resource schema.

> **Tip 3**: In their majority, almost all the tools are based on deadly simple principles. Don't be scared by the tool's 
complexity, there is always a way to understand some parts or subsystems of the tool and start from them. For example,
in Terraform Provider for AWS you can start to investigate code from some very simple resources such as [Macie Member Account Association][macie].


# Obstacle 3: Just reading and "understanding" is not enough

You need to start code and propose your Pull Requests to code owners.

But keep in mind that each tool is a standalone piece of software and it has a lot of things you need to know and touch in
order to do some useful job. For example, if we are talking about Terraform providers, you need to get familiar with
Terraform provider acceptance testing and so on.

The good news here - each popular open source tool has tons of contribution guides, checklists, demos, examples, and so on.
You just need to put some time and motivation into it!

Here is just a brief list of useful resources for Terraform Provider for AWS:
- [terraform-provider-scaffolding][tf-scaff]
- [contribution checklist][checklist]
- [acceptance tests][acc-test]

> **Tip 4**: Many of the repos have issues tagged with labels such as `good_first_issue` (for example, [here][gfi] for Terraform
Provider for AWS). You can start looking at them first, they should be relatively simple to do.

![good first issue](/assets/images/good-first-issue.png)


# Newer ending story

Each popular tool is extended, fixed, and featured with new code each day or even hour. Code is very volatile, a lot of
other guys contribute each minute. For example, Terraform Provider for AWS has about 200 PRs merged on a weekly basis.

I tried to start my contribution three times... And each time I failed as did not have enough to even start a PR.
After some time something was dramatically changed inside the tool (like Terraform SDK, general contribution approach, 
and so on) and each time it's needed to start almost from the beginning.

> **Tip 5**: Don't be afraid of doing something wrong. In most cases, it's not possible to do something good (or at least 
something that compiles) right away. Be sure that other guys from the community will help you and answer most of your
questions (even dummy ones 😀).


# My contributions

I have created a [separate space][opensource] within my blog and here I gonna post all my contributions to open source
projects. In the very beginning, most of them will be related to [Terraform Provider for AWS][tf-aws] as I'm actively 
working on it. But hopefully in the future I'll play with some other stuff.

[tf]: https://www.terraform.io/
[tf-aws]: https://github.com/hashicorp/terraform-provider-aws
[gfi]: https://github.com/hashicorp/terraform-provider-aws/labels/good%20first%20issue
[macie]: https://github.com/hashicorp/terraform-provider-aws/blob/main/internal/service/macie/member_account_association.go

[tf-scaff]: https://github.com/hashicorp/terraform-provider-scaffolding
[checklist]: https://github.com/hashicorp/terraform-provider-aws/blob/main/docs/contributing/contribution-checklists.md
[acc-test]: https://github.com/hashicorp/terraform-provider-aws/blob/main/docs/contributing/running-and-writing-acceptance-tests.md

[opensource]: /opensource/