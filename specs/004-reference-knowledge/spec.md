# Feature Specification: Reference Knowledge

**Feature Branch**: `spec/004-reference-knowledge`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "Reference knowledge skill: Give GN packaged, vetted Grand National reference material (factory specifications, torque values, fluid capacities, part numbers, service procedures, common faults) that it consults when a question needs exact figures or procedures, so answers come from a vetted source rather than only the model's general recall. GN says when an answer comes from the reference material and names the source; when the reference does not cover something GN falls back to the honesty rules of feature 001. The project owner supplies and maintains the reference material. Depends on feature 001."

## Clarifications

### Session 2026-11-14

- Q: Where will Bill-GN's reference material come from? → A: Four kinds of source, each item labelled with its kind: factory data; Bill's own practices; community sources (turbobuick.com, gbodyforum.com, turbobuicks.com, gnttype.org and the Buick Grand National and GNX forum on buickforums.com); and manuals, publications and other information from the vendors of third-party parts.
- Q: When sources disagree about the same thing, whose word should Bill-GN lead with? → A: Bill's. Bill-GN leads with Bill's practice where one exists, always states the factory figure alongside it, and mentions community or vendor sources after. For a third-party part, the vendor's figure leads.
- Q: Who has to approve a reference item before Bill-GN is allowed to use it? → A: The project owner alone. Every item, of every kind, is approved by the owner through the project's normal change process.
- Q: Who writes the first set of reference items that ships with this feature? → A: The implementer drafts items from the sources the owner names (factory data, the named forums, vendor publications), each with its source cited, and the owner checks and approves each one. Items marked as Bill's practice come from the owner.
- Q: What should the first collection of reference items concentrate on? → A: Maintenance essentials: factory specifications, fluid types and capacities, torque values and service intervals for the 1987 Grand National. Troubleshooting and performance material are added afterwards as content, through the same drafting and approval steps.

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

- A question about a car fitted with a third-party part (for example, an aftermarket turbo or
  fuel pump): Bill-GN uses the vendor's figures for that part where the reference has them,
  says they are the vendor's, and does not apply factory figures for the part it replaced.
- A community item and a factory item disagree: Bill-GN gives both, says which is which, and
  does not present the community finding as a factory figure.

- The reference material is unavailable when a question arrives: Bill-GN answers under feature
  001's rules, does not claim any reference source, and the failure is recorded.
- Two reference items of the same kind disagree (for example, two factory publications):
  Bill-GN gives both values, names both sources and says they conflict. Items of different
  kinds are ordered as FR-019 sets out.
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
- **FR-009**: Each item of reference material MUST record its source and the kind of source
  it comes from, and SHOULD record which cars and model years it applies to. The kinds are:
  factory (GM service manuals and publications); Bill (the real Bill's own practices and
  preferences); community (the forums named in Assumptions); and vendor (manuals,
  publications and other information from the makers of third-party parts).
- **FR-017**: When Bill-GN answers from the reference material, the answer MUST make the kind
  of source clear, so a person can tell a factory figure from Bill's practice, a community
  finding or a vendor's instruction.
- **FR-018**: A reference item about a third-party part MUST name the part and its vendor,
  and Bill-GN MUST apply it only to that part, not to the factory part it replaces.
- **FR-019**: When reference items of different kinds disagree about the same thing, Bill-GN
  MUST lead with Bill's practice where one exists, MUST state the factory figure alongside it,
  and MAY then mention community or vendor sources. Where the question concerns a third-party
  part, the vendor's figure for that part MUST lead. Each figure MUST be labelled with its
  kind of source.
- **FR-020**: Where Bill's practice departs from the factory figure on work that affects
  safety, the answer MUST say so and include the safety warning that feature 001 requires.
- **FR-021**: A reference item MUST NOT be used by Bill-GN until the project owner has
  approved it through the project's normal change process. This applies to every kind of
  source, including items marked as Bill's practice and community items.
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
  sufficient to demonstrate the success criteria. Factory, community and vendor items in it
  are drafted during implementation from the named sources, each citing the specific source
  it was taken from, and each MUST be approved by the project owner before use (FR-021).
  Items marked as Bill's practice are written by the project owner.
- **FR-022**: A drafted item MUST be taken from a source that was actually read, and MUST
  cite it precisely enough for the owner to check it (for example, the publication and
  section, or the address of the forum thread). An item MUST NOT be drafted from general
  recall.

### Key Entities

- **Reference item**: One piece of vetted knowledge, such as a specification table, a
  procedure or a fault description. Attributes: its content, its source, the kind of source
  (factory, Bill, community or vendor), the cars and years it applies to, the third-party
  part and vendor it concerns (if any), and when it was last reviewed.
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
- **SC-011**: On a review set of at least 10 questions where the reference holds both Bill's
  practice and a factory figure that differs, 100% of answers lead with Bill's practice, state
  the factory figure, and label each correctly.
- **SC-012**: On a review set of at least 5 questions about third-party parts with vendor
  items in the reference, 100% of answers lead with the vendor's figure and none applies a
  factory figure to the third-party part.

## Assumptions

- The project owner supplies the reference material and is responsible for its accuracy and
  for having the right to use it. Material is written or summarized by the owner, with the
  source noted; factory manuals, forum posts, vendor publications and other copyrighted works
  are not copied in wholesale.
- Community sources for reference items are: turbobuick.com, gbodyforum.com, turbobuicks.com,
  gnttype.org, and the Buick Grand National and GNX forum on buickforums.com. A community
  item is something the owner has chosen and written up from these sites; this feature does
  not search them live. Live lookups of outside sites are feature 005.
- The project owner is the sole approver of reference items (clarified 2026-11-14). The real
  Bill does not review them; the owner is responsible for items marked as Bill's practice
  being a fair account of what he does.
- Factory publications that are not freely available must be provided by the project owner
  for items to be drafted from them. Where a source cannot be read, no item is drafted from
  it.
- Vendor sources are the manuals, publications and other information published by the makers
  of third-party parts that the cars in scope may be fitted with.
- The initial collection covers maintenance essentials for the 1987 Grand National: factory
  specifications, fluid types and capacities, torque values and service intervals. It is on
  the order of tens of items.
- Troubleshooting material (known faults, symptoms, diagnosis) and performance material
  (modifications, tuning settings, third-party parts with vendors' figures) are not part of
  the initial collection. They are added later as content only, in batches, using the
  publishing steps this feature delivers (FR-010) and the same drafting and approval rules
  (FR-021, FR-022). Adding them needs no new feature and no change to Bill-GN. Each batch is
  tracked as its own issue.
- Until those batches exist, Bill-GN answers troubleshooting and performance questions under
  feature 001's rules and says the reference does not cover them.
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
