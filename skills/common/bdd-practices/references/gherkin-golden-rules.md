# Gherkin Golden Rules

## Step Rules

### 1. Steps Must Appear in Order

```gherkin
# ❌ Wrong: Steps out of order
Scenario: Incorrect step order
  Given initial state
  When perform action
  Then verify result
  When perform another action    # ❌ When cannot follow Then
  Then verify another result

# ✅ Correct: Split into two scenarios
Scenario: First behavior
  Given initial state
  When perform first action
  Then verify first result

Scenario: Second behavior
  Given initial state
  When perform second action
  Then verify second result
```

### 2. Correct Usage of Step Types

- **Given**: Establish initial state (describe the scene)
- **When**: Execute action (trigger behavior)
- **Then**: Verify results (observable output)
- **And/But**: Connect steps of the same type

### 3. Tense and Voice

Always use **present tense + third person**.

```gherkin
# ✅ Correct
Given the Google homepage is displayed
When the user enters "panda" in the search bar
Then links related to "panda" are displayed

# ❌ Wrong: Mixed tenses and persons
Given the user navigated to Google homepage    # ❌ Implies action, not state
When the user entered "panda"                  # ❌ Past tense
Then "panda" related links will be displayed  # ❌ Future tense
```

### 4. Step Structure: Subject + Predicate

Every step must have a complete subject-predicate structure so it can be independently understood and reused.

```gherkin
# ✅ Correct: Complete subject-predicate structure
Given the user navigates to Google homepage
When the user enters "panda" in the search bar
Then the results page displays links related to "panda"
And the results page displays image links for "panda"
And the results page displays video links for "panda"

# ❌ Wrong: Missing subject or predicate
Then the results page displays links related to "panda"
And image links for "panda"           # ❌ Missing subject and predicate
And video links for "panda"           # ❌ Cannot be reused
```

## Step Length Recommendations

- Recommended step count: **3-5 steps**
- Maximum step count: **Single digits (<10 steps)**

### Reduction Techniques

#### Declarative Instead of Imperative

```gherkin
# Imperative - 8 steps
When the user scrolls mouse to search bar
And the user clicks the search bar
And the user types letter "p"
And the user types letter "a"
...

# Declarative - 1 step
When the user enters "panda" in the search bar
```

#### Hide Implementation Details

```gherkin
# ❌ Exposing all details
Given the user has email "user@example.com"
And the user has name "John Doe"
And the user has phone "0912345678"
When the user registers

# ✅ Hidden in step definitions
Given John is a new user
When John registers with valid information
```

## Style and Consistency

- **Third person**: Always use third-person perspective
- **Present tense**: Always use present tense
- **Subject-predicate**: Complete sentence structure

```gherkin
# ✅ Consistent style
Feature: User Authentication

  Scenario: Successful login
    Given the user has a valid account
    When the user enters correct credentials
    Then the user sees the dashboard

# ❌ Inconsistent style
Feature: User Authentication

  Scenario: Successful login
    Given I have an account              # ❌ First person
    When the user entered credentials    # ❌ Past tense
    Then the dashboard will be displayed # ❌ Future tense
```

## Scenario Title Writing Guidelines

### Good Title Characteristics

- **Concise**: One line describing the behavior
- **Clear**: Understandable even to someone unfamiliar with the feature
- **User-facing**: Describes user value

```gherkin
# ✅ Good titles
Scenario: Free members can only see free content
Scenario: Paid members can access premium features
Scenario: Search results are sorted by relevance

# ❌ Bad titles
Scenario: Test 1
Scenario: Check permissions
Scenario: Verify API endpoint response
```

## Handling Known Unknowns

Write scenarios defensively — treat data as behavior examples, not test data.

```gherkin
# ❌ Hardcoded validation
Then results contain "Panda Express"

# ✅ Pattern validation
Then each result is related to the search term "panda"
```

## Describe Behavior, Not Implementation

**Functional requirements belong to features, procedures belong to implementation details.**

```gherkin
# ✅ Functional requirement (describes "what to do")
When Bob logs in

# ❌ Procedure reference (describes "how to do it")
Given I visit "/login"
When I enter "Bob" in the "user name" field
And I enter "tester" in the "password" field
And I click the "login" button
Then I should see the "welcome" page
```