# Retry failed pool pump schedule transitions

The pool pump should follow its daily on/off schedule even when a command to its cloud-connected plug fails, and admins should hear about it when it still doesn't. On 2026-09-15 one failed command left the pump off all day.

## Context

- The pump is turned on at 10:00 and off at 16:00 each day by two entries in the Scheduler custom component.
- Its smart plug is controlled through the Tuya cloud, and commands occasionally fail with cloud errors.
- On 2026-09-15 the 10:00 turn-on failed with a Tuya cloud error. The pump stayed off until someone started it by hand, and nothing reported the failure.
- Transitions missed while Home Assistant is down (e.g. 2026-09-10) are rare.

## Goals

- A scheduled on or off still happens when the first command fails, provided the plug recovers within about 10 minutes.
- Admins are notified when a scheduled transition still hasn't happened after that.
- The pump can still be controlled by hand between scheduled times.

## Approach

The two schedule entries call a script instead of switching the pump directly. The script sends the command, waits for the pump to report the target state, retries about once a minute for up to 10 minutes, and notifies admins if the pump never gets there. The next scheduled transition replaces any retries still running.

## Trade-offs and limits

- The schedule is only enforced at the scheduled times, so a manual change between them stands until the next one.
- If the plug is still unreachable after about 10 minutes, the transition is abandoned and admins are notified; nothing retries later.
- Transitions missed while Home Assistant is down aren't caught up; they're rare (see Context).

## Capabilities

- `pool-pump-schedule` (new)
