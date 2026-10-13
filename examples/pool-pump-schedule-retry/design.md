# Design

## Technical context

- `switch.pool_pool_pump` uses the Tuya cloud integration. Its state comes from device reports rather than being set optimistically, so a failed command leaves the state unchanged.
- The schedule entries are "Pool Pump On" (`switch.schedule_pool_pump_on`) and "Pool Pump Off" (`switch.schedule_pool_pump_off`). Each calls `switch.turn_on` or `switch.turn_off` directly. Scheduler service calls are non-blocking.
- The Tuya SDK silently drops a command identical to one sent to the same device less than 10 s earlier, so retries must be at least 10 s apart.
- Tuya command failures raise exceptions that aren't `HomeAssistantError`, such as `tuya_sharing.exceptions.ApiRequestException` and `urllib3` `MaxRetryError`.

## Decisions

### Retry script called by the scheduler

New script `script.pool_pump_set_state` with field `target` (`on`/`off`). Both schedule entries call it instead of `switch.turn_*`. The entries are changed through the scheduler UI or the `scheduler.edit` service, never by editing `.storage`.

Script logic:
- `mode: restart`, so the next scheduled transition replaces an in-progress one.
- Repeat while the pump isn't in the `target` state and fewer than 10 attempts have run: send one command, then wait up to 60 s for the pump state to equal `target`. The wait also paces retries at one per minute, well clear of the Tuya 10 s duplicate window.
- If the pump still isn't in the `target` state after the loop, notify `notify.admins`.

- **Alternative:** automations driven by `input_datetime` helpers instead of the scheduler — rejected because it would drop the scheduler UI used to manage the times.
- **Alternative:** move the plug to local Tuya control so commands don't depend on the cloud — not pursued in this change.

### Commands sent from a separate fire-and-forget script

Each attempt starts `script.pool_pump_command` (field `target`) with `script.turn_on`, and that script calls `switch.turn_on`/`turn_off`. A Tuya exception then aborts only that inner run and is logged, and the retry loop carries on.

- **Alternative:** `continue_on_error: true` on the command — rejected because it only suppresses `HomeAssistantError`, so Tuya's exceptions would still abort the loop.

### Pump switch state as the success signal

Success is judged by the pump's switch state, not its power sensor. The switch state is the device's own report, while power can lag or read low during priming.

### Placement

Both scripts go in a new package file, `packages/raspipool/pool_pump_schedule.yaml`.

## Risks

- **A manual command that matches the pump's current state (e.g. "off" while it's already off, after a failed scheduled "on") doesn't change the state, so retries continue** → Retries end after 10 minutes, or can be stopped by turning off `script.pool_pump_set_state`.
