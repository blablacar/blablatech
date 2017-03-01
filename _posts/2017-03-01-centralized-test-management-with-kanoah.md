---
layout:      post
title:       "Centralized Test Management with Kanoah"
tags:        [testing, automation]
authors:     [alexis-gauthiez]
description: A solution to scattered manual and automating testing.
---

After having [automated end-to-end tests for our mobile apps](http://blablatech.com/blog/one-model-to-rule-them-all) running for a few months with Qualcium, we realized something was missing. We were lacking some kind of keystone which could really kick-off the adoption of our automated test solution throughout the company. We identified a few impediments:

- We didn't have an easy way to reference our corpus of existing test scenarios;
- It wasn't clear how to transition from manual to automated tests;
- Launching automated tests was not trivial…
- … and so was analyzing test results.

As a result, our new framework effectively did not enable the Quality Assurance team to spend less time on regression testing. Although it was able to uncover product defects on a daily basis, an actual human being could not clearly tell which user stories were actually covered. As a consequence, **failing automated test runs gave us some insight about the features under test, but passing ones didn't provide much meaningful information about product quality**.

We had to introduce a new tool in order to track our manual and automated test scenarios together in one place.

## Kanoah: the chosen one

For manual and automated tests to coexist in a seamless fashion, we had to push in both directions: make some adjustments on how we perform and document manual tests and have automated ones transparently integrate with them.

Once we decided that our goal was to centralize our collection of written test cases, both manual and automated, it became clear that we we needed to adopt a true [test management tool](https://en.wikipedia.org/wiki/Test_management#Test_management_tools).

At the time, we were using distinct tools to document and maintain our manual and automated tests. For documenting and performing manual testing, we were relying on Google Sheets. A great general-purpose tool to start with but a mess to deal with when it comes to scaling: reusing tests is basically copy-pasting, searching for existing cases is almost impossible, and we could not clearly describe nor follow test execution result trends. Not to mention the fact that integrating it with an issue tracker or a continuous integration server is, well... complicated.
For automated test coverage documentation, we started using JIRA issues. One task for each scenario, with a custom type and configuration so we could specify test setup, exercise, verifications and teardown separately.

We identified two types of products that could do the job: standalone web applications and JIRA add-ons. In the end we benchmarked over 10 different tools. The first set included commercial products, [TestRail](http://www.gurock.com/testrail/) being the most interesting according to our tests and based on our use case. There are also open source projects, such as [Squash](http://www.squashtest.org/), which offer a decent service at no cost but require some setup and configuration. Soon enough, going for a JIRA add-on became obvious. At the end of the day, things happen on JIRA, and there’s no better place for our test scenarios to live.

Among [the variety of add-ons on the Atlassian marketplace](https://marketplace.atlassian.com/search?application=jira&category=Testing+%26+QA&cost=&hosting=&marketingLabel=&q=), we think [Kanoah Tests](https://www.kanoah.com/) fits BlaBlaCar the best. It gives product managers and developers the visibility they need on end-to-end test coverage and results as it naturally integrates into the Detail View of JIRA issues. Moreover, it provides a REST API at no extra cost. It also fulfills our needs for organizing test cases with labels and components.

We logically decided to go all-in and give Kanoah a full test run, fully adopting it into the daily workflows of the entire QA team for an extended trial period, giving training, and migrating our old test plans. But there was still work to be done.

## Integrating Kanoah Tests with our automated test infrastructure

Bamboo is the continuous integration solution used by the engineering team at BlaBlaCar, so we thought it could serve on its own as the defacto launcher for Qualcium tests. This is definitely not the case. Even though it provides many powerful features to facilitate automated testing, particularly adapted to unit and other tests that live in the same repository as the code under test, it is not as customizable and accessible as we'd like for our stakeholders.

In order to fully centralize our testing activities, we wanted Qualcium launches to create reports directly in Kanoah Tests, more precisely through its Test Player. In short, the Kanoah Test Player is an interface through which test case execution information can be reported, including test progress, results, attachments and more. Thanks to the Kanoah Tests REST API, this reporting can even be done programmatically.

To quickly demonstrate our idea, we created a small executable which takes a Test Run ID as an argument and reads [Mocha’s JSON stream output](https://mochajs.org/#json-stream). It retrieves the list of test scenarios marked as automated from Kanoah and reports them as “In Progress” on the Test Player as soon as the entire test suite starts. As each scenario completes, it is marked as “Pass” or “Fail” (commented with a stack trace in the case of a failure) and attached with screenshots. Once tests have completed, those that have not been executed for one reason or another appear as “Blocked” on Kanoah Tests. This can happen if something in the test environment fails by no fault of the scenario itself, or if the test suite's implementation is somehow out of sync with the test management tool.
We recently turned this proof-of-concept into a bonafide [Mocha reporter](https://mochajs.org/#reporters) for Kanoah.

We also created a web interface to launch a test suite simply by providing a Kanoah Test Run ID, which we reluctantly named *Koalcium* (yes, it is a really bad name 🐨).

![Koalcium in action](/images/2017-03-01-centralized-test-management-with-kanoah/koalcium.gif)

The interface is served by an application which checks and resolves the native app build URL, fetches relevant test case IDs and runs a Bamboo job when the form is submitted.

## Impact so far

We are currently driving the adoption of Kanoah Tests within our organization. Thanks to a team effort, we've identified and aligned on some early best practices around how we use the test management tool. Making the transition kickstarted some insightful discussions about test coverage, reporting, and the overall workflows we use for assuring the quality of our products.

![Stack](/images/2017-03-01-centralized-test-management-with-kanoah/integration.png)

Now the challenge is to keep our automated test suite in sync with scenarios that are referenced on Kanoah, and to gather more feedback as we make Koalcium our team's main interface for launching tests on our mobile apps.
