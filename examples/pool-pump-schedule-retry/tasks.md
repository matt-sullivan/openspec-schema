# Tasks

## 1. Retry scripts and scheduler

- [x] 1.1 Create `packages/raspipool/pool_pump_schedule.yaml` with `script.pool_pump_command` and `script.pool_pump_set_state` per design; verify: `ha core check` passes and both scripts exist after reload.
- [x] 1.2 Change the "Pool Pump On"/"Pool Pump Off" scheduler actions to call `script.pool_pump_set_state` with `target: on`/`off` (scheduler UI or `scheduler.edit`); verify: the scheduler switches' `actions` attributes show the script.

## 2. Acceptance

"Plug offline" means the plug is unpowered at the wall and `switch.pool_pool_pump` shows `unavailable`. When restoring power, use a target opposite to the plug's power-up state (check the Smart Life app). Tests run `script.pool_pump_set_state` by hand in place of a scheduled time.

- [x] 2.1 Scenario "Command succeeds first time": pump off, run with `target: on` → pump turns on after one command; no notification.
- [x] 2.2 Scenario "Pump already in scheduled state": pump on, run with `target: on` → no command sent (trace).
- [x] 2.3 Scenario "Command fails then succeeds": plug offline, run with `target: on`, restore power after a few attempts → an attempt about every minute, the loop continues past inner command errors, and the next retry turns the pump on; retrying stops; no notification. (manual)
- [x] 2.4 Scenario "Pump reaches scheduled state by other means": plug offline, run with `target: on`, restore power and turn the pump on from HA before the next retry → retrying stops. (manual)
- [x] 2.5 Scenario "All retries fail": plug offline for over 10 minutes, run with `target: on` → an admin notification names the failed transition. (manual)
- [x] 2.6 Scenario "Off time arrives during on retries": plug offline, start `target: on`, then run `target: off` → the on run stops and the off run proceeds. (manual)
- [x] 2.7 End-to-end: the next scheduled 10:00 and 16:00 transitions → the script trace runs and the pump changes state. (deferred)

## 3. Wrap-up

- [x] 3.1 Update project docs with the retry behaviour and the Tuya `continue_on_error` limitation; verify: docs name the package file.
