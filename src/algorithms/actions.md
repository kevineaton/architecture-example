---
title: Actions
summary: Algorithms for actions
tags:
  - actions
notes: If the proposed is implemented, rename to actions.d2 and update that diagram to ensure hotlinking works.
---

## Summary

Actions exist solely to convey a change or intent. For example, `startWalk` or `forceRefresh` or `updateSettings`. They are communicated over both Bluetooth and over the Server. When using the Server, they are written into Thing Shadow, which has a limited storage size for the entire body. As such, if they don't need to go to the server, they shouldn't, although the analytics are nice.

As of 20230403, there is minimal de-duplication identification in the actions. Therefore, we first list how actions **currently work**. Then, below, there is a proposal to rectify this back to the original specification that was lost.

### Current General Actions

{{actions}}

### Current Force Refresh Logic

The `forceRefresh` action needs some special clarification due to it's unique properties. It is used to tell the device that it should update its status to the server. However, this can easily become "spammy" so needs some extra considerations.

{{forceRefresh}}

## Proposed UUID Logic

The below logic and flows attempt to de-conflict the existing lack of true deduplication. In the original specifications, actions that were sent over both Bluetooth and the API were supposed to have an identifier to identify that they are indeed the same action. However, at some point that was either lost or otherwise dropped. The following changes should enable that back in.

Note that this is only an issue for actions sent over both Bluetooth AND over the API. If the actions comes straight from the API or comes only over Bluetooth, there is no duplication needed.

There are three general options:

- Option A
  - The server adds a text `uuid` field to the root of the action that is settable by the creator
  - Pros
    - Makes it a top-level field for search and display
    - Makes it obvious without body parsing
  - Cons
    - Adds overhead to every single action in the system, since even if it is `""` it will still take up bytes
    - If it needs to be indexed, adds to DB complexity and indexing
    - Impacts actions that go over both, or goes over just BT, or goes over just API
    - Requires the device to keep track of which `uuid`s it has handled so if it gets one over TS or BT, it knows whether it needs to process or ignore the action
- Option B
  - The server does not add any field
  - The caller adds the field to the `arguments` field of the action
    - Note that the `arguments` field is NOT checked, parsed, or otherwise looked at by the server, so would be an implementation detail
  - Pros
    - No server interaction
    - Only actions sent over both are impacted
    - No indexing or generalized space constraints
  - Cons
    - Not easy to tell which actions are duped at the top level
    - Requires parsing a JSON body that may or may not have the data it needs
    - Requires the device to keep track of which `uuid`s it has handled so if it gets one over TS or BT, it knows whether it needs to process or ignore the action
- Option C
  - Mobile app makes a call to the server to create an action, gets the ID back
  - Mobile app adds the action ID to the BT action when sent to the device
  - Pros
    - Uses existing infrastructure so no additional work needed on server team
    - Ensures ID generation is handled on server
    - No additional argument parsing or added field checks
    - Device already knows the id so does not need to worry about id look ups
  - Cons
    - Requires internet connectivity for mobile app

For Option A or Option B, the `uuid` only needs to be unique enough to identify it compared to other sources. Several options exist, including:

- Source/Time/Action: `source_YYYY-MM-DDTHH:mm:ssZ_action` (such as `ios_2023-04-03T19:04:05Z_startWalk`)
- Source/Random Number: `source_#############` (such as `android_456865313549753`)
- Local UUID generation: `aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa` (such as `acde070d-8c4c-4f0d-9d8a-162843c10333`)
  - Note that we would have to decide on a v1/v2 time-based uuid or on a different version

The below diagram will not assume a specific option and will instead just use the term `id` to mean either a locally generated `uuid` (Option A or B) or the generated server id (Option C).

Also note that server-generate actions will continue to work as normal without any specific `id` or `uuid`, since there's no way for the server to write them over BT. In other words, the server will not add this field or set it. It merely a tool for communication between the device and the apps.

{{actions_uuid_proposed}}

{{actions_uuid_proposed_flow}}
