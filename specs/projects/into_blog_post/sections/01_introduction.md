# Vibe Crafting: High Quality App Development without Coding

I recently built an iOS app without writing a single line of code. I'm an ex-Apple software engineer — I could have written it myself, but I wanted to push the boundary of AI development.

The goal: produce code that's as good or better than I would write, without writing any of it. Zero compromises on architecture or quality. Take as much time as needed to get it right. But offload all the coding.

**TLDR** I ended up developing a process — upfront technical specs, phased autonomous builds, and ample code review (both manual and agentic). It's like vibe coding in that I don't read every line or write any lines, but the AI doesn't get to wing it either. I'm calling it "vibe crafting" for lack of a better term. The result was a codebase better than I would have written — not because I can't, but because I'm lazy. I would never have written 288 tests and a full UI test suite on my own.

This repo contains the actual process I used, along with the real prompts, specs, and planning docs from the project. When I reference a prompt, you can click through and see exactly what I fed the agent.

## Table of Contents

TODO: update this at compliiation time. This format is correct. The content should reflect final article
> **[The Two Tools:](#the-two-tools)** Interactive Agent | Autonomous Agent
> **[The Process:](#the-process)** [Overview](#overview) | [Project Overview](#human-writes-project_overviewmd) | [Spec & Plan](#build-spec--implementation-plan) | [Build Phases](#build-phase-by-phase) | [Review](#review-each-phase) | [Testing & Iteration](#human-testing--iteration) | [Code Review](#end-to-end-agentic-code-review) | [External Review](#external-code-review)
> **[Lessons Learned:](#lessons-learned)** [It Still Needs You](#it-still-needs-you) | [Sycophancy](#sycophancy-is-real) | [AI-Driven Manual Testing](#ai-driven-manual-testing) | [Free Polish](#more-polish-than-youd-bother-with) | [Speccing](#its-not-great-at-speccing) | [Tools & Xcode](#the-hard-parts-tools-and-xcode) | [Tokens & Cost](#tokens-cost-and-the-environment)
