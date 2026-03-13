# Lessons Learned

## It Still Needs You

The process works, but AI still gets things wrong — sometimes confidently, sometimes subtly. Here's where I had to step in.

**Technical design.** Three examples from the iOS app:

The agent wanted to JSON-serialize objects and hash the result to generate stable IDs. JSON serialization isn't deterministic — key ordering can vary between runs, across platforms, or after library updates. I caught this during spec review and pushed for a different approach. The agent agreed immediately, which is both nice and a little unnerving (more on that below).

It assumed iOS background execution was reliable. Built the scheduler around the idea that the app could do meaningful work in the background on a consistent schedule. That's not how iOS works. Background execution is best-effort — the system can throttle or kill it at any time. I had to flag this and redesign the approach.

For a pipeline project using Google Cloud Storage, I needed atomic batch file moves. The agent didn't know about GCS Hierarchical Namespace (HNS), a relatively new feature that supports this. I pointed it to the docs and it picked it up immediately. But without me knowing HNS existed, it would have built a fragile workaround.

**UX design.** The AI is great at implementing UI details — a nicely formatted table cell, a smooth animation, proper loading states. But overall navigation structure needed me. When to use a modal vs. a navigation push, how screens flow together, what the information hierarchy should be — those are judgment calls that require understanding what the user is actually trying to do. Best approach: let the agent take a first pass, then fix what feels wrong rather than over-specifying upfront.

**Product design.** The agent doesn't know why you're building the thing. It can build what you ask for, but deciding *what* to build — the "what" and "why" — is still entirely on you.

## Sycophancy is Real

AI agents will do what you tell them. To a fault. Even if it's a bad idea.

In my spec, I had a one-line requirement about notification recurrence. The agent dutifully designed an entire system around it — one that would have been fragile in both code and reliability. It never once said "hey, this is going to be a pain and might not work well." It just built what I asked for.

The model wants to be helpful, which means agreeing with you and executing your plan. It won't push back unless you explicitly ask.

The mitigation is simple but easy to forget: tell the agent to challenge your assumptions. "Push back on my ideas. Tell me what's wrong with my approach. Suggest alternatives." When you do this, it actually gives good feedback — it just won't volunteer it.

The catch: you have to remember to do this every time. The one time you forget is probably the time you needed it most.

## AI-Driven Manual Testing

Not everything can be unit tested. UI flows, system integrations (AlarmKit, notifications, live activities), hardware-dependent features — these need human eyes and hands. But the AI can still do most of the work.

The concept: when a phase includes UI or system integration work, the agent writes a manual test plan as part of its build loop. It creates a diagnostic view inside the app — a hidden screen with buttons to trigger specific behaviors, display internal state, and exercise edge cases. Then it writes step-by-step test instructions: "tap this button, verify this label shows the next alarm time, check that the notification fires after 30 seconds."

This found real bugs. Alarm schedules were being set incorrectly — off by a time zone conversion. Intents weren't showing up in the system UI because of a missing Info.plist entry. These are exactly the kinds of things unit tests can't catch.

The best part: I ended up doing more testing than I would have on my own. Writing test plans is tedious, so normally I'd test the happy path and move on. Here, the agent wrote thorough plans covering edge cases I would have skipped, and I just followed the instructions. Repeatable test plans as a free side effect of the build process.

(Prompt: [manual_test_plan_guide.md](../prompts/manual_test_plan_guide.md).)

## More Polish Than You'd Bother With

Small improvements become free. Not free-as-in-no-cost — the tokens cost something — but free in terms of your time and attention.

"Make it so the return key on the keyboard advances to the next input field." That's a 15-second prompt. In a normal project, I'd add it to a backlog and probably never get to it. Here, it's done before I finish my coffee.

On another project (a data pipeline in Python), the Google Cloud SDK didn't support async operations... in 2026... come on, folks. The agent rewired the integration to use async REST API calls instead of the threaded SDK approach. That's not a trivial change — it touched the HTTP layer, error handling, retry logic — but from my perspective it was one prompt and a code review. The codebase is faster and better for it.

You end up with a more polished product because the marginal cost of small improvements drops to near zero.

## It's Not Great at Speccing

The AI can draft specs, but it's not great at designing systems from scratch. A few patterns I noticed:

**Multiple competing sources of truth.** As the spec grew across files (functional spec, architecture, component designs, phase plans), details would drift. A decision in the architecture doc wouldn't fully propagate to component specs. The agent would sometimes reference an outdated assumption from an earlier conversation. Keeping everything consistent was ongoing work.

**Confident assumptions about things it hasn't verified.** The iOS background runtime example is a good one — the agent assumed it worked reliably because the API exists. It didn't think to check the real-world constraints. Similarly, it designed a non-defensive scheduler: no fallback for missed executions, no recovery path. These are design decisions that come from experience shipping software, not from reading documentation.

**Design weak spots.** When the agent had to make genuine design tradeoffs — not "which pattern fits here" but "what's the right behavior when two requirements conflict" — it would pick the most obvious path without flagging the tradeoff. You have to actively probe for these.

This is why the process front-loads human attention in the planning phase. The specs are a collaboration, not a delegation.

## Example Projects and Costs

Two real projects, to give a sense of scale.

**iOS App** — the main project described in this post. Offline-first iOS app in SwiftUI with notifications, alarms, live activities, and a full test suite.

- ~185M tokens during the coding phase (autonomous agent)
- On the z.ai Coder plan: **$30/month** for essentially unlimited usage. I only hit the temporary rate limit once across the entire project.
- The same token volume on Sonnet would cost roughly **$2,400**
- The same on GLM 4.7 API pricing: **~$320**

**Pipeline Project** — a data pipeline / backend project in Python with Google Cloud integrations. Similar pattern: bulk of the tokens in the coding phase, minimal spend during planning.

The z.ai Coder plan is what made this practical for me. $30/month for what would otherwise be hundreds or thousands in API costs. Your mileage will vary depending on model choice and provider, but the key insight is: separate your planning model (best available, low token volume) from your coding model (good enough, high token volume).

One last note: all those tokens running on GPUs use electricity. Carbon credits exist and are cheap. Worth considering if you're burning through millions of tokens regularly.

## The Hard Part: Tools and Sandboxing

The hardest part of this process wasn't the AI or the prompts — it was the tooling setup so the agent could actually work autonomously.

Autonomous agents need to build, test, lint, and format code. But you can't just give them unrestricted system access. Sandboxing is essential — you don't want an AI agent with free reign over your filesystem, network, and credentials. The tension: the agent needs enough access to do its job, but not so much that a hallucinated `rm -rf` ruins your day.

I spent more time configuring MCP servers for autonomous tooling for the iOS project than I spent on any individual coding phase. Build commands, test runners, linters, formatters — each one needed to be exposed to the agent in a controlled way. This is a one-time cost per project, but it's real.

**Some toolchains just work.** Go and Python (with uv) were painless. The CLIs work in sandboxed environments, the build and test tools are straightforward, everything's fine.

**Xcode is pain.** iOS development was a different story.

- Xcode's CLI tools (`xcodebuild`, etc.) don't work properly in a sandboxed environment. I had to write a custom MCP server just to bypass the sandbox for builds and compiler warnings.
- Xcode's CLI is second-class to its GUI. Occasionally the agent would get stuck in a loop, and I'd have to open Xcode manually to clear the state.
- Some iOS APIs — AlarmKit, certain notification types — require a physical device. Simulator testing wasn't sufficient. This is where the manual test plans became essential.

The tooling story will get better as the ecosystem matures. Right now, budget extra setup time if your toolchain isn't natively CLI-friendly.
