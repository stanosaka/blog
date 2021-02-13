---
title: SRE
date: 2020-02-09 12:10:52
tags: SRE
---
# Implementing SLOs
Service level objectives (SLOs) specify a target level for the reliability of your service.
SLOs are key to making data-driven decisions about reliability, they're at the core of SRE practices.

SLOs are a tool to help determin what engineering work to prioritize.

# Effective team incident reports
It’s essential to have a team incident report per incident. These artifacts are for the team to help to spread knowledge and prevent stagnation where the team as a whole doesn’t know specific knowledge obtained via the incident. Store these artifacts in a central place. Depending on the organization, these artifacts may be useful to other teams.

Each artifact may have slightly different content based on the nature of the incident. The reader of the team incident report is the individuals on the team, so these reports can be longer and more detailed than external or executive briefings.

An example template for a team incident report:

- Title
- Date
- Author(s)
- Summary of the incident
- Incident participants and their role(s)
- Impact
- Timeline
- Include graphs and logs that help support the facts described in the timeline.
- Lessons learned about what went well, and what needs improvement.
- Action Items - These should include who, what, type of action, and when. Others outside of the incident response team might think of additional action items after reviewing the narrative.

While one person should have the responsibility of being the note taker during the meeting and initial author of the report, everyone on the incident response team should review and update the team report.
