# Feature Specification: Observability

**Feature Branch**: `spec/007-observability`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "Observability: the team can see whether Bill-GN is working, find out quickly when it is not, and diagnose a single failed question, using the team's existing monitoring (Datadog, fed from AWS CloudWatch by the environment's existing setup). Covers: all of Bill-GN's components writing structured logs that reach the monitoring system without per-component setup; failures (including the failed requests feature 002 retains) being visible there; a defined set of metrics for usage, errors, speed, limits reached and cost; traces that follow one question from Slack to the answer; a dashboard; deciding which conditions alert someone; and never putting question or answer text or secrets into logs, metrics or traces. Applies across features 001 to 006."

## Clarifications

### Session 2026-11-14

- Q: How should an operator follow one question from start to finish? → A: As a trace in Datadog. A path is added that delivers traces to Datadog, so one question appears there as a single timeline. New infrastructure and extra monitoring cost are accepted.
- Q: Should this feature include alerts at all? → A: A minimal set. Alerting is kept for a short fixed list: Bill-GN failing most requests over a sustained period, and usage approaching the daily limit. Everything else is record only.
- Q: How should the dashboard and alerts get created in Datadog? → A: Built directly in Datadog, with an exported copy kept in the repository so they can be recreated. For future agents the owner intends to create alarms and dashboards through the Backstage scaffolder when a new component is created; that is not part of this feature.
- Q: What should happen to the per-request records that say who asked? → A: Keep the Slack identifier in records. It is not a name or email address, and it lets an operator find a person's requests.
- Q: Which deployments of Bill-GN should the health view and the alerts cover? → A: The `main` deployment by default, with other branches selectable from a filter in the health view. Alerts fire for `main` only.

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

**Why this priority**: The owner's position is that recording is enough for ordinary
failures. Two conditions are the exception, because looking would catch them too late:
Bill-GN failing most requests, and usage approaching the daily limit. The alert set is
defined in one place and easy to change.

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

- A branch deployment is failing every request: no alert is raised; the failures are visible
  when the health view is switched to that branch, and in its records and traces.
- A branch deployment is deleted: its past data stays searchable for the retention period,
  and it drops out of the health view's filter once it has no recent data.

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
- A platform Bill-GN runs on offers records that include question text: that recording is
  turned off, or the text is removed before storage (FR-017).
- Very high volume, such as a runaway conversation: the usage limits (feature 002) cap the
  work sent to the model, but every incoming message still produces a record, including each
  refusal. Recording volume and cost therefore follow incoming traffic, not answered
  questions, and this must be visible in the health view.
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
- **FR-003**: One question MUST appear in the monitoring system as a single trace: a timeline
  of every step it passed through, across every component, with each step's outcome and
  duration. An operator MUST be able to open that trace from the question's correlation
  identifier, and to move between a trace and the records that belong to it.
- **FR-018**: Traces MUST be delivered to the monitoring system. Where the environment's
  existing delivery path does not carry traces, this feature adds one, as code.
- **FR-004**: Every failure MUST be recorded with the step that failed and the kind of
  failure. Requests retained as failed (feature 002) MUST be visible in the monitoring
  system.
- **FR-005**: Records, measurements and traces MUST NOT contain question text, answer text,
  remembered conversation content, reference material content, or secrets. Records MAY
  contain lengths, counts, identifiers, outcomes and durations.
- **FR-006**: Records of a question that came through Slack MUST identify the person who asked
  only by their Slack identifier. Records of a direct, non-Slack request (feature 001) MUST
  identify the caller by a non-content identifier of the calling component or developer.
  Names and email addresses MUST NOT be recorded in either case.
- **FR-007**: The following MUST be measured and available in the monitoring system:
  questions received; outcomes by kind (answer, decline, input rejection, failure); requests
  rejected as not genuine or not permitted; questions refused by each usage limit; time to
  answer, including the slowest 5%; and model usage with an estimate of its cost.
- **FR-008**: Each later feature's activity MUST be measured in the same place:
  - feature 002: limit notices and refusals by limit (per person, daily), and messages
    ignored because the sender is from another organization;
  - feature 003: memory reads and failures, conversations summarized, requests to forget, and
    deliberate reads of conversation content by an operator;
  - feature 004: use of the reference collection, by kind of source (factory, Bill, community,
    vendor), and questions the reference did not cover;
  - feature 005: lookups by outcome and duration, how many used the open web and which sites
    they drew on (address only), and each time the lookup limit was reached;
  - feature 006: consultations by specialist with outcome and duration, each time the lookup
    limit was reached and who was refused, and shadow comparison verdicts (better, same,
    worse) and comparison failures.
- **FR-009**: There MUST be one health view that shows the measurements in FR-007 for a chosen
  period, and that makes a problem in progress apparent without a search.
- **FR-010**: The health view and the alerts MUST be built directly in the monitoring system,
  and an exported copy of each definition MUST be kept in the project so that it can be
  recreated. Whenever one is changed in the monitoring system, the exported copy MUST be
  updated in the same piece of work.
- **FR-019**: The health view and alerts MUST select Bill-GN's data using the organization's
  standard resource tags, not by naming individual resources, so that they keep working when
  resources are replaced and can be filtered by branch or stack.
- **FR-020**: The health view MUST show the `main` deployment by default and MUST let an
  operator switch it to any other deployed branch. Alerts MUST fire only for the `main`
  deployment. Records and traces from every deployment MUST remain searchable.
- **FR-011**: There MUST be exactly two alert conditions to begin with, and a condition not on
  this list MUST NOT raise an alert:
  - Bill-GN is failing most requests: more than half of the questions received fail over a
    60-minute period in which at least 5 questions were received.
  - Usage is approaching the daily limit: the number of questions in the current day reaches
    80% of the overall daily limit (feature 002).
  Each alert MUST go to the project owner, within 10 minutes of the condition becoming true.
  The thresholds, and the list itself, MUST be changeable by the project owner.
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
- **FR-017**: Question text, answer text and conversation content MUST NOT be stored in any
  record, by Bill-GN or by a platform it runs on. Where a platform offers records that
  include such text, that recording MUST be turned off, or the text removed before the record
  is stored. Keeping such records out of the monitoring system while they are stored
  elsewhere does not satisfy this requirement.

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
- **SC-010**: The health view and each alert can be recreated in the monitoring system from
  the exported copies in the project alone, and a comparison of each recreated item with the
  original finds no difference.
- **SC-011**: When a new component is added by a later feature, its records reach the
  monitoring system with no change to the delivery path.
- **SC-012**: With a branch deployment made to fail every request, 0 alerts are raised; with
  the health view switched to that branch, its failures are shown; and with the view on its
  default, none of the branch's requests are counted.

## Assumptions

- The team's existing monitoring system is Datadog, and the environment already delivers
  logs and measurements from the cloud account to it. This feature uses that path and does
  not build a new one.
- What the existing path already does, from reading the environment's own repositories
  (`aws-datadog-integration` and `aws-datadog-shipping-infra`) on 2026-11-14, not from
  testing:
  - Logs: every log group in every member account is forwarded to Datadog by an account-level
    subscription. Anything a Bill-GN component or its platform writes to a log group is
    therefore delivered with no per-component setup. It also means any text that reaches a
    log group reaches Datadog, which is why FR-017 matters.
  - Measurements: all of the cloud provider's measurement namespaces are streamed to Datadog
    except two that are excluded, so the hosting platform's own measurements for the agent
    should arrive without extra work, provided the account is linked for metric streaming.
  - Traces: the existing path does not appear to deliver traces to Datadog. This feature
    adds that delivery (FR-018, project owner's decision, 2026-11-14). It may need changes to
    the environment's shared monitoring setup outside this project, and is likely to add to
    the monitoring bill; both are to be established in planning.
- Features 001 and 002 already require a record per request with a correlation identifier,
  outcome and duration, and that failed requests are retained. This feature makes those
  records consistent, joined up across components, and visible in the monitoring system. It
  does not replace those requirements.
- The project owner decided in feature 002's clarification that failures are recorded and not
  alerted on, and confirmed here (2026-11-14) a minimal set of two alerts for the cases that
  looking cannot catch in time. Individual failures still raise no alert.
- Alerts are tuned to fire readily at first and tightened over time (project owner's
  decision, 2026-11-14): receiving an alert that turns out to be unnecessary is preferred to
  missing one. The 60-minute window was chosen so that the alert can trigger at low traffic.
- The thresholds in FR-011 (more than half failing over 60 minutes with at least 5 questions;
  80% of the daily limit; notice within 10 minutes) are starting values. How the alert
  reaches the owner (for example email or a Slack message) is chosen in planning.
- The health view and alerts are built directly in Datadog (project owner's decision,
  2026-11-14), by the implementer using the write access the owner has granted, and exported
  to the repository. This replaces the earlier requirement that they be created by
  deployment, so the deployment pipeline needs no monitoring credentials.
- For future agents, the owner intends alarms and dashboards to be created by the Backstage
  scaffolder when a new component is created. That is outside this feature; the exported
  definitions here may serve as a starting point for it.
- The organization's standard tags are those in the project's `cfn-tags.json` (`org:domain`,
  `org:system`, `org:component`) and those the deployment action adds (`org:repo`,
  `org:branch`, `org:stackName`). In Datadog the colon becomes an underscore, for example
  `org_component`. Whether every kind of data Bill-GN produces (logs, measurements, traces)
  arrives carrying these tags is to be confirmed in planning; where it does not, the tag
  values are added to the records themselves.
- Operators are the project owner and developers. There is no separate support team.
- Not recording question and answer text is deliberate and takes priority over ease of
  debugging. An operator diagnosing a bad answer asks the person for the question.
- Cost figures are estimates derived from usage. The cloud bill remains the authoritative
  figure.
- This feature is built last, after features 001 to 006 (project owner's decision,
  2026-11-14). The owner accepts that this may mean reworking code already written for those
  features and amending their specifications.
- Until it is built, features 001 to 006 rely on the per-request records their own
  specifications require, with no health view, no joined-up trace and no alerts.
- Expected usage is low (a small team, tens of questions per day), so the volume and cost of
  records is small.
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
