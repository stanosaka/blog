---
title: agile
date: 2020-02-24 20:39:06
tags:
  - agile
  - software development 
  - agile development
  - devops
---
# Agile is a movement.
Movement: a group of people working together to advance their shared political, social, or artistic ideas.

## Early agile methodologies
- 1995: Scrum
    - Iterative cycles of work
    - Emphasis on cross-functional collaboration
- Mid-90s: Crystal
    - Emphasis on adaptability and "stretch-to-fit" process
- 1990: XP
    - Short, iterative cycles of work
    - Emphasis on collaboration between developers

![agile manifesto](https://i.imgur.com/MjJqXdR.jpg)
**Agile Values**
- Increase cross-functional collaboration
- Minimize works in progress (sepecs, etc)
- Work in iterative cycles to incorporate change and get frequent feedback
- Make time to reflect on how you work

---
# Agile Development
**born-on-the-cloud model**: cloud as the primary platform for both consumed and delivered services
Agility to get projects up and running quickly
streamlined process by reducing transitions from Development to Operations

Java EE application with a Minimum Viable Product (MVP)
setup the cloud infrastructure and Toolchain using the Born-on-the-cloud approach

Hypothesis-driven development

ranked backlog, user stories, story points
tracking and delivering in Timeboxed iterations

**backlog** is a prioritized list of features (User Stores) waiting to be scheduled and implemented

Note that among other things one of the most important responsibilities of the product owner role in Agile development methodology is to keep the backlog ranked by priority, and he or she can set the priorities by simply dragging the issues and ordering them, regardless of when they were added to the backlog

User stories: who, what, why
story points and planning poker

**user story** is an informal, natural language description of a chunk of functionality that is of value from an end-user perspective
As a <who>, I want <what> so that <why>

**Story Points** are estimates of effort as influenced by the amount of work, complexity, risk, and uncertainty.
- (Adapted) Fibonacci sequence: 0,1,2,3,5,8,13,20,40,100

**Planning Poker** is a consensus-based, gamified technique for estimating
Timeboxing refers to the act of putting strict time boundaries around an action for activity


time boxed interrogations along with story points are important to measure the team velocity over time. For example you should track the total amount of story points that team delivers each iteration, and this way you'll be able to determine on average the team velocity metric, or in other words how many story points on average the team is able to deliver. 

Architecture breakdown
- User interface (html/css/javascript)
- CRUD Service(REST APIs with JAX-RS)
- Database(NoSQL)

**Test-Driven Development (TDD)** is a technique for building software by writing automate test cases before writing any code
why: focus on the outcome; helps to manage risk; enables fast iterations and continous integration; keeps the code clear, simple and testable;
grater confidence to make applicaton changes; documentation of how the system works

How:add a test->run all tests and see if the new one fails-> write the code->run tests-> refactor the code->repeat

A **delivery pipeline** is a sequence of automated stages that retrieve input and run jobs, such as builds, tests, and deployments.
It automates the continuous deployment of a project, with miniumum or no-human intervention.

- A Stage retrieves input and run jobs, such as builds,tests, and deployments.
- Jobs run in discrete working directories in containers that are created for each pipeline run
- By default, stages run sequentially after changes are delivered to source control

**showcase the applicaton**: At the end of the iteration before the retrospective meeting that is a showcase meaning, also know as demo meeting, where the work done is demonstrated to the stakeholders and feedback is obtaining. The showcase meeting starts by reviewing what we have committed in the form of user stories. And finally we demonstrate what we have accomplished in the form of working software, so we can acquire feedback on the product we are building.

**retrospective meeting**: how to conduct
1st way:
What went well?
what could be improved?

2nd way:
Start doing
- continuous delivery
stop doing
- time-bxed iterations
continue doing
- ranked backlog

## Common Problems
Value/Focus
---------------------------
|Problems                |Solution           |
|-------------------------------|-----------------------------|
|Not understanding the market   |Hypothesis-driven development|
|Putting wrong products into the market |Minimum Viable Product (MVP)|
|Lack of focus on value         |Timboxing/Iterations|

Productivity/Speed/Cost
---------------------------
|Problems                |Solution           |
|-------------------------------|-----------------------------|
|Too long to get projects up and running |Born on the Cloud   |
|Slow to put products into market| MVP/Delivery pipeline/Continuous delivery |
|Too costly projects         |Automation/Toolchain/Delivery pipeline|

Communication/Visibility
---------------------------
|Problems                |Solution           |
|-------------------------------|-----------------------------|
|Priorities aren't clear  |Ranked backlog     |
|Requirements not reflecting the user's needs|User stories    |
|Poor team communication|Daily standup|
|Lack of visibility into the progress|Iteration wall/Showcase meeting|

Quality/Control
---------------------------
|Problems                |Solution           |
|-------------------------------|-----------------------------|
|Low quality products  |Test driven development/Delivery pipeline|
|Lack of prcess improvement|Retrospective meeting/Continous improvement|
|Bad Requirements estimations|Story points/Planning poker|
|Not honouring past our commitments|Track team's velocity|

## Scrum Sprints
- Sprint goal is the high-level goal of each time-boxed sprint written as concisely as possible
- The development team then plans how to convert the goal times into a product increment
- The sprint backlog is a subset of the product backlog and contains items selected for the sprint, plus additional iteams for
the development team

## The scrum pillars (TIA)
- Transparency
- Inspcection
- Adaptation
## Scrum roles: Product Owner
- Anyone on Scrum team may change the product backlog, but must do it with product owner's knowledge
- Product owner is accountable for the product backlog
## Scrum roles: The development team
- The development team is self-organized and operates with minimal input from external sources
- The development team owns the entire sprint backlog(selected product backlog iteams, plus development tasks)
- The are no managers or team leads-it is a flat hierarchy
