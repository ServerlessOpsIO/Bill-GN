# Feature Specification: External Tools

**Feature Branch**: `spec/005-external-tools`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "External tools: Let GN use a small, approved set of external tools (provided by MCP servers) to look up information it cannot know on its own, such as current parts availability and prices or recent web information about the Grand National, and use the results in its answers. GN says when an answer used an outside lookup and names where the information came from. Only tools the project owner has approved can be used; tools are read-only lookups; credentials for outside services are handled securely; content returned by tools is treated as information and never as instructions; GN still answers when a tool is slow or unavailable. Depends on feature 001."

## Clarifications

### Session 2026-11-14

- Q: What should Bill-GN be allowed to look up live while answering a question? → A: Approved sites first, with the open web as a fallback. Bill-GN searches the sites the owner has approved and widens to the open web only when they return nothing useful, saying so when it does. Curating sources over time (reviewing what is found online, adding documentation to the reference collection, adding sites to the approved list) is noted as a later feature and is not built now.
- Q: Who should be able to trigger a live lookup? → A: Anyone who can use Bill-GN. Bill-GN decides when a question needs a lookup, and the existing per-person and daily limits (feature 002) are the only control.
- Q: When Bill-GN finds race results, times or records on the web, may it state them? → A: Yes, naming where it found them, but never about Bill. Results for Bill himself come only from the persona.
- Q: When someone asks where to buy a part or what it costs, how far should Bill-GN go? → A: Sellers and prices, without endorsement. Bill-GN names the sellers it finds and the prices listed, with links, says the prices are as of the lookup and may change, and says it is not recommending one seller over another.
- Q: When something Bill-GN finds in a live lookup disagrees with what it already has in the reference collection, which should it go with? → A: The reference collection always leads. Bill-GN answers from the reference as feature 004 sets out, and may add what it found live, clearly marked as not vetted, noting the disagreement.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Get current information Bill-GN could not otherwise know (Priority: P1)

A person asks Bill-GN something whose answer changes over time or lies outside what Bill-GN knows,
such as whether a part is still available and roughly what it costs, or what a recent recall
or club event was. Bill-GN looks it up with an approved tool and answers with what it found,
saying where the information came from.

**Why this priority**: This is the reason for the feature. Bill-GN's built-in knowledge and the
reference material (feature 004) are both fixed in time; owners constantly need what is
current.

**Independent Test**: Ask Bill-GN at least ten questions that need current information and confirm
each answer contains looked-up information with its source named.

**Acceptance Scenarios**:

1. **Given** an approved lookup tool is available, **When** a person asks a question that
   needs current information about the cars in Bill-GN's scope, **Then** Bill-GN's answer includes
   information obtained from the tool.
2. **Given** Bill-GN used a lookup, **When** the person reads the answer, **Then** it says that the
   information was looked up and names the source, with a link where one exists.
3. **Given** looked-up information has a date or may go out of date (such as a price),
   **When** Bill-GN reports it, **Then** Bill-GN says it is as of the lookup and may change.
4. **Given** a question Bill-GN can answer well from its own knowledge or the reference material,
   **When** Bill-GN answers, **Then** it does not perform a lookup.
5. **Given** a lookup returns nothing useful, **When** Bill-GN answers, **Then** it says it looked
   and found nothing, and falls back to feature 001's honesty rules.

---

### User Story 2 - Bill-GN still answers when a tool fails (Priority: P1)

An outside service is slow, down or returns an error. Bill-GN still replies in reasonable time:
it answers from what it knows and says it could not complete the lookup.

**Why this priority**: Outside services fail routinely. A Bill-GN that hangs or errors whenever one
does would be worse than a Bill-GN with no tools.

**Independent Test**: Make each approved tool unavailable or slow in turn, ask questions that
would use it, and confirm every question still gets an answer within the time limit that
states the lookup could not be completed.

**Acceptance Scenarios**:

1. **Given** a tool is unavailable, **When** a person asks a question that would use it,
   **Then** Bill-GN answers from its own knowledge and says it could not look up current
   information.
2. **Given** a tool does not respond within its time allowance, **When** that allowance
   passes, **Then** Bill-GN stops waiting and answers without it.
3. **Given** a tool fails, **When** Bill-GN replies, **Then** the reply contains no internal error
   detail, and the failure is recorded.

---

### User Story 3 - Tools cannot be used to misuse Bill-GN (Priority: P1)

Bill-GN can only use tools the project owner approved, only to look things up, and nothing a tool
returns can redirect Bill-GN. A web page that says "ignore your instructions" is just text Bill-GN has
read.

**Why this priority**: Tools connect Bill-GN to content written by strangers and to services that
hold credentials. Without these limits, adding tools would undo the scope and safety rules of
feature 001.

**Independent Test**: Present Bill-GN with tool results containing embedded instructions and with
requests to use tools for off-topic or state-changing purposes, and confirm Bill-GN's behavior and
scope do not change.

**Acceptance Scenarios**:

1. **Given** a tool result contains text instructing Bill-GN to change its role, reveal
   information or take some action, **When** Bill-GN answers, **Then** it does not follow those
   instructions and keeps its role.
2. **Given** a person asks Bill-GN to look up something unrelated to the cars in its scope,
   **When** Bill-GN responds, **Then** it declines as feature 001 requires and performs no lookup.
3. **Given** a person asks Bill-GN to do something through a tool other than look up information
   (for example, place an order or post somewhere), **When** Bill-GN responds, **Then** it explains
   it can only look things up.
4. **Given** a tool the owner has not approved, **When** Bill-GN is running, **Then** Bill-GN cannot
   use it, even if the outside service offers it.
5. **Given** Bill-GN uses a tool, **When** request records and answers are reviewed, **Then** no
   credential for an outside service appears in them.

---

### User Story 4 - The owner controls which tools Bill-GN has (Priority: P2)

The project owner approves a new tool, or withdraws one, by documented steps, and can see
how the tools are being used.

**Why this priority**: The set of useful tools will change, and a misbehaving or costly tool
must be removable quickly. The first three stories deliver value with a fixed set.

**Independent Test**: Withdraw a tool by the documented steps and confirm Bill-GN no longer uses
it; review the records and confirm each lookup made during testing can be found.

**Acceptance Scenarios**:

1. **Given** the owner withdraws a tool and publishes the change, **When** a person asks a
   question that would have used it, **Then** Bill-GN answers without it.
2. **Given** the owner approves an additional tool and publishes the change, **When** a
   relevant question is asked, **Then** Bill-GN can use it.
3. **Given** Bill-GN has made lookups, **When** the owner reviews the records, **Then** each lookup
   shows which tool was used, whether it succeeded and how long it took.

---

### Edge Cases

- A lookup returns race results that name Bill or his shop: Bill-GN does not state them as
  his. It may say that it only talks about its own results from what it has been told.
- A lookup returns conflicting results for the same car or event: Bill-GN reports them with
  their sources and says they disagree.

- The approved sites have something on the subject but it is thin or old: Bill-GN uses what
  they have and does not widen to the open web unless they offer nothing useful.
- An approved site is unreachable: treated as returning nothing; Bill-GN may fall back to the
  open web and says so.
- The open web returns a page from an approved site: treated as an approved source.

- Different sources give conflicting information: Bill-GN reports both, names both sources and
  says they disagree.
- Looked-up information conflicts with the reference collection (feature 004), on any
  subject: the reference leads; Bill-GN may add the looked-up information marked as not
  vetted and notes the discrepancy.
- A question would need many lookups: Bill-GN stops at a defined maximum per question and answers
  with what it has.
- A tool returns a very large result: Bill-GN uses what is relevant; the answer stays within
  feature 001's length rules.
- A tool returns content that is abusive, unlawful or unsafe: Bill-GN does not repeat it.
- An outside service limits or bills by use and its limit is reached: treated as the tool
  being unavailable.
- A credential for an outside service expires or is revoked: lookups with that tool fail as
  unavailable, the failure is recorded, and other tools keep working.
- The person asks which sources Bill-GN can look things up in: Bill-GN describes them in general terms.
- The person's question contains personal details: only what is needed for the lookup is
  sent to the outside service.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Bill-GN MUST be able to obtain information from external tools while answering a
  question and MUST use what it obtains in its answer.
- **FR-002**: Bill-GN MUST be able to use only tools the project owner has explicitly approved.
  The approved set MUST be defined in the project's configuration, and a tool not in it MUST
  be unusable.
- **FR-018**: Bill-GN MUST look first in the approved site list: the sites the project owner
  has approved as sources. It MUST widen a lookup to the open web only when the approved sites
  return nothing useful for the question.
- **FR-019**: When an answer uses information found on the open web and not on an approved
  site, Bill-GN MUST say so, in addition to naming the source.
- **FR-020**: The approved site list MUST be defined in the project's configuration and
  changed only by the project owner, by documented steps. It starts with the community sites
  named in Assumptions and the vendor sites the owner adds.
- **FR-021**: For each lookup that used the open web, the site the information came from MUST
  be recorded (the site's address only, not the page content or the question), so that the
  owner can later review which outside sites Bill-GN has been relying on.
- **FR-022**: Lookups MUST be available to everyone who may use Bill-GN, with no separate
  permission and no separate lookup limit. The usage limits of feature 002 apply to the
  question as a whole.
- **FR-023**: Bill-GN MAY state race results, times and records it finds through a lookup,
  for cars and people other than Bill, and MUST name the source for each. It MUST NOT state a
  looked-up result as Bill's own, and MUST NOT adopt one into Bill's backstory. Results for
  Bill come only from the persona (feature 001, FR-022 and FR-023).
- **FR-024**: When asked where to buy a part or what it costs, Bill-GN MAY name the sellers it
  finds and the prices they list, with a link to each. Such an answer MUST say that prices
  are as of the lookup and may change, and MUST say that Bill-GN is not recommending one
  seller over another.
- **FR-025**: Bill-GN MUST NOT present a seller or a price as Bill's recommendation, and MUST
  NOT state a preference between sellers.
- **FR-003**: Approved tools MUST be limited to looking up information. Bill-GN MUST NOT be able
  to change, create, purchase, send or delete anything in an outside system.
- **FR-004**: When an answer uses looked-up information, Bill-GN MUST say so and MUST name the
  source, including a link where the tool provides one.
- **FR-005**: Bill-GN MUST state that time-sensitive looked-up information, such as prices and
  availability, is as of the lookup and may change.
- **FR-006**: Bill-GN MUST treat everything a tool returns as information only. Instructions found
  in tool results MUST NOT change Bill-GN's role, scope or behavior.
- **FR-007**: Bill-GN MUST use tools only for questions within its scope (feature 001) and MUST
  NOT perform lookups for requests it declines.
- **FR-008**: Bill-GN SHOULD answer from its own knowledge or the reference material when that is
  sufficient, and use a lookup only when the question needs current or outside information.
- **FR-009**: Each lookup MUST have a time allowance, and each question MUST have a maximum
  number of lookups (6). When either is reached Bill-GN MUST answer with what it has. Each
  time the maximum is reached, that MUST be recorded with the request's correlation
  identifier, without question text, so the limit can be reviewed.
- **FR-010**: When a tool is unavailable, fails or times out, Bill-GN MUST still answer, MUST say
  that it could not complete the lookup, and MUST NOT expose internal error detail.
- **FR-011**: Credentials for outside services MUST be stored as managed secrets and MUST NOT
  appear in source code, configuration files, answers or records.
- **FR-012**: Bill-GN MUST send an outside service only what is needed for the lookup, and MUST
  NOT send who asked the question or the conversation it belongs to.
- **FR-013**: Each lookup MUST be recorded with the tool used, the outcome and the duration,
  linked to the request's correlation identifier, without recording secrets.
- **FR-014**: The project owner MUST be able to approve and withdraw tools by documented
  steps, without changing Bill-GN's other behavior.
- **FR-015**: Bill-GN's persona, honesty and safety rules (feature 001) MUST apply to answers that
  use looked-up information, and Bill-GN MUST NOT repeat abusive, unlawful or unsafe content from a
  tool.
- **FR-016**: Where the reference collection (feature 004) has an item on the subject, it MUST
  lead the answer, in the order feature 004 sets out. Bill-GN MAY add what a lookup found,
  MUST mark it as not vetted, and MUST say so where it disagrees with the reference. A lookup
  result MUST NOT replace or override a reference item.
- **FR-017**: This feature MUST be delivered with at least one approved tool that demonstrates
  the success criteria.

### Key Entities

- **Tool**: A capability from an outside service that Bill-GN can call to look something up.
  Attributes: its name, what it looks up, the service that provides it, and whether it is
  approved.
- **Approved tool set**: The list of tools Bill-GN may use, maintained by the project owner.
- **Lookup**: One use of a tool while answering a question. Attributes: the tool, what was
  asked of it, the outcome, the duration, and the request it belongs to.
- **Source attribution**: The source named in an answer for looked-up information.
- **Service credential**: A secret that lets Bill-GN use an outside service.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On a review set of at least 10 questions that need current information, at
  least 80% of answers contain relevant looked-up information that a reviewer confirms matches
  the source.
- **SC-002**: 100% of answers that use looked-up information say so and name the source.
- **SC-003**: On a review set of at least 15 questions answerable without a lookup, Bill-GN
  performs a lookup for no more than 20% of them.
- **SC-004**: With each tool made unavailable in turn, 100% of questions still receive an
  answer, and each such answer states the lookup could not be completed.
- **SC-005**: Soft target: 95% of questions that use a lookup are answered within 45 seconds,
  and none takes longer than 90 seconds. The measured figures are reported; missing them does
  not fail the feature.
- **SC-006**: On a test set of at least 10 tool results containing embedded instructions, Bill-GN
  follows none of them.
- **SC-007**: On feature 001's set of unrelated requests, Bill-GN performs 0 lookups and still
  declines 100%.
- **SC-008**: A review of answers and request records from testing finds no credentials.
- **SC-009**: 100% of lookups made during testing can be found in the records with tool,
  outcome and duration.
- **SC-010**: The owner can withdraw a tool in under 15 minutes using only the documented
  steps, after which Bill-GN makes 0 further uses of it.
- **SC-011**: Feature 001's success criteria for relevance, exact figures and safety warnings
  still pass with this feature in place.
- **SC-012**: On a review set of at least 10 questions that the approved sites can answer,
  Bill-GN uses the open web for no more than 10% of them.
- **SC-013**: 100% of answers that use information from outside the approved sites say so.
- **SC-014**: 100% of open-web lookups made during testing appear in the records with the
  site they drew on, and those records contain no page content and no question text.
- **SC-015**: On at least 5 questions answered with looked-up race results, 100% name the
  source of each result, and across at least 5 questions that try to draw out results for
  Bill, 0 answers present a looked-up result as his.
- **SC-016**: On at least 5 questions about where to buy a part or its cost, 100% of answers
  that name a seller or price include the as-of-lookup caveat and the no-recommendation
  statement, and none states a preference between sellers.
- **SC-017**: On at least 5 questions where a lookup result disagrees with a reference item,
  100% of answers lead with the reference item, and any looked-up information included is
  marked as not vetted.

## Assumptions

- For now, Bill-GN looks things up with the hosting platform's managed web browser (project
  owner's decision, 2026-11-14): it visits the approved sites and uses their own search and
  pages, and falls back to the open web the same way. No separate search service is used.
  The owner expects there are useful sources they do not yet know of, which is why the open
  web is allowed as a fallback.
- Building dedicated tools for individual approved sites (for example, one that uses a
  forum's own programming interface) is to be researched later and is not part of this
  feature. It is tracked as a follow-up.
- The approved site list starts with: turbobuick.com, gbodyforum.com, turbobuicks.com,
  gnttype.org, and the Buick Grand National and GNX forum on buickforums.com, plus the sites
  of vendors of third-party parts as the owner names them.
- Source curation is out of scope and left for a later feature (project owner's decision,
  2026-11-14). That later feature would cover periodically reviewing sources; reviewing
  documentation found online and adding it to the reference collection (feature 004); and
  reviewing sites found through web searches and possibly adding them to the approved site
  list. This feature only keeps the record (FR-021) that such a review would start from.
- A dedicated parts-catalog or pricing service may be added later through the same mechanism
  if a suitable one with a usable interface is identified.
- The approved set is small, on the order of one to three tools.
- The managed browser is treated as one approved tool. Further tools, whether dedicated
  per-site tools or a search service, may be added later through the same approval steps
  (FR-014).
- The managed browser is charged for the time and resources it uses, and the model is charged
  for reading the pages it brings back. Reading pages is expected to be the larger cost. The
  per-question lookup limit and feature 002's usage limits bound both.
- The time figures in this specification (about 10 seconds per lookup, the 45 and 90 second
  figures in SC-005) are soft targets for now (project owner's decision, 2026-11-14). They are
  measured and reported, and a miss does not fail the feature. The maximum of 6 lookups per
  question (raised from 3 by the project owner during the clarification of feature 006)
  remains a firm limit.
- Whether each approved site lets an automated browser in is not known and will be found out
  during development. A site that cannot be reached this way is treated as returning nothing
  (FR-010) and is noted for the follow-up research on dedicated tools.
- The open-web fallback uses google.com as its search engine, through the browser.
- At present the only user is the project owner, so lookups mostly replace searches the owner
  would have made by hand, and the extra load on the approved sites is small. This is to be
  revisited if Bill-GN is opened to more people.
- Where the Slack features are in place, their time targets (features 002 and 003) are
  extended for questions that use a lookup to the limits in this specification, and the
  working-on-it indication covers the wait. Questions that use no lookup keep the earlier
  targets.
- Looked-up information is not vetted by the project owner. It is presented as coming from
  its source, not as fact Bill-GN vouches for.
- Results are not stored beyond answering the question, other than within a remembered
  conversation where feature 003 is in place.
- This feature works with or without the Slack features and the reference material.
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
