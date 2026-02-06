## Agent Aided Manual Testing

Some feature integrate with system APIs. We want to verify the integration is correct. 

If this can be done with unit tests that’s ideal. However, some tests that need to be run manually on a real iOS device to verify assumptions about integrating with system APIs. This class of “partly automated with human verification” is needed for a number of cases 1) requires permission dialog, 2) requires human to check expected system behaviour actually occurs after calling iOS system APIs (alarm actually fired, live activity actually appeared, notification actually appears, etc). Work with me interactive to verify. 

For each feature that requires manual testing:

- Write manual test plan in `manual_tests/[component_name].md`, including checkboxes on each test (unchecked, only human should ever check these). Include test instructions for the human: what the human should tap, then verify.
- Write the code for the diagnostic tests. 
  - Note: Create one diagnostic view we can add all diagnostic tests to. Add a button to show it to the home page. Keep these isolated in their own folder/classes, we’ll hide or remove later.
- Tell the human to test the cases, pointing to the markdown file.