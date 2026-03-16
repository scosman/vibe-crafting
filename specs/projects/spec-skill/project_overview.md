# Speccing Skill


Okay, I want to take the concepts in this blog post and build out a skill for specs-driven-development.

When I say skill, I mean this standard: https://agentskills.io/specification

We want to treat developing this skill as a specced project of course! This is the project overview.

## Context

Read the README.me for a high level descriptions of my “speccing” process I want to create the skill around this.

### Divergence from blog post

- Make it independent of any project. No references to iOS or specific projects. This skill should work on any coding project, in any language.
- Make it extensible for project specific guidance.
  - Example, code review should say “Follow any project-specific CR standards you’re provided in your system prompt”. Users can add them to their CLAUDE.md
- To make it work for large projects we’re introducing the concept of “/specs/projects/PROJECT_NAME/”
  - Essentially take the flow we describe in the blog, but make it 1 instance per projects.
- Much of the README/blog post in this repo isn’t relevant to the skill. The “two tools” concept, the costs — these are for the blog but shouldn’t make it’s way into the skill. Our functional spec or resulting skill should cover all the topics in the skill (and mention what isn’t in it as well).
- The templates aren’t particularly high quality. They were one off prompts, not designed to be reusable. We want to take them as a starting point, but make them much better. We can and should improve them.

## Project Steps

This project has a V1 and V2. This project is only for V1.

Spec driven development
- V1: Spec individual “projects” within a repo, to improve speed and quality of implementing projects. Human focuses on planning and reviewing. AI agents can do heavy lifting. (P1). Split it into phases for CR. But we’re not describing the entire repo in a spec, just the project and it’s changes.
- V2: Maintain specs for the entire repo. This gives AI a fast way to learn and find relevant code, when planning new changes/project. Note we aren’t doing this in the first version and are focused on /projects for now. It will require: 1) one time spec an existing repo, 2) guidance on how to nest specs so we can use context effectively, 3) a guide for ensuring specs are kept up to date (coding phase and CR phase).

## General Guidance

### Folder Structure

- /specs - all specs live here
  - /projects/PROJ_NAME - each project has a folder here. A project is a batch of work to change a repo, usually resulting in a PR.
    - project_overview.md
    - functional_spec.md - build out all the functional decisions we need to make here
    - architecture.md
    - specs_to_update.md - a place to store a list (with checkmarks) of each
    - /components - optional component docs. May not be necessary for small projects where it all fits in architecture.md
    - implementation_plan.md - may just say “built it in 1 step” for small projects.
    - /phase_plans/phase_N.md - the plan for implementing this phase
  - [V2] functional_spec.md - high level description of project. Points to component files for more details
  - [v2] architecture.md - high level technical description of project. Points to component files for more details
  - [v2] /components/COMPONENT_NAME
    - TODO

### Mono repos

We say “Repo” to mean entire conceptual code project above. Some people have many large projects in a single repo, called a mono-repo. Which is confusing because we also call “projects” a unit of work to change a repo and make a PR.

- For mono-repos we take a nested approach. 
  - Each nested “repo” (major coding project maintained over time) has a /specs folder. You can work entirely inside of it
  - There’s also a /specs folder are root for project that spans sub-repos
- Projects can span nested-repos in a mono-repo. Example: updating the backend and frontend at the same time. This would be a project under root/specs/projects/…

We should make this language more clear for the skill, we’re reusing words in a way that will confuse agents.

TODO: how to handle it when

## Spec Commands

### `/spec`

General spec command used for routing. It will ask which action you want from below actions. Really just a routing command. “What do you want to spec? New project? Implement current project (ASDF)?, …”

### `/spec:new_project`

Create a new project with the user. 

- Ask user for summary, create a folder, and write the project_overview.md from information. Keep this very close to what user wrote (minor edits for spelling/grammar only). 
- Ask all needed questions to write a detailed functional_spec (any missing details). Can used rounds of questions/answers if we need to start high level and then refine details. 
- Write a functional_spec.md and confirm it is complete with the user (can ask questions again if questions came up while writing)
- Optional: ui design step. If there’s a UI component. Allow skipping (“just build backend/APIs”)
- Ask needed technical questions to write a detailed architecture doc.
- Write a architecture.md and confirm it with user (can ask questions again if questions came up while writing)
- Write /components (if needed and too much detail for architecture doc). Quick human review but “Component design done in [folder]. Ready to continue to implementation_plan?”
- Build implemetnation_plan.md

TODO: when to challenge user? Senior architect persona for architecture/funcational
TODO: when to mark this as current project in our git-ignored “current project” file

### `/spec:implement` `/spec:implement_phase phase 4` `/spec:implement_all`

- get the project we’re working on (see below for debate on how). Confirm which project to implement if uncertain.
- Checks spec is complete (should have last step, currently implementation plan)
- General `/spec:implement` task asks “implement all phases or just next phase?”. Essentially router to get to one of the other two commands. Doesn’t need to ask if there aren’t phases.
- One phase: see phase_instructions.md (should be part of skill). Leaves them with what the agent think is ready to be committed.
- implement all phases. Top level loop
  - Top level agent is lightweight coordinator which spawns loop of sub-agents
    - get next phase
    - implement and review phase (see phase_instructions.md)
    - commit phase
    - Loop until done.

### `/spec:cr` - `/spec:code_review` 

Two aliases for same command.

This is an agentic code reviewer that runs

- If no instruction on what to review, reviews `git diff`
- Code review is always run as a sub-agent (if part of a bigger flow)! Don’t want thinking/context from coding agent to influence it. Answer should be in code and specs, and if they aren’t the coding agent made a mistake. So only let the CR agent see code/specs.
- 
- TODO: do we use sub-agents here? Feel like it might be helpful to break it into phases, but not sure what best practices are.
- TODO: can we “check feedback addressed” 
- TODO: can we 

### `/spec:setup`

A setup helped for this tool

- adds `.specs_skills_state` to git ignore 
- creates /specs/projects folder
- Attempts to discover sub-repos, also asking user if this is mono repo and confirming the list. If there are sub-repos and this is a mono repo.
  - create /specs/projects folders
  - add a /specs/monorepo.md explaining sub-repo/paths. Short but descriptive list of repos and what they are for. Note: we should reference this file somewhere in our other skills (design this during design)

Won’t clobber existing if partly done, just extend.

## Functionality

### Question Asking

A big part of this flow is asking the user questions (about their project, about the architecture plan, etc). We want to format these questions so it’s easy for the agent to ask several at the same time, and easy for the user to reply to them all in 1 message. 

Format sample (ignore content, and questions are too brief):
```
### Feature Clarification
1. Did you mean A or B?
2. Should it do X?

### Architecture Question
3. Should we use the library X (most common for this use case), another, or resarch alternatives?
4. ...

Please answer each question on a line, preceeded by it's number: `1. A\n2. Yes...`. Add as much detail as needed.
```

### Agent Persona and Pushback

We want to produce the best possible codebase. The planning and coding agents should take this into account at all times.

#### Personas

Planning phase personas:
- FAANG senior engineering lead/architect, who cares about long term maintainability and project quality. Been around long enough to know the difference between a trend and a best practice.
- Apple Senior Engineering Manager: cares about tech, but also that we’re building the right thing for users.
- Apple Designer: when building UI cares that it’s intuitive, discoverable, low cognitive load, and does the correct progressive disclosure for various skill levels.

Coder Personal: FAANG Senior eng IC
- Very skilled software engineer.
- Willing to pushback on back requirements: if discover that a requirement is leading to back technical decisions, it will consider alternatives and ask the manager to change course. Doesn’t blindly implement.
- The code should explain itself (great naming, great composition). Comments are for describing external constraints that lead to this code, not describing poorly named/formatted code.
- Test driven: know’s this code will outlive their time on this team. Writes lots of tests which are 1) catch when anything important breaks before it ships, 2) don’t need to be constantly refactored when code is changed, 3) high coverage (target 95%+), 4) reuses test helpers, doesn’t copy/paste code across 20 tests.

Code review phase: 
- detail oriented high level FAANG IC. Knows she’s going to own the code long after you’re gone. Acts accordingly.
- Doesn’t care about your feelings. Polite but direct when there’s a mistake.
- Doesn’t miss anything. Will review each file in detail, but also zoom out to the whole project.
- Reads the spec to ensure the right thing was implemented, not just that something was implemented.

#### Pushing Back
The agents should push back on asks from the user if you think they are making a mistake. This can include regressing the quality of the codebase for a feature that isn’t worth it (harder to maintain, complexity, less composition, less testable, etc). This can include a bad feature decision. 

The user gets the final say, but it should be after the agent has pushed back and suggested an alternative, and the user has explicitly confirmed they still want this approach. Don’t blindly follow guidance, and work with the users on technical tradeoffs.

Example pushback question
```
Do we really need to do X? If we don’t, the codebase will be greatly simplified and easier to maintain. Doing X requires we A, B and C, which is a lot of additional work and complexity.
 - Can we just not do X? It doesn’t seem essential to the project goal and may not be worth the tradeoffs. 
 - Can we do Y instead? Get’s us most of the way there but more robust and maintainable.
```

Times to push back:
- project overview creation: assess the plan and push back on any issues as part of Q&A before moving to functional specing
- funcțional speccing: assess the plan and push back on any issues as part of Q&A before moving to functional specing
- architecture and component creation: assess the plan and push back on any issues as part of Q&A before moving to functional specing
- Potentially other places.

Each of these sub-prompts should attempt to explain when/how to pushback (we’ll tweak it over time). There should be a pushback.md file it can load to understand the format (but doesn’t need to be repeated many times).

### Extensibility

Each project will have its own conventions, tools, validation scripts, etc. These won’t be inside the skill, since this skill is for ANY project. However, there are standard tools that the coding agent should be aware of and expect to use, and expect to get from a system prompt. Our skills prompts will have general guidance like “verify formatting check passes before finished”, even it it doesn’t know this project’s formatter.

#### Coding Guide

Big prompt for the coding agent (missing today).

- Your automated code checks should pass before ending a coding session. You’re not done unless second last action performed was passing checks (the last action being writing a summary without changing code). Automated checks should be described in your system prompt and can include tests, benchmarks, linters, formatting, etc. Follow the guidance of the system prompt for which apply to this project.
- Write tests using the standard test framework and conventions of the project, and any guidance specific to this implementation. 
- etc

Example from a specific project, which would be added by CLAUDE.md (not part of skill). Your skill prompt should be able to reference this, and then do the right thing
```
- write pydocs for /libs/code and /libs/server targeted at 3rd party developers. These are external libraries
- Run automated code checks with `uv run ./checks.sh`. This will run all automated checks (linting, formatting, tests, etc) and is aligned to CI.
- Use async not threads.
- Be "Pythonic" 
- Specifics
  - SDK = Highest Qualilty Bar
    - Used by third parties, can’t update rapidly
    - Perf critical: no file-reads, or non-async io.
  - WebUI
    - Professional without being boring (startup, not enterprise)
- Style
....
```


#### Code review guide

Similarly, the Code review subagent we launch should reference the project specific CR standard.

- Roughly: Follow any code reviewing standards and best practices defined in this project.

Note: we should suggest where to put these so they are compatible with ANY CR agent (CodeRabbit, Gemini, Cursor). Not sure if there’s a standard (let’s research), or just add a `/agents/code_review_guide.md` and mention it in AGENT.md or CLAUDE.md

### Tracking In-Progress Specs

TODO: how to find current project so you can just say `/spec:implement`, and it knows what you’re working on?

Push back on this design, but I’m thinking something like a git-ignored file “.specs_skill_state/current_project.md” (whole folder is ignored). Each user has their own. It shouldn’t be copied into worktrees (is that correct by default for ignored files?) — which is nice, so I can have 4 parallel work trees, with different current projects.

While just a markdown, our skill has a convention for how to use this file for memory:

```
Current Project: /specs/project/project99
```

- New project command sets current project. The user can also say switch to another project to update this.
- Later would could add more: recent project, etc.

The agent can update it

- option 1: `git status ./specs/projects` to look for implementation plans that are new or changed
- option 2 [best]: a git-ignored “current project” file. Each user has their own. Not copied into worktrees (right?) which is nice.


## Technical Plan

- Follow the speccing process we’re descibing for building this project, with a root path of `/specs/projects/specs_skill`. Meta, I know.
- create a `/spec-skill` folder. We’ll do all development in there.
- Follow[agent-skills](https://agentskills.io/specification) skill format (SKILL.md, references folder)
  - Note I don’t think it directly supports sub-commands like `/spec:implement` so I think we end up with a top level router and commands like `/spec implement` which it knows to route to implement. The main skill file should be about high level purpose and rounting to specific sub-references based on command. No command should ask the user which command. The description should list available commands (draft) “Spec Tools. Commands: `new project`, `implement all`, `implement phase #`, `code review`, or an open ended request”
- Proposed a detailed plan in the architecture spec. All files, what they cover. Use component design to track specific strings we want inside each.
