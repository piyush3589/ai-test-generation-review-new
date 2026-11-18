# ai-test-generation-review-new

# AI Test Generation Review

Practice exercises in using an AI model to draft test cases, then reviewing them
critically. The test cases were generated with Claude, and I reviewed them for weak
spots. The specs are short practice specs, not a real product's requirements.

Date: 18 Nov 2026

## How I review AI-generated tests

For every test case I ask three questions:

1. Can pass or fail be decided from the Expected result? (a number or an exact behaviour, no words like "appropriately")
2. Does the Expected result follow from the spec? If it depends on a rule the spec does not state, I ask, or mark it as an assumption.
3. Does it check something new, or does it repeat another case?

## Exercise 1: Duplicate detection (spare parts)

**Spec:** Two records are duplicates only if they describe the same physical part.
Parts that differ in size, model number or material are NOT duplicates, even if the
names look similar. Example: "Bearing 6204 ZZ" and "Bearing 6205 ZZ" are different parts.

**Prompt:** Role (QA engineer for an MDM system), Context (the spec), Task (20 test cases),
Format (table: ID, Type, Input, Action, Expected), Rules (at least 6 negative and 4 boundary
cases, near-duplicate traps, specific Expected results with a reason, no repeats).

**Output:** 20 cases (6 positive, 7 negative, 7 boundary). A selection with assumptions
marked is in `test_cases.md`.

### Weak cases found

| ID | Problem | Fix |
|---|---|---|
| TC18 | Expected depends on a similarity threshold that is not defined, so pass or fail cannot be decided | Ask for the number, or write "assumed 0.85, to be confirmed" |
| TC05 | Assumes "Hex" and "Hexagon" mean the same. Not in the spec | Confirm the synonym rule, or mark as an assumption |
| TC16 | Maximum description length and last-character rule are not in the spec | Ask for the limit before writing Expected |
| TC20 | Assumes trailing whitespace is trimmed. Reasonable, but not in the spec | Keep the test, mark as an assumption |

## Exercise 2: Reorder point

**Spec:** Reorder point (ROP) = (average daily usage x lead time in days) + safety stock.

**Output:** 8 cases (R1 to R8).

### Weak cases found

| ID | Problem | Fix |
|---|---|---|
| R5 | "System rejects the input" has no exact behaviour, and the validation rule is not in the spec | State the exact result (error shown, ROP not calculated) and mark the assumption |
| R6 | "Rounded appropriately" has no number. 3.3 x 7 + 5 = 28.1, and the rounding rule is not in the spec | Expected = 29, assumption: round up, to be confirmed |
| R8 | Converting "2 weeks" to 14 days is not in the spec. The arithmetic (5 x 14 + 20 = 90) is correct | Ask whether weeks are accepted, or mark the conversion as an assumption |
| R7 | Same input and result as R1, so it adds no new check | Remove it, or change the input to cover a new case |

## What I learned

- A test is only good if pass or fail can be verified from the Expected result.
- When the spec is silent, ask, or mark an assumption instead of hiding it.
- An AI drafts quickly, but I am accountable for the tests I run.
- Near-miss negative tests catch false positives, such as wrongly merging two different parts.

## Limits

- The specs are short practice specs. Thresholds and rules marked as assumptions need confirmation.
- Reviewed by one person; no domain expert has checked these.
