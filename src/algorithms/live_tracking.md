---
title: Live Tracking
summary: Algorithms for Live Tracking
tags:
  - live tracking
---

## General Flow for Starting

This shows the general flow for starting Live Tracking mode. A few assumptions need to be made, however. First and foremost, we cannot guarantee that a collar is on WiFi, Bluetooth, or Cell. So we have to make sure we have a good communication interface for the user to prevent confusion. A key use case for this is that the dog is lost, which means Bluetooth and Wifi are pretty much non-existent. So cell latency for update checks needs to be considered.

**Remember**: Settings are what we *want* to happen. Status bars are what *is* happening. In this design, the user tells the services that they want the mode on or off by changing the setting. The device tells the system that the mode is on or off using the status bar. The one caveat is that when the timeout expires, the device needs to tell the user it turned itself off in the status bar AND in the collar settings. Otherwise, the device will just turn the mode back on with the next boot.

**Potential Mitigation: Cell Actions**: When a pet crosses a geofence, turn the cell check updates to 1 minute. When they cross back, go back to the previous value from the server.

**Potential Mitigation: User Updates** If Live Tracking is requested but a socket cannot be established, reduce polling interval to every 30 second / 1 minute / 2 minutes.

## Status

Accepted, in work.

### Summation of Discussions

- The collar is the one who decides whether a timeout has hit or not. While the user can send info to turn it on or off, pending any additional interaction the collar will timeout.
- The Collar Setting of `liveTrackingEnabled` specifies that the collar should be in live tracking mode. This is for persistence ONLY.  In other words, if this is set, then LT should be on, even if there is a reboot.
- The Status Bar fields of `liveTrackingEnabled` and `liveTrackingEnabledUntil` show what the collar is currently doing. This doesn't say what it SHOULD be doing, only what it understands it is CURRENTLY doing.
- The fundamental issue here is that we need all of the persistence, the state, and the trigger.
- We cannot rely on `updateSettings` as that is a PATCH which is idempotent. While we could use it for on -> off or off -> on, you cannot use it for on -> refresh.
- Similarly, it doesn't make sense to have a setting of `requested` or something similar. The user doesn't WANT it to be requested. That is mixing state and persistence.
- For scalability and maintainability, the servers try to minimize the amount of custom logic that has to go in to action parsing, especially given the spamminess of some actions (especially `forceRefresh`). Additional processing here adds real overhead. Unfortunately, with the desired outcomes, this is unavoidable.
- It's generally a Bad Idea to hardcode values in clients, such as timeouts. For features, values should be stored server side. Avoid "magic numbers."
- Although not a part of this current spec, it's easily imaginable that a user will want to specify a length of time for live tracking, especially when the reason for the tracking is time-based (known walking path, known activity). This inherently necessitates a new action. Simply providing an `updateSettings` action will not convey that, and under no circumstances should another entity modify a status bar for a device. It's not a 2-way communication.
- New fields to support this are `liveTrackingEnabled` on the Collar Settings and `liveTrackingEnabled` and `liveTrackingEnabledUntil` on the Status Bar.
- A new action of `setLiveTracking` will be created. This will be used to start, refresh, and end live tracking.
- The mobile apps will send a PATCH to set the `liveTrackingEnabled` Collar Setting to `on`. This signifies that the user wants this to be on (NOT THAT IT IS ON!).
- The mobile apps will send a POST to sent the new action `setLiveTracking`. Optionally, it will have `arguments` of `liveTrackingEnabled` and `liveTrackingEnabledUntil`.
  - If `liveTrackingEnabled` is omitted, the server will look at the current Collar Settings and use the value there. This does mean there could be a race condition, so it will be best to set it.
  - If `liveTrackingEnabledUntil` is omitted, the server will set it to the UTC timestamp of the current time + the server-specified timeout.
  - This enables more control over the functionality.
- The collar receives an `updateSettings` action automatically sent by the server. It processes it normally.
- The collar receives the `setLiveTracking` action and processes it, updating the values.
- The collar sends a status bar with its current state.
- The mobile apps will then know the difference between request
  - Pending (Setting is `on`, Status Bar is `off`
  - On (both `on`)
  - About to expire (`liveTrackingEnabledUntil` in the near future)
  - Requested or turning off (Setting is `off`, Status Bar is `on`)
  - Off (both `off`).

Yes, there is redundancy, but given the requirements and current work as of this creation, it's the best I can do.

### Ending Live Tracking

Live Tracking can be stopped in one of a few ways:

- User requested it to end
- Timer has expired and no refresh was requested
- Time has expired and a refresh was requested

{{live_tracking}}
