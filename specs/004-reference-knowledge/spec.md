# Feature Specification: Reference Knowledge

**Feature Branch**: `spec/004-reference-knowledge`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "Reference knowledge skill: Give GN packaged, vetted Grand National reference material (factory specifications, torque values, fluid capacities, part numbers, service procedures, common faults) that it consults when a question needs exact figures or procedures, so answers come from a vetted source rather than only the model's general recall. GN says when an answer comes from the reference material and names the source; when the reference does not cover something GN falls back to the honesty rules of feature 001. The project owner supplies and maintains the reference material. Depends on feature 001."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Get an exact figure from a vetted source (Priority: P1)

A person asks Bill-GN for an exact value, such as a torque specification or a fluid capacity. Bill-GN
gives the figure from the project's vetted reference material and says where it came from, so
the person can trust it and check it.

**Why this priority**: Feature 001 can only label exact figures as needing verification.
Vetted figures turn Bill-GN from a knowledgeable talker into something an owner can work from, and
this is the gap feature 001 explicitly leaves open.

**Independent Test**: Ask Bill-GN for at least ten exact values that are in the reference
material and confirm each answer states the reference figure exactly and names its source.

**Acceptance Scenarios**:

1. **Given** the reference material contains a value, **When** a person asks for it, **Then**
   Bill-GN states that value exactly as the reference gives it, with its unit.
2. **Given** Bill-GN answers from the reference material, **When** the person reads the answer,
   **Then** it names the source of the figure (for example, the document and section).
3. **Given** the reference material gives different values for different conditions (for
   example, by model year or component variant), **When** a person asks without saying which,
   **Then** Bill-GN gives the value for the 1987 Grand National and notes that other variants
   differ.
4. **Given** a question that needs several reference values, **When** Bill-GN answers, **Then**
   each value is the reference value and is attributed.

---

### User Story 2 - Bill-GN is clear about what the reference does not cover (Priority: P1)

A person asks for an exact value or procedure that the reference material does not contain.
Bill-GN does not pass off general recall as vetted fact: it says the reference does not cover it
and falls back to the honesty rules of feature 001.

**Why this priority**: A source label that is sometimes wrong is worse than no label. The
value of this feature depends on people being able to tell vetted answers from the rest.

**Independent Test**: Ask Bill-GN for exact values that are deliberately absent from the reference
material and confirm no answer claims the reference as its source.

**Acceptance Scenarios**:

1. **Given** the reference material does not contain a value, **When** a person asks for it,
   **Then** Bill-GN says the reference does not cover it, gives its best understanding labeled as
   unverified, and recommends confirming against an authoritative source.
2. **Given** an answer that mixes reference figures and general knowledge, **When** the
   person reads it, **Then** they can tell which statements come from the reference.
3. **Given** Bill-GN's general recall disagrees with the reference material, **When** Bill-GN answers,
   **Then** the reference value is the one given.
4. **Given** a general question that needs no exact figure (for example, the car's history),
   **When** Bill-GN answers, **Then** it answers as in feature 001 and does not cite the reference
   unnecessarily.

---

### User Story 3 - Follow a vetted procedure (Priority: P2)

A person asks how to do a job that the reference material describes. Bill-GN walks them through
the reference procedure in order, with the reference's figures and warnings.

**Why this priority**: Procedures are where a wrong step causes damage, but figures alone
already deliver most of the value, so this follows the first two stories.

**Independent Test**: Ask Bill-GN how to do at least five jobs covered by the reference material
and confirm the steps, order, figures and warnings match the reference.

**Acceptance Scenarios**:

1. **Given** the reference material describes a procedure, **When** a person asks how to do
   it, **Then** Bill-GN gives the steps in the reference's order, without omitting a step that
   affects safety or correctness.
2. **Given** the reference procedure includes warnings, **When** Bill-GN gives the procedure,
   **Then** those warnings are included along with the safety warnings feature 001 requires.
3. **Given** a procedure too long for one reply, **When** Bill-GN answers, **Then** it gives a
   complete summary of the steps and offers the detail of any step.

---

### User Story 4 - The owner maintains the reference material (Priority: P2)

The project owner adds, corrects or removes reference material, and Bill-GN's answers reflect the
change after it is published, without changes to Bill-GN itself.

**Why this priority**: The material will start incomplete and contain mistakes. If updating it
is hard it will go stale and the feature loses its point.

**Independent Test**: Correct a value in the reference material, publish it by the documented
steps, and confirm Bill-GN gives the corrected value.

**Acceptance Scenarios**:

1. **Given** the owner corrects a value and publishes the change, **When** a person asks for
   that value, **Then** Bill-GN gives the corrected value.
2. **Given** the owner adds material on a new topic and publishes it, **When** a person asks
   about that topic, **Then** Bill-GN answers from the new material.
3. **Given** reference material that is malformed or missing required details such as its
   source, **When** the owner tries to publish it, **Then** it is rejected with a message
   saying what is wrong, and Bill-GN continues to use the last good version.

---

### Edge Cases

- The reference material is unavailable when a question arrives: Bill-GN answers under feature
  001's rules, does not claim any reference source, and the failure is recorded.
- Two parts of the reference material disagree: Bill-GN gives both values, names both sources and
  says they conflict.
- The person asks for a value in different units from the reference: Bill-GN gives the reference
  value in its original units and may add a conversion, labeled as a conversion.
- The person asks about a related car (GNX, other model years) that the reference covers only
  for 1987: Bill-GN says the reference figure is for the 1987 Grand National.
- The person asks Bill-GN to quote long passages of the reference material: Bill-GN summarizes and
  cites; it does not reproduce large portions verbatim.
- The person asks where a figure came from in a follow-up (with feature 003): Bill-GN can name the
  source it used.
- The reference material is empty at first release: Bill-GN behaves exactly as in feature 001.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Bill-GN MUST have access to a body of reference material about the cars in its scope,
  supplied by the project owner.
- **FR-002**: When a question calls for an exact figure or a procedure that the reference
  material contains, Bill-GN MUST answer from the reference material and MUST state figures exactly
  as the reference gives them, including units.
- **FR-003**: When Bill-GN answers from the reference material, it MUST name the source in the
  answer, at least to the level of the document and section.
- **FR-004**: Bill-GN MUST NOT attribute to the reference material anything that is not in it.
- **FR-005**: When the reference material does not cover what was asked, Bill-GN MUST say so and
  MUST apply feature 001's rules for uncertain figures and unknown answers.
- **FR-006**: When the reference material and Bill-GN's general knowledge disagree, the reference
  material MUST take precedence.
- **FR-007**: In an answer that combines reference material and general knowledge, the reader
  MUST be able to tell which statements come from the reference.
- **FR-008**: When giving a reference procedure, Bill-GN MUST preserve the order of steps and MUST
  include the reference's warnings.
- **FR-009**: Each item of reference material MUST record its source, and SHOULD record which
  cars and model years it applies to.
- **FR-010**: The project owner MUST be able to add, correct and remove reference material and
  publish the change by documented steps, without changing Bill-GN's other behavior.
- **FR-011**: Reference material MUST be checked when it is published; material that is
  malformed or lacks a source MUST be rejected with a clear message, and Bill-GN MUST keep using
  the last good version.
- **FR-012**: If the reference material cannot be used when a question arrives, Bill-GN MUST still
  answer under feature 001's rules, MUST NOT cite the reference, and the failure MUST be
  recorded.
- **FR-013**: Request records MUST show whether the reference material was consulted and
  which items were used, without recording the question or answer text.
- **FR-014**: Bill-GN's persona, scope, honesty and safety rules (feature 001) MUST continue to
  apply; reference material adds knowledge and does not widen Bill-GN's scope.
- **FR-015**: Bill-GN MUST NOT reproduce large portions of reference material verbatim.
- **FR-016**: An initial set of reference material MUST be delivered with this feature,
  sufficient to demonstrate the success criteria.

### Key Entities

- **Reference item**: One piece of vetted knowledge, such as a specification table, a
  procedure or a fault description. Attributes: its content, its source, the cars and years
  it applies to, and when it was last reviewed.
- **Reference collection**: The published set of reference items Bill-GN uses. Has a version, so
  the owner can tell which set Bill-GN is answering from.
- **Citation**: The source named in an answer for a statement taken from a reference item.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On a review set of at least 20 questions asking for exact figures that are in
  the reference material, 100% of answers state the reference figure exactly, and none states
  a different figure.
- **SC-002**: On the same set, at least 95% of answers name the correct source.
- **SC-003**: On a review set of at least 10 questions asking for exact figures that are not
  in the reference material, 0 answers attribute a figure to the reference, and 100% follow
  feature 001's rule for uncertain figures.
- **SC-004**: On at least 5 questions about procedures in the reference material, a reviewer
  finds the steps complete and in order, with the reference's warnings, in 100% of answers.
- **SC-005**: After the owner publishes a corrected value, Bill-GN gives the corrected value in
  100% of trials, within 30 minutes of publishing.
- **SC-006**: The owner can correct a value and publish it in under 15 minutes using only the
  documented steps.
- **SC-007**: Malformed reference material is rejected at publishing in 100% of test cases,
  and Bill-GN's answers are unaffected.
- **SC-008**: With the reference material made unavailable, 100% of questions are still
  answered and none cites the reference.
- **SC-009**: Feature 001's success criteria for relevance, declining off-topic requests and
  safety warnings (SC-001, SC-002, SC-004) still pass with this feature in place.
- **SC-010**: 95% of questions are answered within the time limit set in feature 001.

## Assumptions

- The project owner supplies the reference material and is responsible for its accuracy and
  for having the right to use it. Material is written or summarized by the owner; factory
  manuals and other copyrighted works are not copied in wholesale.
- The initial collection is small: on the order of tens of items covering common
  specifications, fluid capacities, torque values, maintenance procedures and well-known
  faults for the 1987 Grand National.
- Reference material is text, including tables. Diagrams, photographs and scanned pages are
  out of scope.
- The material is the same for everyone who asks. Per-person or per-car material is out of
  scope.
- Updates are made by the project owner through the project's normal change process, not by
  people in Slack. Publishing a change within tens of minutes is fast enough.
- This feature works with or without the Slack features. Source names appear in the answer
  text, so they show wherever Bill-GN's answers are shown.
- Looking things up on the internet or in outside systems is out of scope here; that is
  feature 005.
- Expected usage is low (a small team, tens of questions per day).
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
