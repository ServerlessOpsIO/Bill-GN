# Feature Specification: Slack Conversation

**Feature Branch**: `spec/002-slack-conversation`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "Slack conversation: A user sends GN a direct message in Slack, or @-mentions GN in a channel, and receives GN's answer (from the GN expert agent of feature 001) as a threaded reply. Covers: verifying that each incoming request genuinely comes from Slack before anything else happens; acknowledging Slack quickly so it does not retry; handling duplicate deliveries so a question is answered once; showing a 'working on it' indicator while GN prepares the answer; replying with a friendly message when GN cannot answer; ignoring messages from bots and from GN itself; failed work is not silently lost. Each message is treated as a standalone question in this feature (conversation memory is feature 003). Depends on feature 001."

**Naming**: The bot's name is **Bill-GN** (project owner's decision, 2026-11-14). A person
starts a conversation from a channel by @-mentioning it: `@Bill-GN`. The original description
above, written before the name was chosen, calls it GN.

## Clarifications

### Session 2026-11-14

- Q: Who in the Slack workspace is allowed to use Bill-GN: full members only, or also guests and people from other organizations in shared channels? → A: Full members and guests. People from other organizations in shared channels are ignored.
- Q: When someone @-mentions Bill-GN in a channel, what should the people in that channel see? → A: Both a reaction on the person's message that everyone in the channel sees, and a note that only the asker sees, with a link to the direct message.
- Q: When someone @-mentions Bill-GN in a channel with a message that depends on what was said just before, should Bill-GN read the surrounding messages to understand the question? → A: No. Bill-GN reads only the message that mentions it. If the question is not clear on its own, it asks the person in their direct message to restate it.
- Q: Should there be a limit on how many questions one person can ask Bill-GN in a given period? → A: Yes, two limits: a per-person limit and an overall daily limit across everyone.
- Q: When Bill-GN fails to answer people, should it actively alert someone, or is it enough that failures are recorded for an operator to find? → A: Record only in this feature. Observability, including how failures reach the team's monitoring and whether anything alerts, becomes a separate feature (007).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Ask Bill-GN from a channel, get the answer in a direct message (Priority: P1)

A member of the Slack workspace @-mentions Bill-GN in a channel with a question about the Grand
National. Bill-GN does not answer in the channel. It adds a reaction to the person's message, which
everyone in the channel can see, shows the asker a note that only they can see with a link to
the direct message, and sends its answer to that person in their direct message with Bill-GN,
where the conversation can continue.

**Why this priority**: People think of a question while talking in a channel, so that is
where they will reach for Bill-GN. Answering in a direct message keeps every conversation with Bill-GN
in one place and keeps channels free of long replies.

**Independent Test**: In a channel Bill-GN has been added to, @-mention Bill-GN with a Grand National
question and confirm nothing visible to other members is posted in the channel, the message
carries Bill-GN's reaction, the asker sees the private note, and one answer arrives in the asker's direct message with Bill-GN.

**Acceptance Scenarios**:

1. **Given** Bill-GN is a member of a channel, **When** a person posts a message that @-mentions
   Bill-GN with a question, **Then** Bill-GN sends one answer to that person in their direct message
   with Bill-GN and posts no message in the channel that other members can see.
2. **Given** Bill-GN answers a channel mention in a direct message, **When** the person reads it,
   **Then** the direct message shows which question is being answered and links back to the
   message in the channel.
3. **Given** a person @-mentions Bill-GN in a channel, **When** Bill-GN has received the question,
   **Then** within a few seconds the person's message in the channel carries a reaction from
   Bill-GN that everyone in the channel can see, and the asker sees a note, visible only to
   them, saying Bill-GN is answering in their direct message and linking to it. The note does
   not claim the answer has already arrived.
4. **Given** a person @-mentions Bill-GN inside a thread in a channel, **When** Bill-GN answers,
   **Then** it behaves the same way: the answer goes to the person's direct message and
   nothing visible to other members is posted in the thread.
5. **Given** a message in a channel that does not @-mention Bill-GN, **When** it is posted,
   **Then** Bill-GN does not respond.
6. **Given** a message that @-mentions Bill-GN with other text around the mention, **When** Bill-GN
   reads the question, **Then** the mention itself is not treated as part of the question.
7. **Given** a person has asked Bill-GN from a channel, **When** they want to continue, **Then**
   they do so in the direct message, with no mention needed.

---

### User Story 2 - Ask Bill-GN in a direct message (Priority: P1)

A person sends Bill-GN a direct message with a question, without needing to @-mention it, and
receives Bill-GN's answer as an ordinary message in that direct message conversation, with no
thread.

**Why this priority**: Direct messages are the other half of how people expect to reach a
bot, and the request that started this project names both.

**Independent Test**: Open a direct message with Bill-GN, send a Grand National question with no
mention, and confirm one answer appears as the next message in the direct message, not in a
thread.

**Acceptance Scenarios**:

1. **Given** a person has a direct message conversation with Bill-GN, **When** they send a
   question, **Then** Bill-GN posts one answer as a message in the direct message conversation
   itself, not in a thread.
2. **Given** a person sends two separate questions in a direct message, **When** Bill-GN answers
   them, **Then** the answers appear in the conversation in the order the questions were
   sent, and each question is treated as standalone.
3. **Given** a person sends a direct message, **When** Bill-GN has received it, **Then** the person
   sees an indication that Bill-GN is working on it within a few seconds, and that indication no
   longer shows once the answer is posted.
4. **Given** a group direct message that includes Bill-GN, **When** any member posts a question
   without mentioning Bill-GN, **Then** Bill-GN posts one answer as a message in that group
   conversation, not in a thread.

---

### User Story 3 - Bill-GN answers once and fails gracefully (Priority: P2)

Slack sometimes delivers the same message more than once, and sometimes Bill-GN cannot produce an
answer. In either case the person sees sensible behavior: one answer, or one friendly message
saying Bill-GN could not answer, and never silence or a duplicate.

**Why this priority**: Duplicate or missing replies make a bot look broken and erode trust
quickly, but the feature still demonstrates value before this is perfect.

**Independent Test**: Deliver the same message event twice and confirm one answer; make the
expert agent unavailable, ask a question, and confirm one friendly failure message appears
and the failed request can be found afterwards.

**Acceptance Scenarios**:

1. **Given** Slack delivers the same message event more than once, **When** Bill-GN processes
   them, **Then** the person receives exactly one answer.
2. **Given** Bill-GN cannot produce an answer, **When** the attempt fails, **Then** the person
   receives one friendly message, posted where the answer would have appeared, saying Bill-GN
   could not answer and inviting them to try again, with no internal detail.
3. **Given** a request fails in a way that prevents any reply, **When** it has been retried
   without success, **Then** the failed request is kept where an operator can find and inspect
   it, and it is not silently discarded.
4. **Given** Bill-GN declines a question as off-topic (feature 001), **When** it replies, **Then**
   the decline is posted where any other answer would appear.

---

### User Story 4 - Only genuine Slack requests are accepted (Priority: P1)

Requests that claim to come from Slack are checked for authenticity before anything else
happens. A forged or replayed request is rejected and never reaches Bill-GN.

**Why this priority**: The entry point is reachable from the public internet. Without this
check anyone could drive Bill-GN, run up cost, or post into the workspace.

**Independent Test**: Send requests with a missing, wrong and stale authenticity proof and
confirm each is rejected, produces no reply and is recorded.

**Acceptance Scenarios**:

1. **Given** a request whose authenticity proof is missing or does not match, **When** it
   arrives, **Then** it is rejected, no question is processed and no reply is posted.
2. **Given** a genuine request that is replayed after its validity window has passed, **When**
   it arrives, **Then** it is rejected.
3. **Given** a genuine request from Slack, **When** it arrives, **Then** Slack receives an
   acknowledgement within Slack's response deadline, before Bill-GN's answer is ready.
4. **Given** Slack's one-time address verification request made when the app is configured,
   **When** it arrives, **Then** it is answered correctly so the app can be connected.

---

### Edge Cases

- A person reaches the per-person limit: Bill-GN tells them once, with the time they can ask
  again, and stays silent to their further questions until then.
- The overall daily limit is reached: each person who asks is told once that Bill-GN is done
  for the day.
- A busy group direct message: each message counts against the person who posted it, and
  against the daily limit.
- A limit is reached on a channel mention: the notice goes to the asker's direct message, with
  the usual reaction and private note in the channel.

- A channel mention that depends on earlier messages (for example, "@Bill-GN what do you
  think?" in a thread about a misfire): Bill-GN does not read the earlier messages. Its direct
  message reply says it only saw that one message and asks the person to restate the question
  in full.

- A person from another organization @-mentions Bill-GN in a shared channel: ignored, with no
  reply of any kind; the event is recorded.
- A guest @-mentions Bill-GN or sends it a direct message: answered like any member.

- A message from another bot, or from Bill-GN itself: ignored, so Bill-GN never answers itself or loops
  with another bot.
- An edited or deleted message: ignored. Only newly posted messages are answered.
- A message with files or images but no text: Bill-GN replies that it can only read text.
- A message that is empty after the mention is removed: Bill-GN replies with a short prompt asking
  what the person wants to know (the behavior defined in feature 001).
- A question longer than Bill-GN's maximum: the person is told the question is too long (feature
  001).
- An answer longer than Slack allows in one message: it is split across consecutive messages
  in the same direct message conversation, in order, with nothing lost.
- Bill-GN is @-mentioned in a channel it is not a member of: Bill-GN does not see the mention and
  cannot respond.
- Bill-GN cannot send a direct message to the person who mentioned it (for example, their account
  cannot receive one): no answer is posted in the channel; the failure is recorded and kept
  for inspection.
- Several people @-mention Bill-GN in the same channel: each receives their own answer in their own
  direct message.
- A person @-mentions Bill-GN from a channel while Bill-GN is still answering their earlier question:
  both are answered in their direct message, in the order they were asked.
- An @-mention inside a direct message: answered once, not twice.
- A person replies inside a thread in a direct message: Bill-GN answers in the main direct message
  conversation, as for any other direct message.
- Two questions sent in a direct message before Bill-GN has answered the first: both are answered,
  in the order they were sent.
- A group direct message that includes Bill-GN and several people: treated like a one-to-one
  direct message. Bill-GN answers every message any member posts there, with no mention needed,
  and replies in the conversation without a thread.
- Many questions arriving at once: each is answered; none is dropped. Answers may take
  longer.
- Bill-GN takes much longer than usual: the working-on-it indication remains until an answer or a
  failure message is posted, and the person is never left with a permanent indicator and no
  reply.
- The workspace's access to Bill-GN is revoked or its credentials become invalid: requests fail,
  are recorded, and are kept for inspection.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Bill-GN MUST respond when a person @-mentions it in a channel it is a member of,
  whether in a top-level message or in a thread. The response MUST be sent to that person in
  their direct message with Bill-GN, and Bill-GN MUST NOT post a message in the channel that
  other members can see. The asker's private note (FR-019) is the only thing Bill-GN shows in
  the channel besides its reaction.
- **FR-002**: Bill-GN MUST respond when a person sends it a direct message, with no mention
  required. This applies equally to group direct messages that include Bill-GN: Bill-GN MUST respond to
  every message posted there by a person.
- **FR-003**: Bill-GN MUST post its response as an ordinary message in the direct message
  conversation and MUST NOT use a thread. A response to a channel mention MUST show which
  question it answers and MUST link back to the person's message in the channel.
- **FR-019**: When Bill-GN responds to a channel mention, it MUST do both of the following in
  the channel, and nothing else: add a reaction to the person's message, visible to all
  channel members; and show the asker a note, visible only to them, saying Bill-GN is
  answering in their direct message and linking to it. Both MUST appear within 5 seconds of
  the mention, before the answer is ready, so the note MUST NOT say the answer has arrived. Bill-GN MUST NOT post a message that other channel
  members can see.
- **FR-004**: The response MUST be the answer produced by the Bill-GN expert agent (feature 001)
  for the text of the person's message with any mention of Bill-GN removed. Two kinds of
  reply are exceptions and are produced by this feature without the expert agent: the request
  to restate a channel question (FR-022) and the usage limit notices (FR-023, FR-024).
- **FR-005**: The authenticity of every incoming request MUST be verified before any other
  processing. Requests that fail verification, or that fall outside the allowed time window,
  MUST be rejected, MUST NOT produce a reply, and MUST be recorded.
- **FR-006**: Every genuine request MUST be acknowledged to Slack within Slack's response
  deadline, independent of how long Bill-GN takes to answer.
- **FR-007**: A message delivered more than once MUST result in exactly one response.
- **FR-008**: While Bill-GN prepares a response, the person MUST see an indication that Bill-GN is
  working, appearing within 5 seconds of their message and removed or replaced when the
  response is posted.
- **FR-009**: When Bill-GN cannot produce an answer, the person MUST receive one friendly failure
  message, posted where the answer would have appeared, containing no internal detail.
- **FR-010**: Bill-GN MUST ignore messages from bots, including itself, and MUST ignore message
  edits and deletions.
- **FR-021**: Bill-GN MUST answer full members and guests of the workspace. It MUST ignore
  messages from people who belong to another organization (for example, in a channel shared
  with that organization): no answer, no direct message, no reaction and no note.
- **FR-022**: For a channel mention, Bill-GN MUST use only the text of the message that
  mentions it. It MUST NOT read other messages in the channel or thread. When that text
  refers to something outside itself that it does not contain (for example, "what do you
  think?" or "is that right?"), its direct message reply MUST ask the person to
  restate the full question there.
- **FR-023**: Bill-GN MUST enforce a per-person limit on questions (default: 30 per person per
  hour). The hour is a rolling window: a person may ask when fewer than 30 of their questions
  fall within the previous 60 minutes, and the notice gives the time at which that will next
  be true. When a person exceeds it, Bill-GN MUST tell them once that they have reached the
  limit and when they can ask again, and MUST NOT answer their further questions until then.
- **FR-024**: Bill-GN MUST enforce an overall daily limit on questions across all people
  (default: 300 per day). The day is the calendar day in UTC, and the count resets at
  00:00 UTC, which is the time the notice gives. Once it is reached, Bill-GN MUST tell each person who asks, once,
  that it has reached its limit for the day and when it will be available again.
- **FR-025**: Questions refused because of a limit MUST NOT be sent to the expert agent, and
  each refusal MUST be recorded. Both limits MUST be adjustable by the project owner without
  changing Bill-GN's other behavior.
- **FR-011**: Bill-GN MUST NOT respond to channel messages that do not mention it.
- **FR-012**: A request that cannot be completed after retrying MUST be retained for
  inspection by an operator and MUST NOT be silently discarded.
- **FR-013**: Answers MUST be presented readably in Slack, including lists and emphasis, and
  answers that exceed Slack's message size MUST be split across ordered messages in the same
  place.
- **FR-014**: Each message MUST be treated as a standalone question in this feature; earlier
  messages in the conversation are not used.
- **FR-015**: Each request MUST be traceable from its arrival to the posted reply by a single
  correlation identifier, with its outcome and duration recorded. Secrets and message text
  MUST NOT appear in these records.
- **FR-016**: Credentials that let Bill-GN act in Slack and that prove a request is genuine MUST be
  stored as managed secrets and MUST NOT appear in source code, configuration files or
  records.
- **FR-017**: The Slack integration MUST be granted access to the Bill-GN expert agent explicitly,
  as feature 001 requires of every caller.
- **FR-020**: The bot MUST appear in Slack under the name `Bill-GN`, so that people start a
  conversation from a channel by typing `@Bill-GN` and find its direct message under that
  name.
- **FR-018**: The one-time steps to create and connect the Slack app MUST be documented so
  another person can repeat them.

### Key Entities

- **Slack message**: What a person posts. Attributes: the text, who posted it, the channel or
  direct message it is in, when it was posted, and Slack's
  unique identifier for the delivery.
- **Question**: The message text with the mention of Bill-GN removed, passed to the Bill-GN expert
  agent. Carries a correlation identifier.
- **Reply**: What Bill-GN posts back. Attributes: the text (possibly in several parts), the direct message
  conversation it is posted in, the channel message it answers (if any), its kind (answer, decline, input rejection, or failure message), and the
  same correlation identifier.
- **Working indicator**: The temporary sign that Bill-GN is preparing a reply, tied to one message.
- **Failed request**: A request that could not be completed after retries, kept with enough
  information to diagnose and replay it.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Across at least 20 test questions split between channel mentions and direct
  messages, 100% receive exactly one reply in the correct direct message conversation, and Bill-GN
  posts 0 messages that other members can see in any channel.
- **SC-012**: For 100% of channel mentions in that set, the person's message carries Bill-GN's
  reaction, the asker sees the private note with a working link to the direct message, other
  channel members see no note, and the reply links back to the channel message.
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
- **SC-008**: Bill-GN posts no reply to any message from a bot, from itself, to an edit, or to a
  channel message without a mention, across a test set covering each case.
- **SC-013**: Across a test set with at least 3 messages each from a full member, a guest and
  a person from another organization, 100% of the member and guest messages are answered and
  0 of the external person's messages receive any response.
- **SC-014**: For at least 5 channel mentions that depend on earlier messages, 100% of replies
  ask the person to restate the question and none uses content from the earlier messages.
- **SC-015**: With the per-person limit set to a test value, the first question past the limit
  gets the limit notice, further questions get no reply, and questions are answered again
  once the period ends, in 100% of trials. The same holds for the daily limit.
- **SC-016**: 0 questions refused by a limit reach the expert agent.
- **SC-009**: 100% of test requests can be traced from arrival to reply by correlation
  identifier, and a review of the records finds no secrets or message text.
- **SC-010**: A new person can create and connect the Slack app in a fresh workspace using
  only the documented steps, in under 30 minutes.
- **SC-011**: Over any 24-hour period with no messages, the Slack integration incurs no
  usage-based running cost.

## Assumptions

- Bill-GN is installed in a single Slack workspace owned by the project. Distribution to other
  workspaces is out of scope.
- Full members and guests of the workspace may use Bill-GN; people from other organizations
  may not (clarified 2026-11-14). Finer per-person or per-channel permissions are out of
  scope.
- Conversation memory is out of scope. A follow-up in a direct message is answered as a
  standalone question until feature 003.
- Bill-GN holds conversations only in direct messages (project owner's decision, 2026-11-14). A
  channel mention is a way to ask; the answer and any follow-up happen in the asker's direct
  message. Each person has exactly one direct message with Bill-GN, so answers to all their
  channel mentions arrive in that one conversation.
- Answers to channel mentions are therefore private to the asker. Other channel members see
  only the reaction on the message, not the answer or the asker's note.
- Which emoji the reaction uses is chosen during planning. In a channel the reaction may also
  serve as the working-on-it indication.
- Group direct messages behave like one-to-one direct messages (project owner's decision,
  2026-11-14): no mention is needed and no thread is used. Adding Bill-GN to a group direct message
  therefore means Bill-GN reads and answers everything posted there, including messages members
  address to each other.
- Direct messages do not use threads (project owner's decision, 2026-11-14): a direct message
  with Bill-GN reads as one continuous back-and-forth. Feature 003 treats the whole direct message
  as one conversation.
- In channels Bill-GN responds only to messages that @-mention it.
- Slash commands, buttons, the app home tab, reactions as input, files and images are out of
  scope.
- The working-on-it indication uses whatever lightweight mechanism Slack offers (for example a
  reaction on the person's message or a status line); the exact form is chosen during
  planning.
- The 40-second target allows 30 seconds for the expert agent (feature 001, SC-005) plus
  delivery time.
- Slack's response deadline and message size limit are those published by Slack at the time
  of planning.
- The daily limit uses UTC (project owner's decision, 2026-11-14).
- The limit defaults (30 questions per person per hour, 300 per day overall) are starting
  values set well above expected use; they exist to cap runaway cost, not to ration normal
  use.
- This feature records failures and retains failed requests (FR-012, FR-015) but does not
  alert anyone. Monitoring, metrics and alerting are the subject of a separate observability
  feature (007); until it exists, a failure is noticed when someone looks or a person
  complains.
- Expected usage is low (a small team, tens of questions per day), with occasional bursts of
  a few questions at once.
- Bill-GN replies in English only, as in feature 001.
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
