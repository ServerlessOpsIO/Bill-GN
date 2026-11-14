# Feature Specification: Subagent Delegation

**Feature Branch**: `spec/006-subagent-delegation`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "Subagent delegation: GN acts as an orchestrator that hands parts of a question to specialist subagents, each an expert in one area of the Grand National (for example engine and turbo, electrical and engine management, drivetrain and chassis, body and interior, buying and history), and combines their findings into one answer in GN's voice. The person still talks to one GN and gets one reply. Simple questions are answered without delegation. Covers questions that span several specialties, a specialist failing or being slow, limits on how much work one question may trigger, and keeping GN's scope, honesty and safety rules across all specialists. Depends on feature 001 and is more useful after features 004 and 005."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Get a deeper answer to a question that spans specialties (Priority: P1)

A person asks GN a question that touches several areas of the car at once, such as a symptom
that could be fuel, ignition or boost control, or a plan to raise power that affects the
engine, fuel system and transmission. GN consults its specialists and returns one answer that
covers each relevant area and how they relate.

**Why this priority**: This is the reason for the feature. A single generalist tends to cover
such questions shallowly or miss an area; the hard, cross-cutting questions are where owners
most need an expert.

**Independent Test**: Ask GN a review set of cross-specialty questions and have a
knowledgeable reviewer compare the answers with those from GN without delegation for
completeness and correctness.

**Acceptance Scenarios**:

1. **Given** a question that involves two or more specialties, **When** GN answers, **Then**
   the answer addresses each relevant area.
2. **Given** specialists have contributed, **When** the person reads the reply, **Then** it
   is one coherent answer in GN's voice, not separate reports stitched together.
3. **Given** a question where one area's advice affects another (for example, more boost
   requiring more fuel), **When** GN answers, **Then** the answer explains the connection.
4. **Given** two specialists reach different conclusions, **When** GN answers, **Then** it
   either reconciles them or tells the person plainly that there are two views and what
   distinguishes them.
5. **Given** a question that lies squarely in one specialty and needs depth, **When** GN
   answers, **Then** it may consult that one specialist.

---

### User Story 2 - Simple questions stay fast (Priority: P1)

A person asks a simple question. GN answers it directly, as quickly as it did before this
feature, without involving specialists.

**Why this priority**: Most questions are simple. If delegation makes every answer slower and
costlier, the feature is a net loss.

**Independent Test**: Ask GN feature 001's representative review set and confirm most
questions are answered without delegation and within feature 001's time limit.

**Acceptance Scenarios**:

1. **Given** a simple factual question, **When** GN answers, **Then** no specialist is
   consulted.
2. **Given** an off-topic request, **When** GN responds, **Then** it declines without
   consulting any specialist.
3. **Given** an empty or over-length question, **When** GN responds, **Then** it responds as
   in feature 001 without consulting any specialist.

---

### User Story 3 - One reply, whatever happens behind the scenes (Priority: P1)

A specialist is slow or fails, or a question would trigger a great deal of work. The person
still gets one reply in reasonable time; GN answers with what it has and is honest about any
gap.

**Why this priority**: More moving parts mean more ways to fail. The person must never be
worse off than with the single agent of feature 001.

**Independent Test**: Make each specialist unavailable or slow in turn, ask questions that
would use it, and confirm each question still receives exactly one answer within the time
limit.

**Acceptance Scenarios**:

1. **Given** one specialist fails, **When** GN answers, **Then** it answers using the other
   specialists and its own knowledge, and says which area it could not cover in depth.
2. **Given** a specialist does not finish within its time allowance, **When** that allowance
   passes, **Then** GN stops waiting and answers without it.
3. **Given** all specialists fail, **When** GN answers, **Then** it answers as the single
   agent of feature 001 would.
4. **Given** a question that could involve every specialist, **When** GN handles it, **Then**
   the work stays within the defined limits and one reply is returned.
5. **Given** any question, **When** GN replies, **Then** the person receives exactly one
   reply and sees no specialist's intermediate output.

---

### User Story 4 - The same rules apply to every specialist (Priority: P2)

Whichever specialists contribute, the answer keeps GN's scope, honesty and safety rules, and
the owner can see which specialists were used.

**Why this priority**: Each specialist is another place the rules could be weaker. It ranks
below the first three because the final answer is always GN's, which already enforces them.

**Independent Test**: Run feature 001's review sets for unrelated requests, exact figures and
hazardous work with delegation in place and confirm they still pass; confirm the records show
which specialists handled each question.

**Acceptance Scenarios**:

1. **Given** a specialist contributes an exact figure it is not certain of, **When** GN
   answers, **Then** the figure is labeled as needing verification, as in feature 001.
2. **Given** a question about hazardous work handled by a specialist, **When** GN answers,
   **Then** the answer includes the safety warning feature 001 requires.
3. **Given** a specialist is asked about something outside its specialty, **When** it
   responds, **Then** it says so and does not invent an answer.
4. **Given** a question was delegated, **When** the owner reviews the records, **Then** they
   show which specialists were consulted, each one's outcome and how long each took.

---

### Edge Cases

- A question that fits no specialty but is in GN's scope: GN answers it directly.
- A specialist wants to hand work to another specialist: not permitted; only GN delegates.
- Specialists repeat each other: the final answer states each point once.
- The combined material is longer than GN's reply length rule allows: GN prioritizes what
  matters most and offers more detail on request.
- A follow-up in a remembered conversation (feature 003): GN decides on delegation for each
  message; specialists receive the context they need from GN.
- A specialist needs reference material or a lookup (features 004 and 005): it uses the same
  material and approved tools as GN, under the same rules and limits.
- A person asks GN to "ask the engine specialist": GN treats it as an ordinary question and
  decides for itself whether to delegate.
- A person tries to address or redirect a specialist directly: not possible; people interact
  only with GN.
- Many delegated questions arrive at once: each is answered; none is dropped.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: GN MUST be able to hand part or all of a question to one or more specialists
  and MUST combine what they return into a single answer.
- **FR-002**: There MUST be a defined set of specialists that together cover GN's scope, each
  with a stated area of expertise.
- **FR-003**: The person MUST interact only with GN and MUST receive exactly one reply per
  question. Specialists' intermediate output MUST NOT be shown to the person.
- **FR-004**: The reply MUST be a single coherent answer in GN's voice that follows feature
  001's rules for directness and length.
- **FR-005**: GN MUST answer directly, without delegation, when a question is simple, and
  MUST NOT delegate requests it declines or rejects.
- **FR-006**: When a question spans specialties, the answer MUST address each relevant area
  and MUST explain where one area's advice depends on another's.
- **FR-007**: When specialists disagree, GN MUST reconcile the difference or state both views
  and what distinguishes them.
- **FR-008**: Each specialist MUST follow GN's scope, honesty and safety rules (feature 001)
  and MUST say when something is outside its specialty.
- **FR-009**: GN MUST apply feature 001's honesty and safety rules to the final answer
  regardless of what specialists returned.
- **FR-010**: Each specialist MUST have a time allowance. When a specialist fails or exceeds
  it, GN MUST answer with what it has and MUST say which area it could not cover in depth.
- **FR-011**: The work one question may trigger MUST be limited: a maximum number of
  specialist consultations per question, and no delegation by specialists.
- **FR-012**: If no specialist can be used, GN MUST answer as it does in feature 001.
- **FR-013**: Specialists MUST use only the reference material (feature 004) and approved
  tools (feature 005) available to GN, under the same rules and limits, where those features
  are in place.
- **FR-014**: Each consultation MUST be recorded with the specialist, its outcome and its
  duration, linked to the request's correlation identifier, without recording question or
  answer text.
- **FR-015**: The project owner MUST be able to add, change or remove a specialist by
  documented steps.
- **FR-016**: Delegation MUST be able to be turned off, returning GN to its behavior without
  specialists.

### Key Entities

- **Orchestrator**: GN itself, which reads the question, decides whether to delegate, and
  writes the reply.
- **Specialist**: An expert in one area of the cars in GN's scope. Attributes: its name, its
  area of expertise, and the rules it follows.
- **Consultation**: One hand-off of work from GN to a specialist. Attributes: the specialist,
  what was asked, what was returned, the outcome, the duration, and the request it belongs to.
- **Delegation limits**: The maximum number of consultations per question and the time
  allowance per specialist.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On a review set of at least 15 cross-specialty questions, a knowledgeable
  reviewer comparing answers with and without delegation, without knowing which is which,
  judges the delegated answer more complete or more correct for at least 60% and worse for no
  more than 10%.
- **SC-002**: On the same set, at least 90% of delegated answers address every area the
  reviewer considers relevant.
- **SC-003**: On feature 001's representative review set, at least 70% of questions are
  answered without delegation.
- **SC-004**: 95% of questions answered without delegation meet feature 001's time limit.
- **SC-005**: 95% of delegated questions are answered within 60 seconds, and none takes longer
  than 120 seconds.
- **SC-006**: With each specialist made unavailable in turn, 100% of questions receive exactly
  one answer, and each affected answer states which area was not covered in depth.
- **SC-007**: No question triggers more consultations than the defined maximum, across all
  testing.
- **SC-008**: Feature 001's review sets for unrelated requests, exact figures and hazardous
  work (SC-002, SC-003, SC-004) still pass with delegation in place.
- **SC-009**: 100% of consultations made during testing can be found in the records with
  specialist, outcome and duration.
- **SC-010**: The average running cost per question across feature 001's representative
  review set is no more than twice its cost without delegation.
- **SC-011**: With delegation turned off, GN's answers and timings match its behavior before
  this feature.

## Assumptions

- The initial specialists are: engine and turbo; electrical and engine management; drivetrain
  and chassis (transmission, rear axle, suspension, brakes); body and interior; and buying,
  history and values. The exact set is an open question for the clarify phase.
- Specialists differ in their focus and depth of instruction, not in what they are allowed to
  do. They are not separate products and are never addressed directly by people.
- Whether delegation is worth its extra time and cost for this project is unproven; SC-001 and
  SC-010 are the test, and the ability to turn it off (FR-016) is the fallback.
- The defaults are a maximum of 3 consultations per question and about 20 seconds per
  specialist; these may be adjusted during planning.
- Where the Slack features are in place, their time targets are extended for delegated
  questions to the limits in this specification, and the working-on-it indication covers the
  wait.
- This feature works without features 002 to 005, but is expected to add the most value once
  specialists can draw on reference material and lookups.
- Expected usage is low (a small team, tens of questions per day).
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
