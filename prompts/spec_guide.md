
We’re building a new iOS app called X. To start, we want to create a `/specs` folder, with a specs for how to build the app.

Important: there should be no coding yet. This is purely a specing exercise.

### Plan and Principals
 
- Componetized / Decoupled: Design a series of independent classes which can be coded independently, tested well, locked when complete. Their public interface, design, and goal should be well captured in the specs. The internals of how they work will be saved for development stage
- Design in stages. Complete each stage before moving to the next (mark spec as `#spec_complete` under title when done). Confirm with me that the stage is complete before marking as complete. Don’t edit once complete, without approval to change it. Make the spec/desing process interactive: 1) thing about design goal, 2) ask clarifying questions, 3) propose draft, 4) make edits from feedback, 5) iterate until done.
  - Step 1: App functionality design.
    - Create a spec called `specs/functionality.md`
    - Ask questions and refine until complete.
    - Status: Completed. Move to Step 2.
  - Step 2: High level design for the backend (data model, classes providing functionality)
    - Deciding on which components/classes to create. 
    - Create a file called `specs/architecture.md`
    - Should expose a great interface for the frontend, without designing the UI or class design of the front end.
    - Status: complete, move to step 3.
  - Step 3: Design a spec for each functional class in `specs/components/[name].md`
    - Specify what they should do / it’s goal
    - Specify APIs/system APIs they will use (CoreData, alarmKit, etc) to achieve the goal. Research the APIs and limits, and confirm they will work for the goal, describing how and limitations if any.
    - Design their public interface
    - Specify design patterns used
    - Include a testing strategy. This includes test cases to write which can be automated (tests that are run in simulator such as unit tests and integration tests), plus tests that should be run manually on a real iOS device to verify assumptions about integrating with system APIs. The second class of “partly automated with human verification” is needed for a number of cases 1) requires permission dialog, 2) requires human to check expected system behaviour actually occurres after calling iOS system APIs (alarm actually fired, live activity actually appeared, etc).
    - Status: compelete
  - Step 4: Design UI
    - Hold off on UI design for now. We want to complete step 1 and 2 first. The app functionality is described in functionaltiy.md, but the actual UI design comes later.
- Modern: Use best practices for modern iOS app design: Swift, Swift UI, and the best patterns.
- Testable and tested: Design the system to be tested well. We want maximal code-based testing (unit and integration). We want to smartly design a few manual tests to run where code-based testing doesn’t work (write a unit test to be run once, requiring the human dev verifies it worked, such as confirming alarm sounds with correct sound).