# Feature Specification: Subagent Delegation

**Feature Branch**: `spec/006-subagent-delegation`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "Subagent delegation: GN acts as an orchestrator that hands parts of a question to specialist subagents, each an expert in one area of the Grand National (for example engine and turbo, electrical and engine management, drivetrain and chassis, body and interior, buying and history), and combines their findings into one answer in GN's voice. The person still talks to one GN and gets one reply. Simple questions are answered without delegation. Covers questions that span several specialties, a specialist failing or being slow, limits on how much work one question may trigger, and keeping GN's scope, honesty and safety rules across all specialists. Depends on feature 001 and is more useful after features 004 and 005."

## Clarifications

### Session 2026-11-14

- Q: What is delegating to specialists for in this project: to make Bill-GN's answers better, or to prove out the multi-agent pattern? → A: To prove the pattern. It is built to show that orchestrating specialists works reliably within limits. Answer quality is measured and reported but is not a pass or fail condition. Comparing multiple agents with a single agent on performance, accuracy and cost is a separate research follow-up.
- Q: How should the specialists be divided? → A: Three, by area of the car: engine, turbo and fuel; electrical and engine management; drivetrain, chassis and brakes. Everything else Bill-GN answers itself.
- Q: When a specialist needs outside information, should it be able to use the reference collection and live lookups itself? → A: Both, sharing one limit. Specialists can use the reference collection and perform lookups, and all lookups for one question, by anyone, count against a single limit of six. Each time the limit is reached it is recorded, so the limit or the approach can be adjusted. How well this works is evaluated after real usage; if it works poorly, the fallback is that only Bill-GN performs lookups.
- Q: When Bill-GN has consulted its specialists, should the person be told? → A: No. The reply reads as Bill's own answer, with no mention of specialists. Only the records show that delegation happened.
- Q: Once delegation is built and proven, should it be switched on or off in everyday use? → A: Off. The feature ships proven and tested but switched off. Whether to switch it on is evaluated as part of the research into multiple agents against a single agent.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Get a deeper answer to a question that spans specialties (Priority: P1)

A person asks Bill-GN a question that touches several areas of the car at once, such as a symptom
that could be fuel, ignition or boost control, or a plan to raise power that affects the
engine, fuel system and transmission. Bill-GN consults its specialists and returns one answer that
covers each relevant area and how they relate.

**Why this priority**: This is the reason for the feature. A single generalist tends to cover
such questions shallowly or miss an area; the hard, cross-cutting questions are where owners
most need an expert.

**Independent Test**: Ask Bill-GN a review set of cross-specialty questions and have a
knowledgeable reviewer compare the answers with those from Bill-GN without delegation for
completeness and correctness.

**Acceptance Scenarios**:

1. **Given** a question that involves two or more specialties, **When** Bill-GN answers, **Then**
   the answer addresses each relevant area.
2. **Given** specialists have contributed, **When** the person reads the reply, **Then** it
   is one coherent answer in Bill-GN's voice, not separate reports stitched together.
3. **Given** a question where one area's advice affects another (for example, more boost
   requiring more fuel), **When** Bill-GN answers, **Then** the answer explains the connection.
4. **Given** two specialists reach different conclusions, **When** Bill-GN answers, **Then** it
   either reconciles them or tells the person plainly that there are two views and what
   distinguishes them.
5. **Given** a question that lies squarely in one specialty and needs depth, **When** Bill-GN
   answers, **Then** it may consult that one specialist.

---

### User Story 2 - Simple questions stay fast (Priority: P1)

A person asks a simple question. Bill-GN answers it directly, as quickly as it did before this
feature, without involving specialists.

**Why this priority**: Most questions are simple. If delegation makes every answer slower and
costlier, the feature is a net loss.

**Independent Test**: Ask Bill-GN feature 001's representative review set and confirm most
questions are answered without delegation and within feature 001's time limit.

**Acceptance Scenarios**:

1. **Given** a simple factual question, **When** Bill-GN answers, **Then** no specialist is
   consulted.
2. **Given** an off-topic request, **When** Bill-GN responds, **Then** it declines without
   consulting any specialist.
3. **Given** an empty or over-length question, **When** Bill-GN responds, **Then** it responds as
   in feature 001 without consulting any specialist.

---

### User Story 3 - One reply, whatever happens behind the scenes (Priority: P1)

A specialist is slow or fails, or a question would trigger a great deal of work. The person
still gets one reply in reasonable time; Bill-GN answers with what it has and is honest about any
gap.

**Why this priority**: More moving parts mean more ways to fail. The person must never be
worse off than with the single agent of feature 001.

**Independent Test**: Make each specialist unavailable or slow in turn, ask questions that
would use it, and confirm each question still receives exactly one answer within the time
limit.

**Acceptance Scenarios**:

1. **Given** one specialist fails, **When** Bill-GN answers, **Then** it answers using the other
   specialists and its own knowledge, and says which area it could not cover in depth.
2. **Given** a specialist does not finish within its time allowance, **When** that allowance
   passes, **Then** Bill-GN stops waiting and answers without it.
3. **Given** all specialists fail, **When** Bill-GN answers, **Then** it answers as the single
   agent of feature 001 would.
4. **Given** a question that could involve every specialist, **When** Bill-GN handles it, **Then**
   the work stays within the defined limits and one reply is returned.
5. **Given** any question, **When** Bill-GN replies, **Then** the person receives exactly one
   reply and sees no specialist's intermediate output.

---

### User Story 4 - The same rules apply to every specialist (Priority: P2)

Whichever specialists contribute, the answer keeps Bill-GN's scope, honesty and safety rules, and
the owner can see which specialists were used.

**Why this priority**: Each specialist is another place the rules could be weaker. It ranks
below the first three because the final answer is always Bill-GN's, which already enforces them.

**Independent Test**: Run feature 001's review sets for unrelated requests, exact figures and
hazardous work with delegation in place and confirm they still pass; confirm the records show
which specialists handled each question.

**Acceptance Scenarios**:

1. **Given** a specialist contributes an exact figure it is not certain of, **When** Bill-GN
   answers, **Then** the figure is labeled as needing verification, as in feature 001.
2. **Given** a question about hazardous work handled by a specialist, **When** Bill-GN answers,
   **Then** the answer includes the safety warning feature 001 requires.
3. **Given** a specialist is asked about something outside its specialty, **When** it
   responds, **Then** it says so and does not invent an answer.
4. **Given** a question was delegated, **When** the owner reviews the records, **Then** they
   show which specialists were consulted, each one's outcome and how long each took.

---

### Edge Cases

- One specialist uses all six lookups before the others have made any: the others answer
  without lookups; the event is recorded, showing which specialist used them.
- The lookup limit is reached while a specialist is part-way through its work: it stops
  looking things up, answers with what it has, and says what it could not check.

- A question that fits no specialty but is in Bill-GN's scope: Bill-GN answers it directly.
- A specialist wants to hand work to another specialist: not permitted; only Bill-GN delegates.
- Specialists repeat each other: the final answer states each point once.
- The combined material is longer than Bill-GN's reply length rule allows: Bill-GN prioritizes what
  matters most and offers more detail on request.
- A follow-up in a remembered conversation (feature 003): Bill-GN decides on delegation for each
  message; specialists receive the context they need from Bill-GN.
- A specialist needs reference material or a lookup (features 004 and 005): it uses them
  itself, under the same rules as Bill-GN and the shared lookup limit (FR-017).
- A person asks Bill-GN to "ask the engine specialist", or asks whether it uses specialists:
  Bill-GN treats the first as an ordinary question, deciding for itself whether to delegate,
  and answers the second honestly and briefly as the AI assistant it is (feature 001,
  FR-019), without presenting specialists as people.
- A person tries to address or redirect a specialist directly: not possible; people interact
  only with Bill-GN.
- Many delegated questions arrive at once: each is answered; none is dropped.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Bill-GN MUST be able to hand part or all of a question to one or more specialists
  and MUST combine what they return into a single answer.
- **FR-002**: There MUST be three specialists, each with a stated area of expertise: engine,
  turbo and fuel; electrical and engine management; and drivetrain, chassis and brakes.
  Questions outside those areas (for example body and interior, buying, history and values)
  MUST be answered by Bill-GN itself without delegation.
- **FR-003**: The person MUST interact only with Bill-GN and MUST receive exactly one reply per
  question. Specialists' intermediate output MUST NOT be shown to the person.
- **FR-004**: The reply MUST be a single coherent answer in Bill-GN's voice that follows feature
  001's rules for directness and length.
- **FR-005**: Bill-GN MUST answer directly, without delegation, when a question is simple, and
  MUST NOT delegate requests it declines or rejects.
- **FR-006**: When a question spans specialties, the answer MUST address each relevant area
  and MUST explain where one area's advice depends on another's.
- **FR-007**: When specialists disagree, Bill-GN MUST reconcile the difference or state both views
  and what distinguishes them.
- **FR-008**: Each specialist MUST follow Bill-GN's scope, honesty and safety rules (feature 001)
  and MUST say when something is outside its specialty.
- **FR-009**: Bill-GN MUST apply feature 001's honesty and safety rules to the final answer
  regardless of what specialists returned.
- **FR-010**: Each specialist MUST have a time allowance. When a specialist fails or exceeds
  it, Bill-GN MUST answer with what it has and MUST say which area it could not cover in depth.
- **FR-011**: The work one question may trigger MUST be limited: a maximum number of
  specialist consultations per question, and no delegation by specialists.
- **FR-012**: If no specialist can be used, Bill-GN MUST answer as it does in feature 001.
- **FR-013**: Specialists MUST be able to consult the reference collection (feature 004) and
  perform live lookups (feature 005) themselves, where those features are in place, under the
  same rules as Bill-GN.
- **FR-017**: All lookups made for one question, by Bill-GN and by every specialist together,
  MUST count against a single limit of 6 lookups, the same limit that applies to a question
  Bill-GN answers alone (feature 005, FR-009). Once it is reached, no further lookup is made
  for that question, and whoever needed one answers with what it has.
- **FR-018**: Each time the lookup limit is reached, that MUST be recorded with the request's
  correlation identifier, who was refused a lookup (Bill-GN or which specialist), and how many
  lookups each had used, without recording question text. The limit MUST be adjustable by the
  project owner.
- **FR-019**: A reply MUST NOT mention specialists, delegation or other agents, and MUST NOT
  describe colleagues, staff or helpers, which would be invented backstory (feature 001,
  FR-022). When an area could not be covered in depth (FR-010), Bill-GN MUST say so in Bill's
  own voice, as something he could not go into fully, without saying why in terms of
  specialists.
- **FR-014**: Each consultation MUST be recorded with the specialist, its outcome and its
  duration, linked to the request's correlation identifier, without recording question or
  answer text.
- **FR-015**: The project owner MUST be able to add, change or remove a specialist by
  documented steps.
- **FR-016**: Delegation MUST be controlled by a single on or off setting among the project's
  deployment parameters (the `cfn-parameters.json` file, passed to the deployed stack), so
  that the project owner switches it by changing that value and deploying. It MUST be off by
  default, and with it off Bill-GN behaves exactly as it did before this feature. The
  feature's tests are run with it switched on.

### Key Entities

- **Orchestrator**: Bill-GN itself, which reads the question, decides whether to delegate, and
  writes the reply.
- **Specialist**: An expert in one area of the cars in Bill-GN's scope. Attributes: its name, its
  area of expertise, and the rules it follows.
- **Consultation**: One hand-off of work from Bill-GN to a specialist. Attributes: the specialist,
  what was asked, what was returned, the outcome, the duration, and the request it belongs to.
- **Delegation limits**: The maximum number of consultations per question and the time
  allowance per specialist.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Reported, not pass or fail: on a review set of at least 15 cross-specialty
  questions, a knowledgeable reviewer compares answers with and without delegation, without
  knowing which is which, and the share judged better, the same and worse is recorded.
- **SC-002**: On the same set, 100% of delegated questions return exactly one reply that
  combines the specialists' contributions, with no specialist's raw output shown. The share
  of answers that address every area the reviewer considers relevant is recorded.
- **SC-003**: On feature 001's representative review set, at least 70% of questions are
  answered without delegation.
- **SC-004**: 95% of questions answered without delegation meet feature 001's time limit.
- **SC-005**: Soft target: 95% of delegated questions are answered within 60 seconds, and
  none takes longer than 120 seconds. The measured figures are reported; missing them does
  not fail the feature.
- **SC-006**: With each specialist made unavailable in turn, 100% of questions receive exactly
  one answer, and each affected answer states which area was not covered in depth.
- **SC-007**: No question triggers more consultations than the defined maximum, across all
  testing.
- **SC-008**: Feature 001's review sets for unrelated requests, exact figures and hazardous
  work (SC-002, SC-003, SC-004) still pass with delegation in place.
- **SC-009**: 100% of consultations made during testing can be found in the records with
  specialist, outcome and duration.
- **SC-010**: Reported, not pass or fail: the average running cost and time per question
  across feature 001's representative review set, with and without delegation, are recorded.
- **SC-011**: With delegation turned off, Bill-GN's answers and timings match its behavior before
  this feature.
- **SC-012**: Across all testing, no delegated question makes more than 6 lookups in total.
- **SC-013**: 100% of occasions on which the lookup limit is reached during testing appear in
  the records, each showing who was refused and how many lookups each participant had used.
- **SC-014**: Across all delegated replies in testing, 0 mention specialists, delegation or
  other agents, and 0 describe colleagues or helpers.

## Assumptions

- The three specialists were chosen by the project owner (2026-11-14) as enough to prove
  orchestration, including questions that span two or three of them, without more sets of
  instructions to maintain. Others can be added later (FR-015).
- Specialists differ in their focus and depth of instruction, not in what they are allowed to
  do. They are not separate products and are never addressed directly by people.
- The purpose of this feature is to prove that orchestrating specialists works reliably
  within limits (project owner's decision, 2026-11-14). It does not have to produce better
  answers than the single agent to be accepted. The quality, cost and time comparisons are
  recorded so that the separate research into multiple agents against a single agent has data
  to start from.
- Delegation is delivered switched off (project owner's decision, 2026-11-14). Whether to
  switch it on in everyday use is decided as part of the research into multiple agents
  against a single agent, using the comparisons this feature records.
- The defaults are a maximum of 3 consultations per question and about 20 seconds per
  specialist; these may be adjusted during planning.
- Where the Slack features are in place, their time targets are extended for delegated
  questions to the limits in this specification, and the working-on-it indication covers the
  wait.
- Six shared lookups per question is a starting value chosen by the project owner.
  It is to be evaluated after the service has real usage, using the records of when the limit
  is reached. If sharing a limit among specialists works poorly in practice, the fallback is
  for only Bill-GN to perform lookups and pass results to the specialists.
- The limit of 6 lookups is the same whether or not a question is delegated (project owner's
  decision, 2026-11-14); feature 005 was updated to match.
- The time figures in this specification (about 20 seconds per specialist, and the figures in
  SC-005) are soft targets, as in feature 005: measured and reported, and a miss does not
  fail the feature. The limits on the number of consultations and lookups remain firm.
- One specialist using all of the shared lookups before the others is accepted for now. It is
  recorded when it happens (FR-018) and reviewed in the follow-up research.
- This feature works without features 002 to 005, but is expected to add the most value once
  specialists can draw on reference material and lookups.
- Expected usage is low (a small team, tens of questions per day).
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
