We’re building a new iOS app called “Get It Done”. We have a well defined `/specs` folder, with a specs for what the app does, it’s architecture and component design.

Read the specs folder and design an implementation plan for building this one piece at a time.

- Create a file `progress/implementation_plan.md` describing the implementation plan. You don’t need to reiterate content of specs. Focus on defining a good order of building things that is:
  - logical based on dependencies 
  - grouped into steps at the scope of 1 component per step is good, but you can group together 2 components if if it genuinely good to build in parallel.
  - Keep this file short, the specs contain details, this is a place to check progress and pick up next task.
  - Use markdown checklist style so we can update this to track progress. 
  - Assume you’ll start with an empty iOS project in the `src` folder. A Xcode blank project for “new iOS app”. You don’t need to implement the project structure, just the app specs.
- Ask any questions before starting the implementation plan