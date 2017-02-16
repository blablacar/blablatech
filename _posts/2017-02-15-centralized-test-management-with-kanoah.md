---
layout:      post
title:       "Centralized Test Management with Kanoah"
tags:        [testing, automation]
authors:     [alexis-gauthiez]
description: A solution to scattered manual and automating testing.
---

After having [automated end-to-end tests for our mobile apps](http://blablatech.com/blog/one-model-to-rule-them-all) running for a few months, we realized something was missing. We were lacking some kind of keystone which could really kick-off the adoption of our automated test suite throughout the company. We identified a few impediments:
- we do not have a tool to reference our existing tests;
- it is not clear how to transition from manual to automated tests;
- launching automated tests is not trivial…
- … and so is analyzing the results.

As a result our work effectively did not enable the Quality Assurance team to spend less time on regression testing. Although the automated test suite was able to uncover product issues on a daily basis, an actual human being could not clearly tell which user stories were actually covered. As a consequence, **failing runs were insightful but passing ones were not saying much**.

We had to introduce a new tool in order to bring manual and automated testing together. We know our products are evolving blazingly fast, thus we will always have some manual validation going on.

## Kanoah: the chosen one

In order to have manual and automated tests coexist in a seamless fashion, we had to push in both directions: make some adjustments on how we perform and document manual tests and have automated ones transparently integrate with them.

Prior to our benchmark, we were using distinct tools to document and maintain our manual and automated tests. On the one hand, we were relying on Google Sheets for documenting and performing manual testing. A great tool to start with but a mess to deal with when it comes to scaling: reusing tests is basically copy pasting, searching amongst existing cases is almost impossible and we could not give them a clear description nor follow their execution result trends. Not to mention the fact that integrating with an issue tracker or a continuous integration server is… Well, complicated.

On the other hand, we initiated automated test coverage documentation using JIRA issues. One task for each automated test case, with a custom home-made configuration so we could specify test setup, exercise, verifications and teardown separately.

Because our goal was to centralize our collection of written test cases, both manual and automated, [test management tools](https://en.wikipedia.org/wiki/Test_management#Test_management_tools) is what we were looking for. We identified 2 types of products that could do the job: standalone web applications and JIRA add-ons. In the end we benchmarked over 10 different tools. The first set includes commercial products, [TestRail](http://www.gurock.com/testrail/) being the most interesting according to our tests and based on our use case. There are also open-source projects, such as [Squash](http://www.squashtest.org/), which offer a decent service at no cost but some setup and configuration. Soon enough, going for a JIRA add-on became obvious. At the end of the day, things happen on JIRA and there’s no better place for testing to happen.

Amongst [the variety of add-ons on the Atlassian marketplace](https://marketplace.atlassian.com/search?application=jira&category=Testing+%26+QA), we think Kanoah Tests fits BlaBlaCar the best. It gives product managers and developers the visibility they need on testing as it flawlessly integrates with the Issue Detail View. Moreover, it provides a REST API for no extra cost. It fulfills our needs for organizing test cases with labels and components.

## Integrating Kanoah Tests with our automated test infrastructure

We know our tool has some weaknesses in terms of reporting but the real break was Bamboo. We thought Bamboo by itself could do the job of a test runner: it is definitely not the case.
Bamboo is the continuous integration server we use at BlaBlaCar, and, even though it provides some features that help with automated testing, it is not as customizable (e.g. customized runs are pretty basic) and accessible as we want for the entire team.

In order to fully centralize our testing activities, we needed our automated testing tool to report on Kanoah Tests, more precisely on its Test Player. In short, the Kanoah Test Player is an interface on which test cases execution can be reported, including the progress, tests results, attachments and more. Thanks to the Kanoah Tests REST API, reporting can be done programmatically.

To quickly demonstrate our idea, we created a small executable which takes a Test Run ID as argument and reads [Mocha’s JSON stream](https://mochajs.org/#json-stream) output. It retrieves the list of test cases marked as automated from Kanoah and reports them as “In Progress” on the Test Player as soon as the entire test suite starts. As automated test cases finish, they are marked with “Pass” or “Fail”, commented with the stack trace on failure and attached with screenshots. Once automated tests are done, tests that have not been executed appear as “Blocked” on Kanoah Tests. It can happen when something went wrong at some point but the test case itself is not faulty or if the automated test suite implementation is not in sync with the test management tool.
We are considering making it an actual Mocha reporter.

Also, we created a web interface to launch an automated test suite giving a Kanoah Test Run ID: Koalcium (it is a really bad name by the way).

![Koalcium in action](/images/2017-02-15-centralized-test-management-with-kanoah/koalcium.gif)

The interface is served by an application which checks and resolves the native app build URL, fetches relevant test cases IDs and runs a Bamboo job when the form is submitted.

## Impact so far

We are currently driving the adoption of Kanoah Tests within our organization. Thanks to the team efforts, we aligned on best practices when using the test management tool. Making the transition kickstarted some insightful discussions about how we cover our products with tests.

![Stack](/images/2017-02-15-centralized-test-management-with-kanoah/integration.png)
