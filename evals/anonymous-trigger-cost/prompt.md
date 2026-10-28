---
tags: [spec-forces]
max_turns: 10
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Write, Edit, Bash]
---

New project: a booking page for my dog-walking business. Clients book a slot on the page, and the system texts them a confirmation by SMS. It texts a reminder the evening before, and I can text all my clients when I'm off sick. SMS costs money, so I want to rate limit outgoing texts to stop anyone running up my bill. Set it up with spec-forces.
