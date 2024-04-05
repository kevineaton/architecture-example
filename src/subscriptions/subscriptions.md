---
title: Subscription Flows
summary: How subscriptions work and some potential options
tags:
  - subscriptions
  - devices
  - billing
  - activation
---

## Current

Currently, you have a few ways to think of devices. For the purposes of this section, we will talk about devices as such:

- Needs Subscription
  - Freedom 2020 Collar
  - Small Pet Collar
  - Freedom 2023 Collar
- No Subscription, Needs Pre-Registration
  - Smart Bowl
  - Healthbox
- No Subscription, No Pre-Registrations
  - Tagz
  - Pendants

**Important Context**: There's a few things to keep in mind here.

First, we absolutely must do whatever we can to:

- Avoid giving a cut of revenue to another party (we already lose a cut to Stripe and Chargify)
- Minimize customer mistakes, typos, and frustration
- Ensure a good UX and simple flow
- Minimize and mitigate security concerns
- Provide as much future-proofing as possible

Second, we don't want to over-complicate things if we can avoid it. However, when it comes to money, the extra engineering effort is usually worth it, even if it means significant changes.

### Current - Needs Subscription

{{current_needs_sub}}

### Current - No Subscription, Needs Pre-Registration

{{current_no_sub_needs_prereg}}

### Current - No Subscription, No Preregistration

These devices do not require a subscription and there's minimal harm in activating/deactivating them. A key characteristic of these devices is that there is no internal state and they are fairly "simple" devices.

{{current_no_sub_no_prereg}}

## Potential Improvements

On 20230323 and 20230324, several meetings occurred to discuss options for onboarding the Smart Pet Bowl product. As a result, an analysis of the bigger picture reveals that we have the opportunity to improve this flow to better meet the needs of our company and customers. Below is a distillation of two potential options for refactor, in order of easier to complicated.

### Option 1

In this option, we try to consolidate some of the flows above while improving the user experience. However, this flow does NOT provide the future-proofing desired for pre-selling subscriptions.

The key changes are:

- User selects product on the web app FIRST, rather than entering a serial
- If the user selects a product that does not need a subscription, they can activate directly in the app
- If the user selects a product that does need a subscription, they continue to configure that in the web app

{{potential_option_1}}

### Option 2

In this option, the entire subscription system is refactored to focus on improved flexibility.

The key changes are:

- User selects product on the web app FIRST, rather than entering a serial
- If the user selects a product that does not need a subscription, they can activate directly in the app
- If the user selects a product that does need a subscription, they continue to configure that in the web app
- The user sets up a "family subscription", NOT a specific serial subscription
  - In other words, they purchase a Freedom 2023 subscription, not a subscription for serial 2301230000
- The user can optionally set up multiple buckets at once, or purchase multiple
  - For example, there's no technical reason the web app couldn't prompt for how many Freedom 2023 subscriptions,
    or send up at the end a request for 1 2023 sub, 2 bowl subs, and a cat collar sub
- There are no more "pending" devices, just "pending" subscriptions
  - In the app, there will need to be a discussion about how best to verify
    - Option A [Preferred]: App sends a list of 1 or more serials to an endpoint for the user, server says "for each of these, there is / is not a subscription available so it can / cannot be activated"; will allow multiple activations at once if desired
    - Option B: App makes a call to get the list of subscriptions and checks locally to determine if found devices can activate
- Potential: We could then be smart about recommending better solutions
  - Example: User sets up 2 2023 Annuals; we could ask the user if they would rather set up the multi-pet sub
  - Example: User sets up some package amount of subs that we can then put on a "App Total Plus" subscription that all of the devices are later connected to
  - NONE OF THIS BULLET IS DEMONSTRATED; IT IS HERE FOR THOUGHTFUL DISCUSSION

{{potential_option_2}}
