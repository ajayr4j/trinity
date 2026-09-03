---
name: real-submission-call
description: Use at the end of every task, right before finishing. Prevents describing or narrating a submission instead of actually making the required tool call.
---

# Real Submission Call

The task is only complete when the actual required call has been made and its response observed — never when the response text merely says a call would be made.

## Rule

1. The final answer must be delivered via a real tool invocation (e.g. `curl -X POST http://localhost:8000/submit_agent_response -d '{"assistant": "..."}'` executed through the Bash/shell tool, or the equivalent native tool call), not written out as prose that describes what would be sent.
2. After making the call, check the actual response (e.g. `{"status": "accepted", "task_finished": true}`). If the call errored, was malformed, or the response doesn't confirm acceptance, fix and resend — do not treat a written-but-unsent response as done.
3. This applies even under time/session pressure near the end of a long task — a rushed but real submission is correct; a well-formatted but never-sent one scores zero regardless of content quality.
