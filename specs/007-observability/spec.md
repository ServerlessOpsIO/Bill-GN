# Feature Specification: Observability

**Feature Branch**: `spec/007-observability`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "Observability: the team can see whether Bill-GN is working, find out quickly when it is not, and diagnose a single failed question, using the team's existing monitoring (Datadog, fed from AWS CloudWatch by the environment's existing setup). Covers: all of Bill-GN's components writing structured logs that reach the monitoring system without per-component setup; failures (including the failed requests feature 002 retains) being visible there; a defined set of metrics for usage, errors, speed, limits reached and cost; traces that follow one question from Slack to the answer; a dashboard; deciding which conditions alert someone; and never putting question or answer text or secrets into logs, metrics or traces. Applies across features 001 to 006."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Find out what happened to one question (Priority: P1)

Someone reports that Bill-GN did not answer, or answered badly. An operator takes what they
know (who asked and roughly when, or an identifier from a reply) and, in the team's existing
monitoring system, finds that question's path from arrival to outcome: what each part of
Bill-GN did with it, how long each step took, and where it failed.

**Why this priority**: Without this, every problem report ends in guesswork. It is also the
foundation the other stories build on: if single requests cannot be followed, totals and
alerts cannot be trusted.

**Independent Test**: Ask Bill-GN a set of questions including some forced to fail at
different points, then find each one in the monitoring system from its identifier alone and
state where and why it failed.

**Acceptance Scenarios**:

1. **Given** a question that was answered, **When** an operator searches the monitoring system
   for its correlation identifier, **Then** they see every step it passed through, in order,
   with the outcome and duration of each.
2. **Given** a question that failed, **When** an operator looks it up, **Then** they can see
   which step failed and the kind of failure, without reading application code.
3. **Given** only the person who asked and an approximate time, **When** an operator searches,
   **Then** they can narrow down to that person's requests in that period and find the one in
   question.
4. **Given** a request that was retained as failed (feature 002), **When** an operator views
   the monitoring system, **Then** that failed request is visible there and linked to its
   records.
5. **Given** any record of any request, **When** an operator reads it, **Then** it contains no
   question text, no answer text and no secrets.

---

### User Story 2 - See at a glance whether Bill-GN is healthy (Priority: P1)

An operator opens one view and sees whether Bill-GN is working: how many questions it is
getting, how many fail, how fast it answers, and whether anything unusual is happening.

**Why this priority**: The project owner chose not to be alerted in feature 002, so looking
is how problems get noticed. Looking has to be quick and reliable or it will not happen.

**Independent Test**: Generate a known pattern of traffic (a number of answered questions,
declines, failures and limit refusals) and confirm the view shows matching numbers.

**Acceptance Scenarios**:

1. **Given** Bill-GN has been in use, **When** an operator opens the health view, **Then**
   they see, for a chosen period: questions received, answers, declines, failures, and the
   share of questions that failed.
2. **Given** the health view, **When** an operator reads it, **Then** they see how long
   answers take, including the slow end and not only the average.
3. **Given** the health view, **When** an operator reads it, **Then** they see how many
   questions were refused by a usage limit and how many requests were rejected as not genuine
   or not permitted.
4. **Given** a problem is in progress, **When** an operator opens the view, **Then** the
   problem is apparent within one minute of looking, without running a search.
5. **Given** a part of Bill-GN that a later feature adds (memory, reference material, lookups,
   specialists), **When** that feature is in place, **Then** its activity and failures appear
   in the same view.

---

### User Story 3 - Know what Bill-GN costs (Priority: P2)

The project owner sees how much Bill-GN is being used and what that usage costs, and can
tell which kind of work is driving the cost.

**Why this priority**: The owner chose the lowest-cost model and added usage limits because
cost matters. Without visibility, neither choice can be checked.

**Independent Test**: Run a known set of questions and confirm the reported usage matches,
and that estimated cost can be read per day.

**Acceptance Scenarios**:

1. **Given** Bill-GN has been in use, **When** the owner views usage, **Then** they see the
   amount of model usage per day and an estimate of its cost.
2. **Given** usage figures, **When** the owner reads them, **Then** they can tell how much
   comes from each kind of work Bill-GN does, as later features add them (plain answers,
   remembered conversations, lookups, specialists).
3. **Given** a day with no questions, **When** the owner views usage, **Then** it shows no
   usage-based cost.

---

### User Story 4 - Be told when something is badly wrong (Priority: P3)

For the few conditions the owner decides are worth interrupting someone, the monitoring
system raises an alert to the right place.

**Why this priority**: The owner's current position is that recording is enough. This story
exists so that the decision is explicit and the alert set, even if empty at first, is defined
in one place and easy to change.

**Independent Test**: Trigger each defined alert condition and confirm the alert arrives
where specified; confirm that conditions not on the list raise no alert.

**Acceptance Scenarios**:

1. **Given** the list of alert conditions the owner has approved, **When** one of those
   conditions occurs, **Then** an alert is raised to the specified place within the specified
   time.
2. **Given** a failure that is not on the list, **When** it occurs, **Then** it is recorded
   and visible but raises no alert.
3. **Given** the owner wants to add, change or remove an alert, **When** they follow the
   documented steps, **Then** the change takes effect without changing Bill-GN itself.

---

### Edge Cases

- The monitoring system or the path to it is unavailable: Bill-GN keeps answering questions;
  records are delivered when the path recovers where the existing setup allows, and a gap is
  visible as a gap, not as "no errors".
- A component fails before it can write a record (for example, it never starts): the absence
  is detectable, because the step before it shows the request going in and nothing coming
  out.
- A failure happens while recording a failure: the original request is still handled; the
  recording problem itself is visible.
- A person's question text or an answer appears in a record by mistake (for example, inside
  an error message from another service): treated as a defect; the spec requires records to
  be checked for this.
- A platform Bill-GN runs on offers records that include question text: those are left off,
  or kept out of the monitoring system.
- Very high volume, such as a runaway conversation: records still arrive, and the cost of
  recording stays proportionate because limits (feature 002) cap the volume.
- Identifiers of people: records identify who asked by their Slack identifier, not by name or
  email address.
- Clocks differ between components: steps of one request still appear in the right order.
- An operator without access to the cloud account: can do everything in this specification
  from the monitoring system alone.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Every component of Bill-GN MUST write structured records of what it does, and
  those records MUST reach the team's existing monitoring system through the environment's
  existing delivery path, with no manual setup per component.
- **FR-002**: Every record belonging to one question MUST carry the same correlation
  identifier, from the moment the question arrives to the posted answer, across every
  component it passes through.
- **FR-003**: An operator MUST be able to follow one question end to end in the monitoring
  system and see each step's outcome and duration.
- **FR-004**: Every failure MUST be recorded with the step that failed and the kind of
  failure. Requests retained as failed (feature 002) MUST be visible in the monitoring
  system.
- **FR-005**: Records, measurements and traces MUST NOT contain question text, answer text,
  remembered conversation content, reference material content, or secrets. Records MAY
  contain lengths, counts, identifiers, outcomes and durations.
- **FR-006**: Records MUST identify the person who asked only by their Slack identifier.
- **FR-007**: The following MUST be measured and available in the monitoring system:
  questions received; outcomes by kind (answer, decline, input rejection, failure); requests
  rejected as not genuine or not permitted; questions refused by each usage limit; time to
  answer, including the slowest 5%; and model usage with an estimate of its cost.
- **FR-008**: Each later feature MUST add its own measurements to the same place: memory
  reads and failures (003), reference material use (004), lookups by tool with outcome and
  duration (005), and specialist consultations with outcome and duration (006).
- **FR-009**: There MUST be one health view that shows the measurements in FR-007 for a chosen
  period, and that makes a problem in progress apparent without a search.
- **FR-010**: The health view and any alerts MUST be defined as code in the project and
  created by the project's normal deployment, not built by hand.
- **FR-011**: The conditions that raise an alert, where each alert goes, and how quickly, MUST
  be defined in one list approved by the project owner. A condition not on the list MUST NOT
  raise an alert. The list MAY be empty.
- **FR-012**: Changing the alert list or the health view MUST NOT require changing Bill-GN's
  behavior.
- **FR-013**: A failure of recording, measuring or tracing MUST NOT prevent Bill-GN from
  answering a question.
- **FR-014**: It MUST be possible to tell a period with no problems from a period with no
  data.
- **FR-015**: The documentation MUST say where to look, how to follow one question, what each
  measurement means, and what is deliberately not recorded.
- **FR-016**: Records MUST be kept for a defined period and then removed, following the
  monitoring system's existing retention settings unless the owner specifies otherwise.
- **FR-017**: Any platform-provided records that would include question text MUST be turned
  off or excluded from the monitoring system.

### Key Entities

- **Record**: One structured entry describing something a component did. Attributes: when,
  which component, the correlation identifier, the step, the outcome, the duration, and
  non-content details such as lengths and counts.
- **Correlation identifier**: The single identifier that ties together everything recorded
  about one question.
- **Trace**: The ordered set of steps one question passed through, with timings.
- **Measurement**: A number tracked over time, such as questions received or time to answer.
- **Health view**: The single place that shows the measurements together.
- **Alert condition**: A defined situation that notifies someone, with where the notice goes
  and how quickly.
- **Failed request**: A request retained by feature 002 because it could not be completed.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: For 100% of at least 20 test questions, including at least 5 forced to fail at
  different steps, an operator can find the question in the monitoring system from its
  correlation identifier and state its outcome and, for failures, the failing step.
- **SC-002**: An operator who has not seen the question before can do this in under 5 minutes
  using only the documentation.
- **SC-003**: 100% of the steps a question passes through appear in its trace, in order.
- **SC-004**: After a known pattern of at least 50 test questions, every count in the health
  view (received, each outcome, rejections, limit refusals) matches the known pattern exactly.
- **SC-005**: Records appear in the monitoring system within 5 minutes of the event for 95% of
  events.
- **SC-006**: A review of every record, measurement and trace produced during testing finds
  no question text, no answer text and no secrets.
- **SC-007**: With the path to the monitoring system made unavailable, 100% of questions are
  still answered.
- **SC-008**: Reported model usage for a known set of questions is within 5% of the usage the
  model provider reports for the same period.
- **SC-009**: Each alert condition on the approved list raises its alert within its specified
  time in 100% of trials, and no condition off the list raises one.
- **SC-010**: The health view and alerts can be recreated from the project's code alone in a
  fresh environment, with no manual steps beyond those documented as one-time setup.
- **SC-011**: When a new component is added by a later feature, its records reach the
  monitoring system with no change to the delivery path.

## Assumptions

- The team's existing monitoring system is Datadog, and the environment already delivers logs
  and measurements from the cloud account to it. This feature uses that path and does not
  build a new one. Whether that path picks up everything Bill-GN's platforms emit is the main
  open question for the clarify and plan phases.
- Features 001 and 002 already require a record per request with a correlation identifier,
  outcome and duration, and that failed requests are retained. This feature makes those
  records consistent, joined up across components, and visible in the monitoring system. It
  does not replace those requirements.
- The project owner decided in feature 002's clarification that failures are recorded and not
  alerted on. The alert list therefore starts empty or very short, and is the owner's to fill.
- Operators are the project owner and developers. There is no separate support team.
- Not recording question and answer text is deliberate and takes priority over ease of
  debugging. An operator diagnosing a bad answer asks the person for the question.
- Cost figures are estimates derived from usage. The cloud bill remains the authoritative
  figure.
- This feature can be delivered in stages alongside the others: the parts covering features
  001 and 002 first, with each later feature adding its own measurements when it is built.
- The order of work is undecided: observability may be more useful built early, before
  features 003 to 006, even though it is numbered last.
- Expected usage is low (a small team, tens of questions per day), so the volume and cost of
  records is small.
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
