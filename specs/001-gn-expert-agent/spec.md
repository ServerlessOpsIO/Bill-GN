# Feature Specification: GN Expert Agent

**Feature Branch**: `001-gn-expert-agent`

**Created**: 2026-11-13

**Status**: Draft

**Input**: User description: "GN expert agent. Build the core AI agent behind a Slack bot named 'GN'. The agent answers questions as an expert on the 1987 Buick Grand National. It accepts a text prompt and returns a text answer. It stays in persona and on topic, is honest about uncertainty, and flags safety-critical work. This feature is the foundation and is proven by invoking the agent directly, without Slack; Slack integration, conversation memory, reference-knowledge skills, external tools, and subagent delegation are separate later features."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Ask GN a Grand National question (Priority: P1)

Someone with a question about the 1987 Buick Grand National sends it to GN and receives a
knowledgeable, practical answer written the way an experienced owner and mechanic would
explain it.

**Why this priority**: This is the whole reason GN exists. Without a useful expert answer,
nothing else in the project has value.

**Independent Test**: Send GN a set of representative Grand National questions (history,
specifications, maintenance, troubleshooting, modifications, buying advice) and confirm each
answer is relevant, accurate in substance, and usable by an owner.

**Acceptance Scenarios**:

1. **Given** GN is available, **When** a caller asks a factual question about the car (for
   example, what engine it has and how much power it made), **Then** GN returns a direct answer
   to that question in plain language.
2. **Given** GN is available, **When** a caller describes a symptom (for example, the car
   hesitates under boost), **Then** GN returns likely causes in a sensible order to check,
   with what to look for at each step.
3. **Given** GN is available, **When** a caller asks for advice (for example, what to inspect
   before buying one), **Then** GN returns specific, prioritized guidance and not generic
   used-car advice.
4. **Given** GN is available, **When** a caller asks about a closely related car of the same
   family and era (for example, the GNX or a Regal T-Type), **Then** GN answers and makes clear
   where that car differs from the 1987 Grand National.
5. **Given** a caller asks one question and then sends a second question that refers back to
   the first (for example, "and what about the rear?"), **When** GN answers the second,
   **Then** it treats it as a standalone question and does not use the earlier exchange.
6. **Given** a caller who has not been granted access, **When** that caller sends a question,
   **Then** the request is rejected as not authorized and no answer is produced.

---

### User Story 2 - GN stays in its lane (Priority: P2)

Someone asks GN about something that has nothing to do with the Grand National. GN politely
declines to act as a general assistant and steers the person back to what it can help with.

**Why this priority**: The persona is the product. A bot that answers anything is not a Grand
National expert, and unrelated use adds cost and risk without serving the purpose.

**Independent Test**: Send GN a set of unrelated requests (general knowledge, coding, other
vehicles, attempts to change its role) and confirm it declines each one courteously and offers
Grand National help instead.

**Acceptance Scenarios**:

1. **Given** GN is available, **When** a caller asks an unrelated question, **Then** GN
   briefly says this is outside what it covers and invites a Grand National question.
2. **Given** GN is available, **When** a caller asks a general automotive question that applies
   directly to owning or working on a Grand National, **Then** GN answers it in the context of
   the Grand National.
3. **Given** GN is available, **When** a caller instructs GN to ignore its role or behave as a
   different assistant, **Then** GN keeps its role and responds as it would to any other
   off-topic request.

---

### User Story 3 - GN is honest about what it knows (Priority: P2)

Someone asks GN for an exact figure or about a job where a mistake could cause injury or
damage. GN distinguishes what it is confident about from what should be verified, and it
calls out safety risks.

**Why this priority**: Owners may act on GN's answers with tools in hand. A confident wrong
torque value or a missing safety warning does real harm, and it destroys trust in the bot.

**Independent Test**: Send GN questions that ask for exact values (torque specifications,
part numbers, clearances, fluid capacities) and questions about hazardous work (fuel system,
brakes, lifting the car, electrical work, raising boost), and confirm the answers flag
uncertainty and risk appropriately.

**Acceptance Scenarios**:

1. **Given** GN is available, **When** a caller asks for an exact value GN is not certain of,
   **Then** GN says it is not certain, gives its best understanding labeled as such, and
   recommends confirming against the factory service manual or another authoritative source.
2. **Given** GN is available, **When** a caller asks about work with a risk of injury, fire,
   or serious damage, **Then** GN's answer includes a clear warning about that risk and the
   precautions to take.
3. **Given** GN is available, **When** a caller asks something GN does not know, **Then** GN
   says so plainly and does not invent an answer.
4. **Given** GN is available, **When** a caller asks about a modification that risks engine
   damage (for example, raising boost without supporting changes), **Then** GN explains the
   risk and what is needed to do it responsibly.

---

### Edge Cases

- An empty or whitespace-only question: GN returns a short prompt asking what the caller wants
  to know, and does not fail.
- A very long question: GN accepts questions up to a defined maximum length; beyond it, the
  caller receives a clear message that the question is too long.
- A question that mixes on-topic and off-topic parts: GN answers the Grand National part and
  declines the rest.
- An ambiguous question (for example, "what oil?"): GN answers the most likely meaning and
  states the assumption it made. Because each request stands alone, GN does not ask a
  clarifying question that would depend on a follow-up.
- A question about a different model year of Grand National (1982, or 1984 to 1986): GN
  answers and notes relevant differences from 1987.
- A question in a language other than English: GN replies in English.
- GN cannot produce an answer because of an internal failure: the caller receives a clear
  error indication, never a blank or misleading answer.
- A request for abusive or unlawful content, even when phrased around the car (for example,
  how to defeat an emissions inspection or alter a vehicle identification number): GN
  declines. This is distinct from hazardous but legitimate work, which GN answers with a
  warning.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: GN MUST accept a single text question from a caller and return a single text
  answer.
- **FR-002**: GN MUST answer as an expert on the 1987 Buick Grand National, covering at least:
  history and production, specifications, the turbocharged V6 drivetrain, maintenance,
  troubleshooting and common faults, parts, modifications, and buying and ownership advice.
- **FR-003**: GN MUST also answer questions about closely related cars of the same family and
  era (other Grand National model years, the GNX, and turbocharged Regal variants) and MUST
  make clear where they differ from the 1987 Grand National.
- **FR-004**: GN MUST decline requests unrelated to these cars, briefly and courteously, and
  MUST offer to help with a Grand National question instead.
- **FR-005**: GN MUST keep its role when a caller instructs it to abandon or change that role.
- **FR-006**: GN MUST NOT present an exact figure (such as a torque specification, part
  number, clearance, or capacity) as certain when it is not; it MUST label such figures as
  needing verification and point to an authoritative source.
- **FR-007**: GN MUST state plainly when it does not know something and MUST NOT fabricate an
  answer.
- **FR-008**: GN MUST include a clear warning and relevant precautions when a question
  involves work that risks injury, fire, or serious mechanical damage.
- **FR-009**: GN MUST put the direct answer to the question in the first two sentences of its
  reply, followed by supporting detail, and MUST keep a reply to 300 words or fewer unless the
  question asks for a detailed procedure or explanation. Replies MUST be in English.
- **FR-010**: GN MUST handle empty, over-length, and ambiguous questions as described in Edge
  Cases, and MUST enforce a maximum question length.
- **FR-011**: GN MUST return a clear error indication to the caller when it cannot produce an
  answer.
- **FR-012**: GN MUST be invocable only by callers the project owner has explicitly granted
  access. A request from any other caller MUST be rejected as not authorized, MUST NOT
  produce an answer, and MUST be recorded.
- **FR-013**: Each request MUST be recorded with a correlation identifier, outcome, and
  duration so that an individual request can be traced and failures diagnosed. Secrets MUST
  NOT appear in these records.
- **FR-016**: GN MUST decline requests for abusive or unlawful content, including when they
  are phrased around the car.
- **FR-014**: GN MUST be invocable directly, independent of Slack, so that this feature can be
  verified on its own and reused by later features.
- **FR-015**: Each request MUST be treated independently in this feature; GN does not carry
  context from one request to the next.

### Key Entities

- **Question**: The text a caller sends to GN. Attributes: the text, who sent it (the calling
  component or developer), when, and a correlation identifier.
- **Answer**: The text GN returns for a question. Attributes: the text, its kind, and the same
  correlation identifier. The kinds are: a normal answer, a decline (off-topic or disallowed
  content), an input rejection (empty or over-length question), and an error.
- **Persona**: The definition of who GN is: its subject expertise, scope boundaries, voice, and
  rules for honesty and safety. It governs every answer.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On a review set of at least 30 representative Grand National questions, at
  least 90% of answers are judged relevant and substantively correct by a knowledgeable
  reviewer.
- **SC-002**: On a review set of at least 15 unrelated or role-changing requests, GN declines
  100% of them and never answers as a general assistant.
- **SC-003**: On a review set of at least 10 questions asking for exact figures, 100% of
  answers either give a figure the reviewer confirms is correct or label it as needing
  verification; none states a wrong figure as certain.
- **SC-004**: On a review set of at least 10 questions about hazardous work, 100% of answers
  include an appropriate warning.
- **SC-005**: Across the SC-001 review set, 95% of questions receive a complete answer within
  30 seconds, measured from the caller sending the question to the caller receiving the
  answer.
- **SC-006**: A developer can send GN a question and read its answer directly, without Slack,
  in under 5 minutes using the documented steps.
- **SC-007**: 100% of requests in the review sets can be found in the request records by
  correlation identifier, each with its outcome and duration, and a review of those records
  finds no secrets.
- **SC-008**: Over any 24-hour period in which no questions are asked, GN incurs no
  usage-based running cost.
- **SC-009**: 100% of requests sent by a caller without access are rejected and produce no
  answer.
- **SC-010**: Across the SC-001 review set, at least 90% of replies give the direct answer in
  the first two sentences and stay within the length limit.

## Assumptions

- Callers of this feature are developers testing GN directly and, later, the Slack
  integration. End users reach GN only through later features.
- GN answers from general model knowledge in this feature. Vetted reference material, external
  tools, and specialist subagents are added by later features and are expected to improve
  accuracy on exact figures.
- Conversation memory across questions is out of scope; each question stands alone.
- Slack-specific behavior (message formatting, threading, progress indicators) is out of
  scope.
- GN replies in English only; other languages are out of scope for the proof of concept.
- In this feature the callers granted access are the project's developers and the deployment
  pipeline. The Slack integration is granted access by its own feature.
- The maximum question length defaults to 4,000 characters unless planning finds a reason to
  change it.
- Answers are plain text. Light structure such as short lists is acceptable; rich formatting
  is deferred to the Slack feature.
- The review sets used for the success criteria are written and judged by the project owner,
  who has first-hand knowledge of the car.
- Expected usage is low (a small team, tens of questions per day), consistent with a proof of
  concept.
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
