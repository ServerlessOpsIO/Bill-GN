# Feature Specification: Slack Conversation

**Feature Branch**: `spec/002-slack-conversation`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "Slack conversation: A user sends GN a direct message in Slack, or @-mentions GN in a channel, and receives GN's answer (from the GN expert agent of feature 001) as a threaded reply. Covers: verifying that each incoming request genuinely comes from Slack before anything else happens; acknowledging Slack quickly so it does not retry; handling duplicate deliveries so a question is answered once; showing a 'working on it' indicator while GN prepares the answer; replying with a friendly message when GN cannot answer; ignoring messages from bots and from GN itself; failed work is not silently lost. Each message is treated as a standalone question in this feature (conversation memory is feature 003). Depends on feature 001."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Ask GN in a channel (Priority: P1)

A member of the Slack workspace @-mentions GN in a channel with a question about the Grand
National. GN shows that it is working on it and then posts its answer as a reply in a thread
under the person's message.

**Why this priority**: This is the product: GN answering where people already talk. Without
it, GN is reachable only by developers.

**Independent Test**: In a channel GN has been added to, @-mention GN with a Grand National
question and confirm one answer appears in a thread under that message.

**Acceptance Scenarios**:

1. **Given** GN is a member of a channel, **When** a person posts a top-level message that
   @-mentions GN with a question, **Then** GN posts one answer as a reply in a thread under
   that message.
2. **Given** a person @-mentions GN inside an existing thread, **When** GN answers, **Then**
   the answer is posted in that same thread.
3. **Given** a person @-mentions GN, **When** GN has received the question, **Then** the
   person sees an indication that GN is working on it within a few seconds, and that
   indication no longer shows once the answer is posted.
4. **Given** a message in a channel that does not @-mention GN, **When** it is posted,
   **Then** GN does not respond.
5. **Given** a message that @-mentions GN with other text around the mention, **When** GN
   reads the question, **Then** the mention itself is not treated as part of the question.

---

### User Story 2 - Ask GN in a direct message (Priority: P1)

A person sends GN a direct message with a question, without needing to @-mention it, and
receives GN's answer as an ordinary message in that direct message conversation, with no
thread.

**Why this priority**: Direct messages are the other half of how people expect to reach a
bot, and the request that started this project names both.

**Independent Test**: Open a direct message with GN, send a Grand National question with no
mention, and confirm one answer appears as the next message in the direct message, not in a
thread.

**Acceptance Scenarios**:

1. **Given** a person has a direct message conversation with GN, **When** they send a
   question, **Then** GN posts one answer as a message in the direct message conversation
   itself, not in a thread.
2. **Given** a person sends two separate questions in a direct message, **When** GN answers
   them, **Then** the answers appear in the conversation in the order the questions were
   sent, and each question is treated as standalone.
3. **Given** a person sends a direct message, **When** GN has received it, **Then** the person
   sees the same working-on-it indication as in a channel.
4. **Given** a group direct message that includes GN, **When** any member posts a question
   without mentioning GN, **Then** GN posts one answer as a message in that group
   conversation, not in a thread.

---

### User Story 3 - GN answers once and fails gracefully (Priority: P2)

Slack sometimes delivers the same message more than once, and sometimes GN cannot produce an
answer. In either case the person sees sensible behavior: one answer, or one friendly message
saying GN could not answer, and never silence or a duplicate.

**Why this priority**: Duplicate or missing replies make a bot look broken and erode trust
quickly, but the feature still demonstrates value before this is perfect.

**Independent Test**: Deliver the same message event twice and confirm one answer; make the
expert agent unavailable, ask a question, and confirm one friendly failure message appears
and the failed request can be found afterwards.

**Acceptance Scenarios**:

1. **Given** Slack delivers the same message event more than once, **When** GN processes
   them, **Then** the person receives exactly one answer.
2. **Given** GN cannot produce an answer, **When** the attempt fails, **Then** the person
   receives one friendly message, posted where the answer would have appeared, saying GN
   could not answer and inviting them to try again, with no internal detail.
3. **Given** a request fails in a way that prevents any reply, **When** it has been retried
   without success, **Then** the failed request is kept where an operator can find and inspect
   it, and it is not silently discarded.
4. **Given** GN declines a question as off-topic (feature 001), **When** it replies, **Then**
   the decline is posted where any other answer would appear.

---

### User Story 4 - Only genuine Slack requests are accepted (Priority: P1)

Requests that claim to come from Slack are checked for authenticity before anything else
happens. A forged or replayed request is rejected and never reaches GN.

**Why this priority**: The entry point is reachable from the public internet. Without this
check anyone could drive GN, run up cost, or post into the workspace.

**Independent Test**: Send requests with a missing, wrong and stale authenticity proof and
confirm each is rejected, produces no reply and is recorded.

**Acceptance Scenarios**:

1. **Given** a request whose authenticity proof is missing or does not match, **When** it
   arrives, **Then** it is rejected, no question is processed and no reply is posted.
2. **Given** a genuine request that is replayed after its validity window has passed, **When**
   it arrives, **Then** it is rejected.
3. **Given** a genuine request from Slack, **When** it arrives, **Then** Slack receives an
   acknowledgement within Slack's response deadline, before GN's answer is ready.
4. **Given** Slack's one-time address verification request made when the app is configured,
   **When** it arrives, **Then** it is answered correctly so the app can be connected.

---

### Edge Cases

- A message from another bot, or from GN itself: ignored, so GN never answers itself or loops
  with another bot.
- An edited or deleted message: ignored. Only newly posted messages are answered.
- A message with files or images but no text: GN replies that it can only read text.
- A message that is empty after the mention is removed: GN replies with a short prompt asking
  what the person wants to know (the behavior defined in feature 001).
- A question longer than GN's maximum: the person is told the question is too long (feature
  001).
- An answer longer than Slack allows in one message: it is split across consecutive messages
  in the same place (the thread in a channel, the conversation in a direct message), in
  order, with nothing lost.
- GN is @-mentioned in a channel it is not a member of: it cannot reply; the event is
  recorded and no error is shown to the person.
- An @-mention inside a direct message: answered once, not twice.
- A person replies inside a thread in a direct message: GN answers in the main direct message
  conversation, as for any other direct message.
- Two questions sent in a direct message before GN has answered the first: both are answered,
  in the order they were sent.
- A group direct message that includes GN and several people: treated like a one-to-one
  direct message. GN answers every message any member posts there, with no mention needed,
  and replies in the conversation without a thread.
- Many questions arriving at once: each is answered; none is dropped. Answers may take
  longer.
- GN takes much longer than usual: the working-on-it indication remains until an answer or a
  failure message is posted, and the person is never left with a permanent indicator and no
  reply.
- The workspace's access to GN is revoked or its credentials become invalid: requests fail,
  are recorded, and are kept for inspection.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: GN MUST respond when a person @-mentions it in a channel it is a member of.
- **FR-002**: GN MUST respond when a person sends it a direct message, with no mention
  required. This applies equally to group direct messages that include GN: GN MUST respond to
  every message posted there by a person.
- **FR-003**: In a channel, GN MUST post its response as a reply in a thread: under the
  person's message when that message is top-level, or in the existing thread when the message
  is already in one. In a direct message, GN MUST post its response as an ordinary message in
  the conversation and MUST NOT use a thread.
- **FR-004**: The response MUST be the answer produced by the GN expert agent (feature 001)
  for the text of the person's message with any mention of GN removed.
- **FR-005**: The authenticity of every incoming request MUST be verified before any other
  processing. Requests that fail verification, or that fall outside the allowed time window,
  MUST be rejected, MUST NOT produce a reply, and MUST be recorded.
- **FR-006**: Every genuine request MUST be acknowledged to Slack within Slack's response
  deadline, independent of how long GN takes to answer.
- **FR-007**: A message delivered more than once MUST result in exactly one response.
- **FR-008**: While GN prepares a response, the person MUST see an indication that GN is
  working, appearing within 5 seconds of their message and removed or replaced when the
  response is posted.
- **FR-009**: When GN cannot produce an answer, the person MUST receive one friendly failure
  message, posted where the answer would have appeared, containing no internal detail.
- **FR-010**: GN MUST ignore messages from bots, including itself, and MUST ignore message
  edits and deletions.
- **FR-011**: GN MUST NOT respond to channel messages that do not mention it.
- **FR-012**: A request that cannot be completed after retrying MUST be retained for
  inspection by an operator and MUST NOT be silently discarded.
- **FR-013**: Answers MUST be presented readably in Slack, including lists and emphasis, and
  answers that exceed Slack's message size MUST be split across ordered messages in the same
  place.
- **FR-014**: Each message MUST be treated as a standalone question in this feature; earlier
  messages in the thread or conversation are not used.
- **FR-015**: Each request MUST be traceable from its arrival to the posted reply by a single
  correlation identifier, with its outcome and duration recorded. Secrets and message text
  MUST NOT appear in these records.
- **FR-016**: Credentials that let GN act in Slack and that prove a request is genuine MUST be
  stored as managed secrets and MUST NOT appear in source code, configuration files or
  records.
- **FR-017**: The Slack integration MUST be granted access to the GN expert agent explicitly,
  as feature 001 requires of every caller.
- **FR-018**: The one-time steps to create and connect the Slack app MUST be documented so
  another person can repeat them.

### Key Entities

- **Slack message**: What a person posts. Attributes: the text, who posted it, the channel or
  direct message it is in, the thread it belongs to (if any), when it was posted, and Slack's
  unique identifier for the delivery.
- **Question**: The message text with the mention of GN removed, passed to the GN expert
  agent. Carries a correlation identifier.
- **Reply**: What GN posts back. Attributes: the text (possibly in several parts), where it
  is posted (a channel thread or a direct message conversation), its kind (answer, decline, input rejection, or failure message), and the
  same correlation identifier.
- **Working indicator**: The temporary sign that GN is preparing a reply, tied to one message.
- **Failed request**: A request that could not be completed after retries, kept with enough
  information to diagnose and replay it.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Across at least 20 test questions split between channel mentions and direct
  messages, 100% receive exactly one reply in the correct place: the thread for a channel
  mention, the conversation itself for a direct message.
- **SC-002**: 95% of questions receive their answer in Slack within 40 seconds of being
  posted.
- **SC-003**: For 95% of questions, the working-on-it indication appears within 5 seconds.
- **SC-004**: 100% of genuine requests are acknowledged within Slack's response deadline, so
  Slack reports no delivery failures during testing.
- **SC-005**: When the same message event is delivered three times, the person sees exactly
  one reply, in 100% of trials.
- **SC-006**: 100% of forged, altered or expired requests are rejected and produce no reply.
- **SC-007**: When the expert agent is made unavailable, 100% of questions receive a friendly
  failure message or, if no reply is possible, appear among the retained failed requests;
  none disappears.
- **SC-008**: GN posts no reply to any message from a bot, from itself, to an edit, or to a
  channel message without a mention, across a test set covering each case.
- **SC-009**: 100% of test requests can be traced from arrival to reply by correlation
  identifier, and a review of the records finds no secrets or message text.
- **SC-010**: A new person can create and connect the Slack app in a fresh workspace using
  only the documented steps, in under 30 minutes.
- **SC-011**: Over any 24-hour period with no messages, the Slack integration incurs no
  usage-based running cost.

## Assumptions

- GN is installed in a single Slack workspace owned by the project. Distribution to other
  workspaces is out of scope.
- Everyone in that workspace may use GN. Per-person or per-channel permissions are out of
  scope.
- Conversation memory is out of scope. A follow-up in a channel thread or in a direct message
  is answered as a standalone question until feature 003.
- Group direct messages behave like one-to-one direct messages (project owner's decision,
  2026-11-14): no mention is needed and no thread is used. Adding GN to a group direct message
  therefore means GN reads and answers everything posted there, including messages members
  address to each other.
- Direct messages do not use threads (project owner's decision, 2026-11-14): a direct message
  with GN reads as one continuous back-and-forth. Feature 003 treats the whole direct message
  as one conversation and each channel thread as one conversation.
- In channels GN responds only to messages that @-mention it, including follow-ups inside a
  thread it has already answered in.
- Slash commands, buttons, the app home tab, reactions as input, files and images are out of
  scope.
- The working-on-it indication uses whatever lightweight mechanism Slack offers (for example a
  reaction on the person's message or a status line); the exact form is chosen during
  planning.
- The 40-second target allows 30 seconds for the expert agent (feature 001, SC-005) plus
  delivery time.
- Slack's response deadline and message size limit are those published by Slack at the time
  of planning.
- Expected usage is low (a small team, tens of questions per day), with occasional bursts of
  a few questions at once.
- GN replies in English only, as in feature 001.
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
