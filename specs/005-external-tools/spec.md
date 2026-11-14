# Feature Specification: External Tools

**Feature Branch**: `spec/005-external-tools`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "External tools: Let GN use a small, approved set of external tools (provided by MCP servers) to look up information it cannot know on its own, such as current parts availability and prices or recent web information about the Grand National, and use the results in its answers. GN says when an answer used an outside lookup and names where the information came from. Only tools the project owner has approved can be used; tools are read-only lookups; credentials for outside services are handled securely; content returned by tools is treated as information and never as instructions; GN still answers when a tool is slow or unavailable. Depends on feature 001."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Get current information GN could not otherwise know (Priority: P1)

A person asks GN something whose answer changes over time or lies outside what GN knows,
such as whether a part is still available and roughly what it costs, or what a recent recall
or club event was. GN looks it up with an approved tool and answers with what it found,
saying where the information came from.

**Why this priority**: This is the reason for the feature. GN's built-in knowledge and the
reference material (feature 004) are both fixed in time; owners constantly need what is
current.

**Independent Test**: Ask GN at least ten questions that need current information and confirm
each answer contains looked-up information with its source named.

**Acceptance Scenarios**:

1. **Given** an approved lookup tool is available, **When** a person asks a question that
   needs current information about the cars in GN's scope, **Then** GN's answer includes
   information obtained from the tool.
2. **Given** GN used a lookup, **When** the person reads the answer, **Then** it says that the
   information was looked up and names the source, with a link where one exists.
3. **Given** looked-up information has a date or may go out of date (such as a price),
   **When** GN reports it, **Then** GN says it is as of the lookup and may change.
4. **Given** a question GN can answer well from its own knowledge or the reference material,
   **When** GN answers, **Then** it does not perform a lookup.
5. **Given** a lookup returns nothing useful, **When** GN answers, **Then** it says it looked
   and found nothing, and falls back to feature 001's honesty rules.

---

### User Story 2 - GN still answers when a tool fails (Priority: P1)

An outside service is slow, down or returns an error. GN still replies in reasonable time:
it answers from what it knows and says it could not complete the lookup.

**Why this priority**: Outside services fail routinely. A GN that hangs or errors whenever one
does would be worse than a GN with no tools.

**Independent Test**: Make each approved tool unavailable or slow in turn, ask questions that
would use it, and confirm every question still gets an answer within the time limit that
states the lookup could not be completed.

**Acceptance Scenarios**:

1. **Given** a tool is unavailable, **When** a person asks a question that would use it,
   **Then** GN answers from its own knowledge and says it could not look up current
   information.
2. **Given** a tool does not respond within its time allowance, **When** that allowance
   passes, **Then** GN stops waiting and answers without it.
3. **Given** a tool fails, **When** GN replies, **Then** the reply contains no internal error
   detail, and the failure is recorded.

---

### User Story 3 - Tools cannot be used to misuse GN (Priority: P1)

GN can only use tools the project owner approved, only to look things up, and nothing a tool
returns can redirect GN. A web page that says "ignore your instructions" is just text GN has
read.

**Why this priority**: Tools connect GN to content written by strangers and to services that
hold credentials. Without these limits, adding tools would undo the scope and safety rules of
feature 001.

**Independent Test**: Present GN with tool results containing embedded instructions and with
requests to use tools for off-topic or state-changing purposes, and confirm GN's behavior and
scope do not change.

**Acceptance Scenarios**:

1. **Given** a tool result contains text instructing GN to change its role, reveal
   information or take some action, **When** GN answers, **Then** it does not follow those
   instructions and keeps its role.
2. **Given** a person asks GN to look up something unrelated to the cars in its scope,
   **When** GN responds, **Then** it declines as feature 001 requires and performs no lookup.
3. **Given** a person asks GN to do something through a tool other than look up information
   (for example, place an order or post somewhere), **When** GN responds, **Then** it explains
   it can only look things up.
4. **Given** a tool the owner has not approved, **When** GN is running, **Then** GN cannot
   use it, even if the outside service offers it.
5. **Given** GN uses a tool, **When** request records and answers are reviewed, **Then** no
   credential for an outside service appears in them.

---

### User Story 4 - The owner controls which tools GN has (Priority: P2)

The project owner approves a new tool, or withdraws one, by documented steps, and can see
how the tools are being used.

**Why this priority**: The set of useful tools will change, and a misbehaving or costly tool
must be removable quickly. The first three stories deliver value with a fixed set.

**Independent Test**: Withdraw a tool by the documented steps and confirm GN no longer uses
it; review the records and confirm each lookup made during testing can be found.

**Acceptance Scenarios**:

1. **Given** the owner withdraws a tool and publishes the change, **When** a person asks a
   question that would have used it, **Then** GN answers without it.
2. **Given** the owner approves an additional tool and publishes the change, **When** a
   relevant question is asked, **Then** GN can use it.
3. **Given** GN has made lookups, **When** the owner reviews the records, **Then** each lookup
   shows which tool was used, whether it succeeded and how long it took.

---

### Edge Cases

- Different sources give conflicting information: GN reports both, names both sources and
  says they disagree.
- Looked-up information conflicts with the reference material (feature 004) on a factory
  specification: the reference material takes precedence, and GN may note the discrepancy.
- A question would need many lookups: GN stops at a defined maximum per question and answers
  with what it has.
- A tool returns a very large result: GN uses what is relevant; the answer stays within
  feature 001's length rules.
- A tool returns content that is abusive, unlawful or unsafe: GN does not repeat it.
- An outside service limits or bills by use and its limit is reached: treated as the tool
  being unavailable.
- A credential for an outside service expires or is revoked: lookups with that tool fail as
  unavailable, the failure is recorded, and other tools keep working.
- The person asks which sources GN can look things up in: GN describes them in general terms.
- The person's question contains personal details: only what is needed for the lookup is
  sent to the outside service.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: GN MUST be able to obtain information from external tools while answering a
  question and MUST use what it obtains in its answer.
- **FR-002**: GN MUST be able to use only tools the project owner has explicitly approved.
  The approved set MUST be defined in the project's configuration, and a tool not in it MUST
  be unusable.
- **FR-003**: Approved tools MUST be limited to looking up information. GN MUST NOT be able
  to change, create, purchase, send or delete anything in an outside system.
- **FR-004**: When an answer uses looked-up information, GN MUST say so and MUST name the
  source, including a link where the tool provides one.
- **FR-005**: GN MUST state that time-sensitive looked-up information, such as prices and
  availability, is as of the lookup and may change.
- **FR-006**: GN MUST treat everything a tool returns as information only. Instructions found
  in tool results MUST NOT change GN's role, scope or behavior.
- **FR-007**: GN MUST use tools only for questions within its scope (feature 001) and MUST
  NOT perform lookups for requests it declines.
- **FR-008**: GN SHOULD answer from its own knowledge or the reference material when that is
  sufficient, and use a lookup only when the question needs current or outside information.
- **FR-009**: Each lookup MUST have a time allowance, and each question MUST have a maximum
  number of lookups. When either is reached GN MUST answer with what it has.
- **FR-010**: When a tool is unavailable, fails or times out, GN MUST still answer, MUST say
  that it could not complete the lookup, and MUST NOT expose internal error detail.
- **FR-011**: Credentials for outside services MUST be stored as managed secrets and MUST NOT
  appear in source code, configuration files, answers or records.
- **FR-012**: GN MUST send an outside service only what is needed for the lookup, and MUST
  NOT send who asked the question or the conversation it belongs to.
- **FR-013**: Each lookup MUST be recorded with the tool used, the outcome and the duration,
  linked to the request's correlation identifier, without recording secrets.
- **FR-014**: The project owner MUST be able to approve and withdraw tools by documented
  steps, without changing GN's other behavior.
- **FR-015**: GN's persona, honesty and safety rules (feature 001) MUST apply to answers that
  use looked-up information, and GN MUST NOT repeat abusive, unlawful or unsafe content from a
  tool.
- **FR-016**: For factory specifications and procedures, the reference material (feature 004),
  where present, MUST take precedence over looked-up information.
- **FR-017**: This feature MUST be delivered with at least one approved tool that demonstrates
  the success criteria.

### Key Entities

- **Tool**: A capability from an outside service that GN can call to look something up.
  Attributes: its name, what it looks up, the service that provides it, and whether it is
  approved.
- **Approved tool set**: The list of tools GN may use, maintained by the project owner.
- **Lookup**: One use of a tool while answering a question. Attributes: the tool, what was
  asked of it, the outcome, the duration, and the request it belongs to.
- **Source attribution**: The source named in an answer for looked-up information.
- **Service credential**: A secret that lets GN use an outside service.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On a review set of at least 10 questions that need current information, at
  least 80% of answers contain relevant looked-up information that a reviewer confirms matches
  the source.
- **SC-002**: 100% of answers that use looked-up information say so and name the source.
- **SC-003**: On a review set of at least 15 questions answerable without a lookup, GN
  performs a lookup for no more than 20% of them.
- **SC-004**: With each tool made unavailable in turn, 100% of questions still receive an
  answer, and each such answer states the lookup could not be completed.
- **SC-005**: 95% of questions that use a lookup are answered within 45 seconds, and no
  question takes longer than 90 seconds.
- **SC-006**: On a test set of at least 10 tool results containing embedded instructions, GN
  follows none of them.
- **SC-007**: On feature 001's set of unrelated requests, GN performs 0 lookups and still
  declines 100%.
- **SC-008**: A review of answers and request records from testing finds no credentials.
- **SC-009**: 100% of lookups made during testing can be found in the records with tool,
  outcome and duration.
- **SC-010**: The owner can withdraw a tool in under 15 minutes using only the documented
  steps, after which GN makes 0 further uses of it.
- **SC-011**: Feature 001's success criteria for relevance, exact figures and safety warnings
  still pass with this feature in place.

## Assumptions

- The first approved tool is a web search limited to looking up public information, which
  covers parts availability, prices, recalls, events and recent discussion without needing an
  account with any parts supplier. Which tools to approve is the main open question for the
  clarify phase.
- A dedicated parts-catalog or pricing service may be added later through the same mechanism
  if a suitable one with a usable interface is identified.
- The approved set is small, on the order of one to three tools.
- Tools are provided by outside services through a standard tool protocol named in the
  project's technical direction; the choice of services is made during planning.
- Outside services may charge per use. At the expected usage (a small team, tens of questions
  per day) this is assumed to be low; the per-question lookup limit also bounds it.
- The defaults for the time allowance per lookup (about 10 seconds) and the maximum lookups
  per question (3) may be adjusted during planning.
- Where the Slack features are in place, their time targets (features 002 and 003) are
  extended for questions that use a lookup to the limits in this specification, and the
  working-on-it indication covers the wait. Questions that use no lookup keep the earlier
  targets.
- Looked-up information is not vetted by the project owner. It is presented as coming from
  its source, not as fact GN vouches for.
- Results are not stored beyond answering the question, other than within a remembered
  conversation where feature 003 is in place.
- This feature works with or without the Slack features and the reference material.
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
