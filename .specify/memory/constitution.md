# Slack AI Agent Constitution

## Core Principles

### I. Serverless and Managed First

All compute MUST run on managed, scale-to-zero AWS services: Amazon Bedrock AgentCore Runtime
for the agent, AWS Lambda for glue code, and Amazon API Gateway and Amazon SQS for ingress and
buffering. Always-on compute (EC2 instances, long-running containers, persistent socket
connections) MUST NOT be introduced.

**Rationale**: A proof of concept has to cost nothing when idle and require no servers to
patch or operate.

### II. Everything as Code

All infrastructure MUST be defined in the AWS SAM template in this repository. Deployments MUST
happen only through the GitHub Actions workflows. Resources MUST NOT be created or changed by
hand in the AWS console, except for one-time bootstrap steps that are documented in the README.

**Rationale**: The repository is the single source of truth, so any environment can be
reproduced and every change is reviewed.

### III. Secure by Default

- IAM roles MUST be least-privilege: actions and resources are scoped to what each component
  uses, and wildcard resources require a written justification in the template.
- Secrets (Slack tokens, signing secrets, API keys) MUST be stored in AWS Secrets Manager and
  MUST NOT appear in source code, template parameters, plain environment variables, or logs.
- Every inbound Slack request MUST have its signature verified before any other work is done on
  it. Requests that fail verification MUST be discarded.

**Rationale**: The bot exposes a public endpoint and an agent with tools; both are attack
surface from the first deploy.

### IV. Proof-of-Concept Simplicity

Designs MUST use the fewest components that satisfy the current specification. Capabilities
MUST NOT be added for hypothetical future needs (YAGNI). Any component beyond the minimum MUST
be justified in the plan's complexity tracking.

**Rationale**: The goal is to learn whether the approach works, and every extra part slows
that down.

### V. Observable

- All components MUST emit structured (JSON) logs that include a correlation identifier
  traceable from the Slack event to the agent invocation.
- Tracing MUST be enabled on Lambda functions and the agent runtime.
- Every asynchronous hand-off MUST have a dead-letter queue so failed events are visible and
  can be replayed.

**Rationale**: Agent behaviour is non-deterministic; failures cannot be diagnosed without a
record of what happened.

### VI. Tested

- Every Lambda handler and every piece of non-trivial agent code MUST have unit tests, written
  alongside the implementation. Strict test-first ordering is NOT required.
- The SAM template MUST pass `cfn-lint` before merge.
- Tests and linting MUST run in the branch workflow and MUST pass before merge to `main`.

**Rationale**: Enough testing to change the code with confidence, without the overhead of
full test-driven development on a proof of concept.

## Technology Constraints

- **Agent framework**: Strands Agents SDK (Python).
- **Agent hosting**: Amazon Bedrock AgentCore Runtime.
- **Models**: Anthropic Claude models, accessed through Amazon Bedrock.
- **Infrastructure**: AWS SAM / CloudFormation, deployed with GitHub Actions.
- **Interface**: Slack, through the Slack Events API and Web API.

Changing any of these is a constitutional amendment.

## Development Workflow

- Work follows the Spec Kit flow: specify, clarify, plan, tasks, implement.
- All changes are made on a feature branch and merged to `main` through a pull request.
- Each plan MUST include a Constitution Check that confirms compliance with every principle
  or records a justified exception.

## Governance

This constitution supersedes other project practices where they conflict. Amendments MUST be
made by pull request that changes this file, states the reason, and updates the version:

- **MAJOR**: a principle is removed or redefined in a backward-incompatible way.
- **MINOR**: a principle or section is added or materially expanded.
- **PATCH**: clarifications and wording changes.

Pull request reviews MUST verify compliance with these principles. Deviations MUST be justified
in the plan's complexity tracking or resolved before merge.

**Version**: 1.0.0 | **Ratified**: 2026-11-13 | **Last Amended**: 2026-11-13
