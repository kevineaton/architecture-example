---
title: Walks
summary: Walks provide the ability for a customer to put the collar in a special mode and store metadata about a period of time identified as a `walk`. Walks have the differing distinction of other activities in that it's a unique mode that triggers various changes in logic on the apps and devices. This is ssomewhat similar to Live Tracking so many of the same concepts come up. 
tags:
  - walks
  - activities
---

## Overview

Walks provide the ability for a customer to put the collar in a special mode and store metadata about a period of time identified as a `walk`. Walks have the differing distinction of other activities in that it's a unique mode that triggers various changes in logic on the apps and devices. This is ssomewhat similar to Live Tracking so many of the same concepts come up.

**Remember**: Settings are what we *want* to happen. Status bars are what *is* happening. In this design, the user tells the services that they want the mode on or off by changing the setting. The device tells the system that the mode is on or off using the status bar.

The big difference is that the `Walk` is a discrete "thing" where as a live tracking session isn't. So there are some conceptual differences.

### Status

Proposal.

### Major Changes from Previous

In the previous flow, all of the walks and status information was stored locally on the collar. Walk mode was started on the phone, sent over BT, and then the collar kept track. When the walk ended, the collar consolidated the information and sent it up to the server. If BT failed, or if the collar never started, then nothing would actually come up, or what came up would not be complete or what the customer expects. The server will add a `walkModeEnabled` of `on` or `off`, although that would not be required necessarily. It would, however, make it more consistent with `lostMode` and `liveTrackingEnabled`.

This change places creation and finishing as two separate processes that:

- Does not rely on Bluetooth connectivity
- Is resilient to Bluetooth drops
- Allows editing and management

#### Potential Improvements

- Change the collar setting for walk mode to an int identifying the current walk id

### Summation of Flow - Full Connectivity

- The user chooses Start Walk in some sort of UI.
- The Mobile App POSTs to the server for a walk, leaving `endTime` unset.
- The server creates the Walk Object and an ID is created.
- The server sets the `walkModeEnabled` collar setting as `on`. Note: this can be discussed, perfectly happy with the Mobile App also making this call.
  - This will ensure that the Walk will continue post-crash or restart.
- The Mobile App receives the Walk and sends a `startWalk` action to the device with the `walkId`
- The device receives the action, either via Bluetooth OR the server
  - This allows the future feature of starting a walk without the phone being connected to the device, such as for dog walkers or other accounts
  - If sent over BT and the Walk Id cannot be included for whatever reason, the device will need to make a call to get the latest Walk for the user and assume that it is that walk IF there is NO `endTime`
  - If that last call isn't possible, it will need to rely on the walk id sent during the `stopWalk` action
- The device receives the action and updates the status bar. The device is now in Walk mode
- The Mobile App monitors the status bar for the following states, even if they are not currently supported in the UI:
  - Setting `off` and status bar does not report on a walk: No Walk
  - Setting `on` and status bar does not report on a walk: Pending start
  - Setting `on` and status bar does report on a walk: On a walk
  - Setting `off` and status bar does report on a walk: Pending stop
- The walk continues as normal. BT dropping or device crashes do not change this as the device knows it is on a walk, knows what the walk id is, and can get that information if it doesn't
- The user chooses End Walk in some sort of UI
- The Mobile App sends an action to the device over BT or via the server to `stopWalk` with the Walk Id, just in case.
- The device receives the action and sends a PATCH to the server with the locations it has and the end time *just in case the locations have not been uploaded anyway*
- The device updates its status bar to no longer be in walk mode
- The Mobile App sends a PATCH to the collar settings turning off the `walkModeEnabled` setting
- The Mobile App also sends a PATCH to the server for the walk with and `endTime`. This allows redundancy if the device cannot talk to the server.
- The server, any time a POST or PATCH is received with a start and end time, currently does a GET on the locations between those times AND takes the body of the PATCH and consolidates the locations. Therefore, sending additional data with a start and end time of a walk is not harmful.

### Summation of Flow - No Server Connectivity

The biggest difference in this flow is that a Walk Id cannot be generated since there's no connectivity to the server. This also means that the settings cannot be changed and all information must occur over BT. If at any point connectivity could be established, the Mobile Apps could create a Walk object with the gathered data so far. Alternatively, they could set the Walk information later.

- The user chooses Start Walk in some sort of UI.
- The Mobile App receives the Walk and sends a `startWalk` action to the device without a `walkId`
  - The device stores the `startTime` of the walk locally
- The device receives the action via Bluetooth
  - Since there is no walk object, it's possible that the data could be lost if we are not careful here; the device will need to rely on the walk id sent during the `stopWalk` action
- The device receives the action and updates the status bar. The device is now in Walk mode
- Since there is no internet connectivity, it is assumed the Collar Settings cannot be modified, so the Mobile Apps will only be able to tell if the status bar reports on or off
  - Option A
    - The Mobile App kicks off a 1 minute timer where it tells the user the walk is pending start
    - If the Status Bar reports the device is in walk mode, the Mobile App tells the user it is in walk mode
    - If after the timer expires there is no Status Bar indicating the walk mode began, tell the user the walk could not be started
  - Option B
    - The Mobile App tells the user the walk has started, regardless of the reported Status Bar
    - Since the Mobile App will make the POST at the end, it means the walk-mode-specific features won't be turned on, but any locations will be back-ported into the walk
- The walk continues as normal. BT dropping or device crashes do not change this as the device knows it is on a walk and the Mobile Apps have a `startTime`
- The user chooses End Walk in some sort of UI
- The Mobile App sends an action to the device over BT to `stopWalk`
  - Assuming there still is not connectivity, the device can keep any non-uploaded walks or communicate them over BT as a response to this action
- The device updates its status bar to no longer be in walk mode
- The Mobile App sends a POST to the server with the `startTime`, `endTime`, and `locations` of the walk
- The server, any time a POST or PATCH is received with a start and end time, currently does a GET on the locations between those times AND takes the body of the PATCH and consolidates the locations. Therefore, sending additional data with a start and end time of a walk is not harmful.

**Note:** This flow has the inherent challenge of what to do if the collar has locations it hasn't sent back over BT as part of the walk if the Mobile App sends the information at the end. However, if they are eventually uploaded, we could add a check to the server that if the `activity` field of the `location` is set to `walk` then we can try to handle it and re-associate it.

{{walks}}
