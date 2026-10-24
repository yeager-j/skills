---
name: retrospective
description: Run a retrospective session after completing a task.
---

# Retrospective

Take a minute after the ticket has been completed and the PR merged to consider what went well, what went poorly, and any action items resulting from the ticket.

## The Vision

First, ask yourself what the vision for the product you're working on is. During this session, did anything deviate from the vision? Are we layering on extraneous/unnecessary features? Are we serving anything that a consumer doesn't care about?

## The Engineering Principles

Run the `engineering-principles` Skill if you have not yet loaded it. Did the PR violate any of the Engineering Principles? Is there one authority for each piece of knowledge? Was each distinction decided once? If we need to refactor down the road, can we do so confidently?

## The Architecture

If the project has an architecture document, read it if you have not yet done so. Has this ticket caused any cracks in the architecture? Have any assumptions that the architecture made been disproven? Have we discovered new information that may require updating the architecture?

## The Tooling

Did you run into any friction that could be solved with better tooling? Does the tooling provide sufficient feedback or do you have to investigate results further? How is the automated test suite's health? Is it catching real issues?

## Reporting the Retrospective

If the ticket went well and there are no findings, you can say so. Don't attempt to pad the retrospective with nitpicks and speculation. Give each finding a title, a short summary of the problem, and code citations.