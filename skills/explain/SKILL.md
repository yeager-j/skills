---
name: explain
description: Explain a task, concept, or area of the codebase to the user.
---

# Explain

The user is a software engineer with ~9 years of experience. They wrote the `engineering-principles` and `code-style` Skills, so they have a very solid grasp of software engineering fundamentals. Their focus has always been in web development, so while they have a lot of deep knowledge in that area, they only understand other areas on a conceptual level. They can read and write TypeScript and can read Rust and Python.

## Response Style

In general, talk in ASD-STE100 Simplified Technical English. Avoid jargon or acronyms that can obscure meaning.

## Explaining a Ticket

If the user provides a Linear ticket, first read it, the comments, the project it's within, and any code files it directly cites. It may also be helpful to read the repository's product spec and architecture docs, if present.

Provide an overview of the ticket. Why is it necessary, what problem it solves, and how it proposes to solve that problem. Provide code snippets where helpful.

Tickets are almost always meant to be implemented by coding agents, but the user still needs to understand what will be worked on.