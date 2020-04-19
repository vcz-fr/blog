---
layout: blog

title: "Automating Postman"
categories: ["Development"]
tags: ["repost", "gekko", "api", "cicd"]

date: "2020-04-14"
---

## The Test Pyramid

From the outside, testing can appear like a non-productive activity. In other words, when someone writes or maintains
tests, they do not contribute directly to the service but rather to the code that verifies that it works as intended.
Since the service is developed to follow a specification, why is it important to check that it follows it, then?

Code is brittle. The larger the project, the wider a small change may spread on its surface. As a developer, one may
develop expertise on the hundreds if not the thousands of the lines of code they encounter on a daily basis. While that
helps locating the components to change to implement certain features or locating the relative origin of an error, it
does not prevent errors from happening in the first place.

> Well executed, Testing can thread safety nets for services.

There are all sorts of tests out there. Some will analyze a few lines at best, others a module or two at a time and
others will call chucks of the whole application, if not the ecosystem. It is possible to test for code syntax, feature
completeness, avoiding regressions, code minimalism, performance, thread and memory safety, security, etc. Before
starting out on this topic, it is necessary to make sure the team knows about its needs.

Typically, tests should verify the following:
1. The application compiles;
1. The application works: it does what it claims;
1. The application works **well**: it is performant, secure, maintainable, etc.

That last element represents the concerns the team might have expressed while the others are common to all projects. It
is still possible to add steps before and after the aforementioned ones like syntax checking, also known as _linting_,
validating dependencies, package size, etc. These are all elements to consider as the structure hereby presented is in
no way complete; it is merely a framework of a classical Continuous Integration and Delivery pipeline.

Validating that an application is performant or maintainable means that there exists a way to extract and compare KPIs
somewhere. Testing its security or safety can rely on the detection of known anti-patterns. Checking the formatting
depends on known configuration, namely _linter_ rules. Everything here is clearly defined and repeatable across
projects. However, this cannot be said of the assessment that an application does what it claims.

The [Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html) is a construct that is often used to explain the way
a service should be tested: lots of small, fast and inexpensive **Unit tests** to validate the foundational layer, the
code that has a small and clearly defined responsibility, then less moderate **Integration tests** to prove these bricks
can interact and work together and finally less slow and expensive **End-To-End tests** that run real life scenarios
across the service. A common distribution for these tests is 70% - 20% - 10%, which means 10% of the test cases should
relate to the whole application.

Making sure that the smaller components of the application work before testing larger components is crucial. End
users will report errors that cannot be directly translated to patches, that is why their reported errors must be
converted to tests for all the layers and especially the related code units. **Unit tests** are fast and can be run in
parallel, which makes them perfect to for local execution.

A test suite should indeed not be only relied upon during Continuous Integration and Continuous Delivery pipelines. Any
developer should be able to pick it up and run it partially or fully to validate their code changes locally. Version
Control Systems may feature some kind of workflow event mechanism to even automate this part. In Git, there are [Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks).

## In the subject of APIs

API design, like any engineering task, follows trends which [we wrote about a while ago -in French-](https://www.gekko.fr/les-bonnes-pratiques-a-suivre-pour-developper-des-apis-rest/).
Swagger and more generally OpenAPI is one of them, as is using services and applications to help developers testing
their own or third party APIs. Both subjects seem orthogonal, like placed on the two ends of the spectrum with design on
one end and testing on the other but are they?

OpenAPI, for instance, allows defining the format of the inputs and outputs for each method of each endpoint. This is
notably the validation that [Amazon API Gateway](https://aws.amazon.com/api-gateway/) applies when processing requests
and responses for REST APIs. Consequently, it is wise to add as much information as possible right from the beginning,
that is the API specification.

Once one gets familiar with the specification language, using tools can become beneficial to increasing productivity,
getting started faster, visualizing current work and shortening the feedback loop. The right tool can reduce friction. [OpenAPI.tools](https://openapi.tools/)
contains a list of such tools that help with the design, integration and test of OpenAPI APIs. For this article, we will
be concentrating on one of the most popular of the list, [Postman](https://www.postman.com/).

## Postman

Postman is known and loved by the community. It helps with debugging APIs during development. The requests can be
bundled together into a collection and this collection can be documented and shared across the team to share knowledge.

Less is known about the fact that API responses can be tested using a human-like syntax. This is called BDD for
"Behavior Driven Development" and, as the [documentation states](https://learning.postman.com/docs/postman/scripts/test-scripts/),
it is based on [ChaiJS](https://www.chaijs.com/api/bdd/). In addition, it is possible to test the response body schema,
run a script before the request or before a collection of requests. All of this can be exported to a Postman collection
file. If the API specification is ready and complete, it can be imported to a collection that will contain the resources,
routes, parameters pre-filled so that one can get started faster with the tests!

This article is not about using Postman, though. There are numerous ways to get started: documentations, tutorials,
conferences, blog posts, trial and error, etc. Being confident with the possibilities offered by the platform and knowing how to write tests both fundamentally and with Postman is a prerequisite when the need for automation arises.

## Automation

In its core, automation translates to finding ways to run manual tasks automatically. Scheduling and fallbacks are
secondary to the topic. Automation happens when value brought by a human is minimal. For instance, if a human clicks on
a button to deploy an application and moves on with their day, this action can be automated. If they have been
instructed to click on the button at a specific time interval, this action could have been scheduled in the first place.
Now if the deployment requires attention, there is value in having somebody following it but this value ought to be
minimized with the output of more observable metrics and better practices for the application.

