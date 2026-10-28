# spec-forces

A Claude skill that starts every new project with two artifacts:

- a Gherkin `.feature` file for the intent that examples can express, and
- a `FORCES.md` file for the tensions behind each rule and the conditions under which each side has prevailed.

## Why

Executable specifications put the operator in charge of every rule, and rules often describe a mechanism instead of the outcome that matters. Forces carry the outcome. The skill lifts each rule to its force in an interview, then re-derives the rule from the force before any scenario is written.

The founding example is a mailing-list server. Its first rule limited all outgoing email to prevent denial of service. The real force was cost: an anonymous caller must not be able to make the operator spend money. Only two emails could be triggered anonymously, and only those two needed a limit. The team found this halfway through building the limiter.

## Context

Dave Farley's video [The Industry Is Catching Up: The Only Way to Generate AI Code That Works](https://www.youtube.com/watch?v=EkH_uoybj8k) on Modern Software Engineering argues that markdown specs drift and that the specification must be executable. This skill puts that idea into a form Claude can follow directly. It adds one thing: a `FORCES.md` beside the `.feature` file. An agent can make a wrong scenario pass, and the forces give the operator and the model a way to notice that a rule no longer serves its purpose.

## Install

Copy this folder into `~/.claude/skills/spec-forces/`, or upload it as a skill in Claude. Then start a project with "Set it up with spec-forces".

## How it works

1. An interview based on Example Mapping, with forces as an extra card colour. It always asks who could trigger each behaviour, including hostile and careless actors.
2. `FORCES.md`, one numbered entry per force.
3. A `.feature` file, with scenarios grouped under `Rule:` and tagged with their forces.
4. A review gate. No code is written before the operator approves.
5. Optional runnable step definitions, generated as pending and run in strict mode.

The operator can skip the interview. The skill then drafts everything and marks each assumption as its own.

## Evals

The `evals/` folder uses the skill-creator case layout. The `anonymous-trigger-cost` case tests the founding claim in a new domain, an SMS booking page. A literal mailing-list replay would be contaminated, because SKILL.md contains the mailing-list example and so already holds the answer. The case asks whether the skill surfaces the anonymous-trigger question before any code exists.

## Status

Version 0.1.0. The claim that forces would have surfaced the mailing-list issue earlier is an intuition still being tested. The anonymous-trigger eval is the first test of it.

## License

MIT. See [LICENSE](LICENSE).
