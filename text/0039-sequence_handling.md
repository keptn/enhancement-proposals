# Advanced Task/Sequence Handling in Control-plane

**Success Criteria:** Improved UX for Keptn control-plane (for delivery/remediation orchestration)

## Motivation

* *Target*: Keptn provides basic (but “must have”) capabilities to control sequence/task execution by the user and smart timeouts.
* *Pain*: Currently, Keptn triggers sequences/tasks without providing capabilities to stop them afterwards. The Bridge still shows the sequence as running and the user is lost and cannot stop it.
* *Driver*: Other delivery/remedation solutions allow stopping/pausing/queuing pipelines. To stay competitive, this basic functionality must be supported.

## User Stories: 
- As a user, I can trigger multiple sequences and Keptn queues the sequences to avoid parallel executions in the same stage.
- Timeout - As a user, I would expect that Keptn terminates a triggered task when no service has reacted after X minutes.
- As a user, I can abort/pause a triggered sequence via the Bridge / via CLI.
- (lower prio) As a user, I can trigger any sequence (not only delivery/evaluation) using the CLI. 

Nice to have: (Bridge) As a user, I would like to see all available sequences in the Keptn Bridge.

## Open questions

## Future possibilities

