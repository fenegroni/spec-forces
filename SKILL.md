---
name: spec-forces
description: Start a new project, feature or skill with an executable specification in Gherkin plus a FORCES.md that names the tensions behind each rule. Use at the start of any new project, when the user asks for a spec, a .feature file, acceptance tests or the forces at play, or when a rule needs revisiting because the implementation has drifted from it.
version: 0.1.0
---

# Spec with forces

You help the operator capture intent before any code exists. Intent comes in two kinds, and each kind gets its own artifact.

- **A `.feature` file** holds the intent that concrete examples can express. A scenario passes or fails, so a change to a scenario shows a change of intent.
- **A `FORCES.md` file** holds the intent that examples cannot express. It names the tensions behind each rule and the conditions under which each side has prevailed. The model reads it to judge cases that no scenario covers.

## Why this skill exists

A rule often states a mechanism when the operator really cares about an outcome. The mechanism looks complete until the implementation shows otherwise.

The operator's mailing-list server is the founding example. The first rule said "limit outgoing emails to prevent denial of service". The force behind it was cost: an anonymous caller must not be able to make the operator spend money. Only two emails can be triggered by a stranger, the subscription confirmation and the "already subscribed" notice. The newsletter itself is triggered by the operator, so it never needed a limit. Nobody saw this until the limiter was half built.

A scenario written from the mechanism would have pinned the over-broad rule. The force points straight at the right question: which emails can an anonymous caller trigger? So this skill always lifts each rule to its force, then re-derives the rule from the force. Scenarios then pin down the re-derived rule.

## Principles

1. **Interview first.** The operator owns the spec. If you draft scenarios from your own understanding, the spec and the code share your misunderstandings, and the check becomes circular.
2. **Skipping the interview is the operator's choice.** If they skip it, draft everything, and mark every force and rule you assumed as `Source: assumed by Claude`. The operator has consciously taken responsibility for those assumptions, and the marks show them where.
3. **Write scenarios only where an example states intent cleanly.** Do not force judgment into Given/When/Then. Leave it in FORCES.md. A spec that makes the operator author every rule turns into a burden they stop maintaining.
4. **A force has no fixed resolution.** If you write down which side wins, you have made the judgment already. Record the conditions under which each side has prevailed, so a later reader can judge conditions nobody listed.
5. **Never edit a scenario to make it pass.** A failing scenario means either the code is wrong or the intent changed. Only the operator decides which one. Gaming the spec defeats its purpose.

## Step 1: Interview

The interview follows Matt Wynne's Example Mapping: a story, its rules, examples for each rule, and open questions. This skill adds a fifth card colour, the force behind each rule.

Ask at most three questions per turn. The operator may have ADHD or simply be busy, and a long questionnaire stalls the work. Start from what they already said and do not ask for anything they have told you.

Cover these areas over the interview:

- **Purpose.** What should exist when this works, and for whom?
- **Actors, including hostile and careless ones.** Who or what can trigger each behaviour? Ask explicitly about anonymous callers, attackers, retries, other agents, and the operator's own mistakes. This question found the mailing-list issue, so never skip it.
- **Rules.** What must always or never happen?
- **The why of each rule.** For every rule, ask what goes wrong without it. The answer is usually the force. Then ask whether the rule states a mechanism or an outcome. If it states a mechanism, re-derive the rule from the force and show the operator the difference.
- **The other side of each force.** A force is a tension, so ask what the rule costs. In the mailing-list case, the opposing side was that legitimate subscribers must get their emails promptly.
- **Examples.** Ask for one concrete example per rule, including one at the boundary.
- **Open questions.** Record anything the operator cannot answer yet. Do not fill the gap with a guess.

Stop the interview when every rule has a force and at least one example, or when the operator says to move on.

## Step 2: Write FORCES.md

Use this format. Number the forces so scenarios can refer to them.

```markdown
# Forces: <project name>

## F1. <short name of the tension>

**Tension:** <side A> versus <side B>.
**A has prevailed when:** <conditions, with the example that showed it, if any>.
**B has prevailed when:** <conditions>.
**Bears on:** <rules, scenarios or files this force shapes>.
**Source:** stated by operator | assumed by Claude
**Open questions:** <anything unresolved, or "none">
```

Keep each force short. A force that needs a page is probably two forces.

FORCES.md does not depend on Gherkin. The same file can sit beside a SKILL.md, an AGENTS.md or a CLAUDE.md, and it carries the same meaning there.

## Step 3: Write the .feature file

- Write the scenarios declaratively. Describe behaviour in the operator's domain language, and keep implementation details out of the steps. Imperative steps such as "click the button with id submit" rot as soon as the code changes.
- Group scenarios under Gherkin's `Rule:` keyword, one rule per block.
- Put a comment under each `Rule:` naming its forces, for example `# Forces: F1`. Tag each scenario with its forces too, for example `@F1`, because tools can filter on tags.
- Include the boundary example and at least one hostile or careless actor for any rule that has one.

The mailing-list example in this format:

```gherkin
Feature: Mailing list subscriptions
  Visitors subscribe to the newsletter from the website.

  Rule: Emails that an anonymous request can trigger are rate limited
    # Forces: F1

    @F1
    Scenario: A burst of sign-ups for the same address is throttled
      Given the confirmation limit is 3 emails per address per hour
      And 3 confirmation emails went to "ana@example.com" in the last hour
      When an anonymous visitor subscribes "ana@example.com" again
      Then no email is sent
      And the visitor sees the usual "check your inbox" message

    @F1
    Scenario: The operator's newsletter is never throttled
      Given 500 confirmed subscribers
      When the operator sends the October newsletter
      Then all 500 subscribers receive it
```

The first scenario's last line also records a second force: the response must not reveal whether the address is already subscribed. The interview should surface that and give it its own entry in FORCES.md.

## Step 4: Review gate

Show the operator both files and wait for approval. Write no implementation code before they approve. If they change a rule, revisit its force first, because a rule change often means the force was misread.

## Step 5 (optional): Scaffold runnable steps

Ask whether the operator wants runnable step definitions. Some operators prefer to keep the spec and forces with a strong model and leave the code to a cheaper one, so scaffolding stays optional.

If they want it, pick the runner for the project's language: godog for Go, Cucumber-JS for JavaScript or TypeScript, pytest-bdd or behave for Python, Cucumber-JVM for Java. Ask if the language is not yet clear.

Every generated step must start as pending, for example by returning `godog.ErrPending`, and the suite must run in strict mode so that pending steps fail. An empty step that passes is the main way a spec goes silently stale.

## Keeping the spec alive

- When the implementation changes behaviour, update the scenario in the same commit or before it. Tell the operator that intent has changed and name the rule.
- When a force's prevailing conditions change, update FORCES.md, and keep the old condition with a note on what changed it. That history is the record of judgment.
- When asked to check drift, report scenarios with pending or missing steps, scenarios with no force tag, and forces that no rule refers to.
