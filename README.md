<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->

<a name="readme-top"></a>

<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Don't forget to give the project a star!
*** Thanks again! Now go create something AMAZING! :D
-->

<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->

[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url] [![MIT License][license-shield]][license-url] [![LinkedIn][linkedin-shield]][linkedin-url] <!-- PROJECT LOGO --> <br /> <div align="center"> <a href="https://github.com/LukeMacdonald/githubactions-learning"> <img src="img/logo.png" alt="Logo" width="80" height="80"> </a> <h3 align="center">CI Pipeline with Github Actions</h3> <p align="center"> <br />
<a href="https://github.com/LukeMacdonald/githubactions-learning"><strong>Explore the docs »</strong></a>
<br />
<br />

  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
        <li><a href="#built-with">Built With</a></li>
    <li>
      <a href="#analysis-of-problem">Analysis of Problem</a>
    </li>
    <li>
            <a href="#explanation-of-solution">Explanation of Solution</a>
            <ul>
                <li><a href="#github-flow-of-solution-(order-of-branch/feature-creation)">GitHub Flow of Solution (Order of Branch/Feature Creation)</a></li>
                <li><a href="#flow-of-ci-pipeline-workflow-jobs">Flow of CI Pipeline Workflow Jobs</a>Flow of CI Pipeline Workflow Jobs</li>
                <li><a href="#justification-of-pipeline">Justification of Pipeline</a></li>
        </ul>
    </li>
    <li><a href="#successful-scenario">Successful Scenario</a></li>
    <li><a href="#failure-scenarios">Failure Scenarios</a></li>
    <li><a href="#contact">Contact</a></li>

  </ol>
</details>

<!-- ABOUT THE PROJECT -->

### Built With

<div style="column-count: 2; -webkit-column-count: 2 -moz-column-count: 2;">

- [![GitHubActions][GitHubActions-badge]][GitHubActions-url]

</div>

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->

## Analysis of Problem

Alpine Inc has recently developed a web application designed for users to create simple Notes. The application was built
using NodeJs, Express and MongoDB.

The source code for is application is solely built and deployed manually from the lead developer's laptop. Pete has
taken a recent leave of absence from work which has created multiple issues for the team:

- a new feature designed for one of the company's largest clients was not released.
- Increase in the number of customer support calls caused by the introduction of bugs into production.
- The team's morale has been reduced as they have had to work overtime to manage this issue.
- Alpine Inc.'s revenue has been impacted.

A strategy has been proposed by Alpine Inc to automate the build and testing processes of the application so that they
can effectively:

- reduce dependency on the Lead Developer
- Pick up bugs and defects as soon as possible before they can be pushed into production.
- reduce the strain on the teams and increase morale.
- improve the reliability of releases.

## Explanation of Solution

### GitHub Flow of Solution (Order of Branch/Feature Creation)

```mermaid
gitGraph
    commit id: "initial commit"
    commit
    branch feature-lint-testing
    checkout feature-lint-testing
    commit id: "add lint test"
    commit id: "fixed linting bugs"
    checkout main
    merge feature-lint-testing
    commit
    branch feature-unit-test
    checkout feature-unit-test
    commit id: "add unit test"
    commit id: "fixed unit test bugs"
    checkout main
    merge feature-unit-test
    commit
    branch feature-integration-test
    checkout feature-integration-test
    commit id: "add integration test"
    commit id: "fixed integration bugs"
    checkout main
    merge feature-integration-test
    commit
    branch feature-e2e-test
    checkout feature-e2e-test
    commit id: "added e2e test"
    commit id: "fixed e2e test bugs"
    checkout main
    merge feature-e2e-test
    commit
    branch feature-deploy-artefact
    checkout feature-deploy-artefact
    commit
    commit
    checkout main
    merge feature-deploy-artefact
    commit
    branch feature-logging
    checkout feature-logging
    commit
    commit
    checkout main
    merge feature-logging
    commit
    branch feature-sast-testing
    checkout feature-sast-testing
    commit
    commit
    checkout main
    merge feature-sast-testing
    commit id: "final commit"

```

When developing my pipeline, I decided to split each test into its own jobs and then decided that I could create a new
feature branch for each of these jobs.

The flow of the jobs went in order of dependence.

- For example, if linting were to fail, then it is most likely that unit tests would fail. Therefore, I created the job
  for linting before creating the job for unit tests.
- To save time on execution if one of the jobs were to fail, the pipeline jobs will run in serial rather than parallel
  because if all jobs were to run in parallel if unit tests were to fail then it is most likely that e2e tests will also
  fail, but we would still have to wait for the e2e testing to finishing executing become the workflow would exit.
  - Therefore, time is now saved because if the unit test fails then the workflow will immediately exit and the e2e testing
    won’t even begin running.

### Flow of CI Pipeline Workflow Jobs

```mermaid
  flowchart LR;
A[ Setup ]
B[ Lint Testing]
G[SAST]
C[ Unit Testing ]
D[Integration Testing ]
E[ E2E Testing ]
F[ Deploy ]
A --> B
B --> G
G --> C
C --> D
D --> E
E --> F

```

1. **Setup**
   - The first step in the workflow is the setup job. This is where the workflow will check its cache and see if the
     dependencies have already been cached, if so then the dependencies cache is used to restore the dependencies, if not then
     the dependencies are installed and then cached.
2. **Lint Testing**
   - The first set after the initial setup job is lint testing. Lint testing is run first because its operations are the
     ones least dependent and influenced by other jobs. I.e. there are no other jobs that if they failed then that
     means that linting would also fail.
   - Linting is the process of checking the code for syntax errors, style violations, and other common programming mistakes.
   - A condition is placed on the job so that if **_npm run test-lint_** fails then they workflow will be stopped and a
     file that includes all the output from the lint testing will be uploaded as an artifact for the developer to view
     and understand why the test failed.
     - File is only uploaded if the test fails, if it succeeds then no file is uploaded.
3. **SAST Testing**
   - The next job after lint testing is sast testing. Static Application Security Testing is a type of security testing that
     analyzes the source code or compiled version of an application to identify potential security vulnerabilities
   - nodejsscan was the tool used by this pipeline for SAST
   - SAST is performed after linting because it helps to ensure that the code is correct and consistent before testing the security.
4. **Unit Testing**
   - The next job after sast testing is unit testing. Unit testing is a type of testing that focuses on testing individual units
     or components of the code in isolation.
   - Unit testing is performed after sast because it helps to ensure that there are no security vulnerabilities before testing each
     of the components in unit testing.
   - Unit testing is run before integration and E2E testing because if unit testing was to
     fail then both of those tests are likely to fail as well, therefore it saves on time to run unit testing first.
   - When running the unit test a code coverage report is generated by jest and at the end of the job that code
     coverage report is uploaded as an artifact.
   - Similar to lint testing if the job fails then a log of the unit tests will be uploaded as an artifact for the
     developer to use to determine what went wrong and what actions can be taken to repair the mistakes.
5. **Integration Testing**
   - The next job after unit testing is integration testing. Integration testing is a type of testing that checks if
     the different components of the code work together correctly. It's often done after unit testing because while
     unit testing tests the individual components, integration testing tests how these components work together.
     Therefore, if a unit test were to fail then the integration test would fail aswell.
   - Exactly like with unit testing when running the integration test a code coverage report is generated by jest and
     at the end of the job that code coverage report is uploaded as an artifact.
   - Also, exactly like unit testing and lint testing if the job fails then a log of the unit tests will be uploaded as
     an artifact for the developer to use to determine what went wrong and what actions can be taken to repair the
     mistakes.
6. **End to End (E2E) Testing**
   - The final testing job to be run is e2e testing. This is because this test is the one most likely to be impacted by
     all the others (i.e. if one of those tests failed its most likely this one would fail) therefore making it more
     time cost-effective to place this test last.
   - End-to-End testing is a type of testing that checks the entire system or application to ensure that it works as expected
     from start to finish. It involves testing the user interface, the back-end code, and all the components in between.
   - Similar to all other tests if this test were to fail when running this would produce a log file for the developer
     to download which can help them understand what went wrong with the code.
7. **Deployment (if on main branch)**
   - This is the final job of the workflow and this job will only run when a change is made to the main branch as
     compared to all the other jobs which run on all branches.
   - In this job is run last after all the testing because it involves packaging up the application using the \* \***\*\*\*\*\*\***npm pack\***\*\*\*\*\*\*\*** command and then that will be uploaded as an artefact that can be used for
     deployment.
   - It is important that the deployment job only runs if all other jobs correctly run and ensure so bugs are in the
     application.

### Justification of Pipeline

1. Cache Dependencies
   - As each job is run on a different system this would usually require each job to have to install the dependencies of
     the application. This however can cause the execution of each job to grow significantly. Therefore, in order to
     manage the pipelines workflow time for effectively a setup job has been included in the pipeline which installs
     the dependencies needed in the application and caches those dependencies which can then be accessed by the other
     jobs without them having to install them.
     - The process of installing and caching dependencies only has to occur once (or for the first workflow after a
       dependency change occurred) this means that even the setup job won’t always have to install the dependencies
       and instead can check if a cache exists instead and restore dependencies based on that cache.
2. Code Coverage Percentage
   - After some research it was found that 80% or higher code coverage is generally aim for, therefore for global
     thresholds for code coverage was set to 80%.
   - Code coverage reports were only generated by unit testing and integration testing when the job successfully completed.
3. Changes made to configuration files
   - When running the e2e testing there was an error in the tests. After some investigation it was found that one of
     the browser hosts “webkit” AKA Desktop Safari was causing the error as it was not compatible with the latest
     version of ubuntu used to run the workflow. Therefore, this host was removed as one of the testing browsers from
     the playwright configuration file.
   - Inside the **_jest.config.js_** code was added to allow for a log file which exported all the output from the jest
     testing into a file which could be accessed by GitHub Actions to upload in case of a failure so that the developer
     could determine what went wrong.

## Successful Scenario

<img src="img/successful.png">

- Produces a code coverage report for unit and integration testing
- Produces a packaged version of the app

## Failure Scenarios

NOTE: These failure scenarios were run before sast testing was included. Therefore the screenshots do not include sast in the workflow.

### Linting Failure

<img src="img/lint-failure-scenario.png">

- Produces a text file containing why linting failed
- All jobs after lint job were cancelled and workflow exited.

### Unit Test Failure Scenario

<img src="img/unit-failure-scenario.png">

- Produces a xml containing why unit test failed
- All jobs after unit testing job were cancelled and workflow exited.

### Integration Test Failure Scenario

<img src="img/integration-failure-scenario.png">

- Produces a xml containing why integration test failed, also produced report on code coverage of unit testing
- All jobs after integration testing job were cancelled and workflow exited.

### E2E Test Failure Scenario

<img src="img/e2e-failure-scenario.png">

- Produces an html file containing why e2e test failed, and code coverage reports for unit and integration testing.
- All jobs after e2e testing job were cancelled and workflow exited.

## Contact

Luke Macdonald - lukemacdonald21@gmail.com.com

Project Link: [https://github.com/LukeMacdonald/githubactions-learning ](https://github.com/LukeMacdonald/githubactions-learning)

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->

[contributors-shield]: https://img.shields.io/github/contributors/LukeMacdonald/githubactions-learning.svg?style=for-the-badge
[contributors-url]: https://github.com/LukeMacdonald/githubactions-learning/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/LukeMacdonald/githubactions-learning.svg?style=for-the-badge
[forks-url]: https://github.com/LukeMacdonald/githubactions-learning/network/members
[stars-shield]: https://img.shields.io/github/stars/LukeMacdonald/githubactions-learning.svg?style=for-the-badge
[stars-url]: https://github.com/LukeMacdonald/githubactions-learning/stargazers
[issues-shield]: https://img.shields.io/github/issues/LukeMacdonald/githubactions-learning.svg?style=for-the-badge
[issues-url]: https://github.com/LukeMacdonald/githubactions-learning/issues
[license-shield]: https://img.shields.io/github/license/LukeMacdonald/githubactions-learning.svg?style=for-the-badge
[license-url]: https://github.com/LukeMacdonald/githubactions-learning/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/luke-macdonald-292a4a208
[product-screenshot]: images/screenshot.png
[GitHubActions-badge]: https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white
[GithubActions-url]: https://github.com/features/actions
