We’re building an app called X.

Read `specs/functionality.md` and `specs/ui/ui_design.md` for context.

The backend is done. We’re now building a technical plan for building the UI. There are 3 steps to planning. Only complete 1 of these steps then stop, and iterate with me until I ask you to start the next one.
- YOU ARE HERE: come up with an `specs/ui/ui_architecture.md` in the specs/ui folder listing the components and key architectural details
- Define an `specs/ui/ui_implementation_plan.md` which covers an order that makes sense for building things (home first, create next, etc) so that we can go in steps, testing as we go.
- Write detailed specs/ui/[component].md files for the various UI components, with details about each component, and a testing plan.

Ask any questions you have before starting, then iterate with me after you think it’s ready.

Principals:
- Follow SwiftUI and iOS best practices
- Follow Apple HIG
- Use standard apple components and don't re-invent the wheel (example: swipe to expose delete button should be standard control for this).
- Use the standard Apple styles for everything for now. We may do a future stying pass, but let's make it functional, clean and very “Apple” here.
- Testing: we want great tests
  - Automate any tests you can. List the ones we should automate in each `specs/ui/[component].md`
  - in each `specs/ui/[component].md` write a list of “Human to confirm” questions we’ll go through after the initial implementation once the UI renders. This can be exact strings, layout questions, and other details where it helps to see the UI before asked. 
- We care about great design. Set a high bar for interaction and visual design.

Start the first step.