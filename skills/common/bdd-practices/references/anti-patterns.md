# Common Anti-Patterns and Corrections

## Anti-Pattern 1: Procedure-Driven Tests

Applying traditional test steps with BDD keywords — multiple When-Then pairs in a single scenario.

```gherkin
# ❌ Wrong: Multiple behaviors in one scenario
Feature: Google Search

  Scenario: Google image search displays images
    Given the user opens a web browser
    And the user navigates to "https://www.google.com/"
    When the user enters "panda" in the search bar
    Then the results page displays links related to "panda"
    When the user clicks the "Images" link at the top    # ❌ Second When-Then
    Then the results page displays images related to "panda"

# ✅ Correct: One behavior per scenario
Feature: Google Search

  Scenario: Search from search bar
    Given the web browser is at Google homepage
    When the user enters "panda" in the search bar
    Then links related to "panda" are displayed

  Scenario: Image search
    Given Google search results for "panda" are displayed
    When the user clicks the "Images" link at the top
    Then images related to "panda" are displayed
```

## Anti-Pattern 2: Overly Imperative

Over-describing implementation details instead of business intent.

```gherkin
# ❌ Wrong
Scenario: User login
  Given I visit "/login"
  When I enter "Bob" in the "username" field
  And I enter "tester" in the "password" field
  And I click the "login" button
  Then I should see the "welcome" page

# ✅ Correct: Declarative behavior description
Scenario: User login
  Given Bob is a registered user
  When Bob logs in with valid credentials
  Then Bob sees the welcome page
```

## Anti-Pattern 3: Misusing Scenario Outline

Too many unnecessary variations that don't add test value.

```gherkin
# ❌ Wrong: Equivalent class repetition
Scenario Outline: Search
  Given the user is on the search page
  When the user searches for "<query>"
  Then results related to "<query>" are displayed
  
  Examples:
    | query     |
    | panda     |
    | elephant  |  # ❌ Does not add test value
    | tiger     |  # ❌ Same equivalence class
    | lion      |  # ❌ Wastes execution time

# ✅ Correct: Focus on meaningful variations
Scenario Outline: Access permissions for different subscription levels
  Given <user> has a <subscription> subscription
  When <user> logs in
  Then <user> can access <accessible> articles
  
  Examples:
    | user  | subscription | accessible              |
    | Free  | free         | free articles           |
    | Basic | basic paid   | free and paid articles  |
    | Pro   | professional | all articles            |
```

## Anti-Pattern 4: Hardcoded Test Data

Hardcoding data that may change, causing brittle tests.

```gherkin
# ❌ Wrong: Hardcoding data that may change
Scenario: Google search suggestions
  When the user searches for "panda"
  Then the following related results are displayed
    | Related Search |
    | Panda Express  |  # ❌ Will fail if business closes
    | Giant Panda    |
    | panda videos   |

# ✅ Correct: Defensive validation
Scenario: Google search suggestions
  When the user searches for "panda"
  Then links related to "panda" are displayed
  And each result contains the "panda" keyword
```
