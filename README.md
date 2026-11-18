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

Action for every case: run duplicate detection on Part A and Part B.

| ID | Type | Part A / Part B | Expected |
|---|---|---|---|
| TC01 | Positive | "Bearing 6204 ZZ" / "Bearing 6204 ZZ" | Duplicate flag, exact match |
| TC02 | Positive | "BEARING 6204-ZZ" / "bearing 6204 ZZ" | Duplicate, only case and hyphen differ |
| TC03 | Positive | "Gate Valve 2 in SS304" / "Gate Valve 2 inch SS304" | Duplicate, "in" and "inch" are the same unit |
| TC04 | Positive | "SS304 Gate Valve 2 inch" / "Gate Valve 2 inch SS304" | Duplicate, only word order differs |
| TC05 | Positive | "Hex Bolt M10x50 GI" / "Hexagon Bolt M10x50 GI" | Duplicate, same size and material |
| TC06 | Positive | "Washer 10 mm" / "Washer 10mm" | Duplicate, only a space differs |
| TC07 | Negative | "Bearing 6204 ZZ" / "Bearing 6205 ZZ" | Not duplicate, model number differs |
| TC08 | Negative | "Gate Valve 2 inch SS304" / "Gate Valve 3 inch SS304" | Not duplicate, size differs |
| TC09 | Negative | "Gate Valve 2 inch SS304" / "Gate Valve 2 inch SS316" | Not duplicate, material differs |
| TC10 | Negative | "Hex Bolt M10x50" / "Hex Bolt M10x60" | Not duplicate, length differs |
| TC11 | Negative | "Bolt M12 LH" / "Bolt M12 RH" | Not duplicate, thread direction differs |
| TC12 | Negative | "Flange 4 inch 150#" / "Flange 4 inch 300#" | Not duplicate, pressure rating differs |
| TC13 | Negative | "Bearing 6204 ZZ" / "Bearing 6204 2RS" | Not duplicate, shield vs seal type differs |
| TC14 | Boundary | Both descriptions empty | Not duplicate, "missing data" error |
| TC15 | Boundary | "Bearing 6204 ZZ" / empty | Not duplicate, error or skip, no crash |
| TC16 | Boundary | 500-character description, identical in both / last character differs in one | Identical: duplicate. Different: per spec |
| TC17 | Boundary | `Valve 2" SS304` / `Valve 2 in SS304` | Duplicate, special character handled |
| TC18 | Boundary | Similarity score exactly at the threshold | Per spec (score >= threshold is a duplicate) |
| TC19 | Boundary | A record compared with itself | Not duplicate, same record ID, self-match skipped |
| TC20 | Boundary | "Bearing 6204 ZZ " (trailing space) / "Bearing 6204 ZZ" | Duplicate, whitespace trimmed |

### Weak cases found

| ID | Problem | Fix |
|---|---|---|
| TC18 | Expected depends on a similarity threshold that is not defined, so pass or fail cannot be decided | Ask for the number, or write "assumed 0.85, to be confirmed" |
| TC05 | Assumes "Hex" and "Hexagon" mean the same. Not in the spec | Confirm the synonym rule, or mark as an assumption |
| TC16 | Maximum description length and last-character rule are not in the spec | Ask for the limit before writing Expected |
| TC20 | Assumes trailing whitespace is trimmed. Reasonable, but not in the spec | Keep the test, mark as an assumption |

## Exercise 2: Reorder point

**Spec:** Reorder point (ROP) = (average daily usage x lead time in days) + safety stock.

**Prompt:** Role (QA engineer for an inventory system), Context (the ROP formula), Task (8 test cases),
Format (table: ID, Type, Input, Expected), Rules (positive, negative and boundary cases, specific
Expected results, mark assumptions).

**Output:** 8 cases (R1 to R8).


## Exercise 2: Reorder point

Spec: ROP = (average daily usage x lead time in days) + safety stock.

| ID | Type | Input | Expected |
|---|---|---|---|
| R1 | Positive | usage 5/day, lead time 10 days, safety stock 20 | ROP = 70 (5 x 10 + 20) |
| R2 | Positive | usage 2/day, lead time 7 days, safety stock 0 | ROP = 14 |
| R3 | Boundary | usage 0, lead time 10 days, safety stock 20 | ROP = 20 |
| R4 | Boundary | usage 5/day, lead time 0 days, safety stock 20 | ROP = 20 |
| R5 | Negative | usage -5/day, lead time 10 days, safety stock 20 | System rejects the input |
| R6 | Positive | usage 3.3/day, lead time 7 days, safety stock 5 | ROP is rounded appropriately |
| R7 | Positive | usage 5/day, lead time 10 days, safety stock 20 | ROP is 70 |
| R8 | Negative | usage 5/day, lead time 2 weeks, safety stock 20 | System converts to 14 days and gives 90 |

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
