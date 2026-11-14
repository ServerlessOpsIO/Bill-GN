# Feature Specification: Thread Memory

**Feature Branch**: `spec/003-thread-memory`

**Created**: 2026-11-14

**Status**: Draft

**Input**: User description: "Thread memory: Follow-up messages in the same Slack thread (in a channel or a direct message) continue the same conversation with GN, so a person can ask 'and what about the rear?' and GN understands it in light of what was said earlier in that thread, including after a long pause. Each thread is its own conversation; different threads and different people's threads never share context. Covers how long a conversation is remembered, what happens when it gets very long, and removing remembered conversations. Depends on feature 002 (Slack conversation) and changes feature 001's 'each request stands alone' behavior for Slack threads only."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Ask a follow-up in the same conversation (Priority: P1)

A person asks GN a question, reads the answer, and then asks a follow-up in the same
conversation (the same thread in a channel, or the same direct message) that only makes sense in light of the first exchange. GN answers the follow-up correctly
without the person repeating themselves.

**Why this priority**: Real troubleshooting is a back-and-forth. Without memory every message
must restate the whole problem, which is the main limitation of features 001 and 002.

**Independent Test**: In one channel thread, and separately in a direct message, ask "What
brakes does the car have at the front?" and then "and what about the rear?"; confirm the
second answer is about rear brakes in both.

**Acceptance Scenarios**:

1. **Given** GN has answered a question in a channel thread, **When** the person asks a
   follow-up in that thread that refers back to it, **Then** GN's answer reflects the earlier
   exchange.
2. **Given** a conversation with several exchanges, **When** the person refers to something from
   the first exchange, **Then** GN still takes it into account.
3. **Given** GN answered a question assuming one meaning, **When** the person corrects it in
   the same conversation (for example, "no, I meant the transmission"), **Then** GN answers again
   using the correction.
4. **Given** GN has answered a question in a direct message, **When** the person sends a
   follow-up as their next message in that direct message, **Then** GN's answer reflects the
   earlier exchange, with no thread involved.
5. **Given** a channel thread GN has answered in, **When** a different person @-mentions GN
   with a follow-up in that thread, **Then** GN treats it as part of the same conversation.

---

### User Story 2 - Come back to a conversation later (Priority: P2)

A person returns to a conversation hours or days later, for example after trying what GN suggested
in the garage, and continues where they left off.

**Why this priority**: Working on a car happens between messages. Memory that lasts only
minutes would fail the most common real use.

**Independent Test**: Hold an exchange with GN, wait longer than GN's idle period, ask a
follow-up in the same conversation, and confirm GN uses the earlier exchange.

**Acceptance Scenarios**:

1. **Given** an exchange in a conversation, **When** the person asks a follow-up up to 30 days
   after the last message in that conversation, **Then** GN's answer reflects the earlier
   exchange.
2. **Given** a conversation whose last message is older than the retention period, **When** the
   person asks a follow-up, **Then** GN answers it as a new conversation and says it no longer
   has the earlier discussion.
3. **Given** a conversation that has grown very long, **When** the person asks another
   question, **Then** GN still answers, keeps the most recent exchanges and the main points
   of earlier ones, and does not fail or slow noticeably.

---

### User Story 3 - Conversations stay separate and can be removed (Priority: P2)

What is said in one conversation never shows up in another, and a remembered conversation can
be removed on request.

**Why this priority**: Leaking one person's conversation into another's is a privacy failure,
and a team needs a way to clear what has been stored.

**Independent Test**: Hold conversations on different subjects in two channel threads and in
a direct message, then ask an ambiguous follow-up in each and confirm each answer draws only
on its own conversation; remove one conversation and confirm a follow-up there is treated as
new.

**Acceptance Scenarios**:

1. **Given** two channel threads with different conversations, **When** a follow-up is asked
   in one, **Then** the answer uses only that thread's conversation.
2. **Given** the same person has a channel thread and a direct message with GN, **When** they
   ask a follow-up in one, **Then** nothing from the other is used.
3. **Given** two people each have a direct message with GN, **When** one asks a follow-up,
   **Then** nothing from the other person's direct message is used.
4. **Given** a remembered conversation, **When** an operator removes it, **Then** a later
   follow-up in that thread or direct message is answered as a new conversation.
5. **Given** a conversation past its retention period, **When** that period ends, **Then**
   the remembered conversation is no longer used and is deleted without anyone acting.

---

### Edge Cases

- A new top-level @-mention in a channel: starts a new conversation with no memory of other
  conversations.
- A new message in a direct message: continues that direct message's one conversation, however
  different its subject. GN uses earlier exchanges only where they are relevant.
- A person wants to start afresh in a direct message: they say so, and GN answers the new
  question on its own terms; the earlier exchanges stay remembered until they expire or an
  operator removes them.
- Two follow-ups sent in quick succession in one conversation: both are answered, in the order they
  were sent, and the second takes the first into account.
- Messages in a channel thread from people that do not mention GN: not seen by GN
  and not part of the remembered conversation.
- A message in the conversation is edited or deleted in Slack after GN answered: the remembered
  conversation is not changed.
- The remembered conversation cannot be read or saved because of a failure: GN answers the
  message as a standalone question and tells the person it could not recall the earlier
  discussion; the failure is recorded.
- A follow-up that is off-topic in a conversation that was on-topic: GN declines it as it
  would any off-topic request, and the conversation continues afterwards.
- An earlier attempt to change GN's role, remembered in the conversation: has no lasting effect on
  later answers.
- GN invoked directly rather than through Slack (feature 001's direct use): unchanged, each
  request stands alone.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: GN MUST treat all messages addressed to it within one conversation as a single
  exchange and MUST use the earlier exchanges of that conversation when answering a new
  message in it.
- **FR-002**: In a channel, a conversation MUST be one thread. In a direct message, the
  conversation MUST be the whole direct message between that person and GN, with no thread
  required. Different conversations MUST NOT share any context, including a channel thread
  and a direct message belonging to the same person.
- **FR-003**: A conversation MUST remain available for follow-ups for 30 days after the last
  message in it, regardless of how long the pause between messages is within that period.
- **FR-004**: A remembered conversation MUST stop being used as soon as 30 days have passed
  since its last message, and MUST be deleted from storage automatically within 24 hours
  after that.
- **FR-005**: When a follow-up arrives after the conversation has been removed, GN MUST answer
  it as a new conversation and MUST tell the person it no longer has the earlier discussion.
- **FR-006**: When a conversation grows too long to use in full, GN MUST keep the most recent
  exchanges in full and the main points of earlier ones, and MUST continue to answer.
- **FR-007**: An operator MUST be able to remove a specific remembered conversation on
  request, following documented steps.
- **FR-008**: Messages in one conversation MUST be handled in the order they were sent.
- **FR-009**: If the remembered conversation cannot be read or saved, GN MUST still answer
  the message as a standalone question, MUST tell the person it could not recall the earlier
  discussion, and the failure MUST be recorded.
- **FR-010**: A remembered conversation MUST contain only the person's messages addressed to
  GN and GN's replies in that conversation.
- **FR-011**: Remembered conversations MUST be stored so that only GN and authorized operators
  can read them, and MUST be protected at rest.
- **FR-012**: GN's persona, scope, honesty and safety rules (feature 001) MUST apply to every
  answer regardless of what earlier messages in the conversation said.
- **FR-013**: Request records MUST identify the conversation a request belongs to without
  recording message text.
- **FR-014**: Direct use of GN without Slack MUST keep working as in feature 001, with every
  request independent.
- **FR-015**: The documentation MUST state what GN remembers, for how long, and how to have
  it removed.

### Key Entities

- **Conversation**: The remembered exchange for one channel thread or one direct message.
  Attributes: the thread or direct message it belongs to, its ordered turns, when it was last active, and when it expires.
- **Turn**: One message addressed to GN and GN's reply to it, in order within a conversation.
- **Conversation summary**: The main points of older turns, kept in place of their full text
  once a conversation is long.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On a review set of at least 15 two-message exchanges where the second message
  depends on the first, at least 90% of second answers are judged by a reviewer to correctly
  use the first exchange.
- **SC-002**: On at least 5 pairs of different conversations (channel threads and direct
  messages), 0 answers contain information that appeared only in the other conversation.
- **SC-003**: A follow-up sent 24 hours after the previous message is answered using the
  earlier exchange in 100% of at least 5 trials.
- **SC-004**: In a conversation of 50 turns, GN still answers, and 95% of answers arrive
  within the time limit set in feature 002.
- **SC-005**: After an operator removes a conversation, 100% of follow-ups in that thread or
  direct message are answered as new conversations.
- **SC-006**: On at least 3 test conversations with a shortened retention period, 100% of
  follow-ups sent after the period ends are answered as new conversations, and each
  conversation is found deleted from storage within 24 hours after the period ends.
- **SC-007**: When memory is made unavailable, 100% of messages still receive an answer that
  states the earlier discussion could not be recalled.
- **SC-008**: A review of request records finds conversation identifiers and no message text.
- **SC-009**: Adding memory adds no more than 5 seconds to the time taken to answer for 95%
  of questions, compared with feature 002.

## Assumptions

- In a channel a conversation is a thread. In a direct message the conversation is the whole
  direct message, because direct messages do not use threads (project owner's decision,
  2026-11-14, see feature 002).
- A direct message conversation therefore has no natural end. It lasts until 30 days pass
  with no message, and long conversations are handled by keeping recent exchanges and the main
  points of older ones (FR-006). A command for a person to reset it is out of scope.
- GN does not remember anything across conversations: no per-person profile, no memory of the
  person's car, and no learning from past conversations. That would be a separate feature.
- The retention period defaults to 30 days after the last message, chosen to cover a job
  that spans a few weekends while limiting how long messages are kept.
- GN remembers only what was addressed to it and what it replied. It does not read other
  messages in a channel thread.
- Any person who @-mentions GN in a thread joins that thread's conversation; conversations in
  channels are shared by the thread's participants, as the thread itself is.
- Removal on request is an operator task for the proof of concept. A way for people to clear
  a conversation themselves from Slack is out of scope.
- Slack remains the visible record of the conversation; removing GN's memory does not delete Slack
  messages.
- Expected usage is low (a small team, tens of questions per day), with conversations
  typically under 20 turns.
- Technology choices are fixed by the project constitution and are addressed in the plan, not
  in this specification.
