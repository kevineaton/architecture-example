---
title: Replenishment
summary: Replenishment is the architecture for how our system re-orders (replenishes) a consumable for a user
tags:
  - replenishment
  - third-party
---

# Replenishment

Replenishment is the architecture for how our system re-orders (replenishes) a consumable for a user. There are three possible flows, depending on capabilities.

## Definitions

- Provider: an organizational provider that offers to sell the consumables tracked. For example, this may be a retailer or warehouse.
- Customer: A App customer who uses the App ecosystem
- Server: A networked computer system that provides access to a RESTful API for server-to-server communication

## Server to Server - Provider Focus

In this flow, both servers communicate over an agreed-upon protocol. Most of the customer information is handled by the Provider. Often, this is made available through a web-portal in which the Customer enters payment information and selects products. When set up, the Provider sends a token back to the App server. That token is then used to make purchases on behalf of the customer. The Provider provides updated status information for the order to App and/or the customer directly. The customer manages their payment information and product selection in the Provider's systems.

{{replenishment_s2s_provider}}

## Server to Server - App Focus

In this flow, both servers communicate over an agreed-upon protocol. Most of the customer information is handled by the App server. The customer will set up a payment method and a desired product for replenishment within the App App. When it is time to place an order, the App server sends a unique token to the Provider's server with what product to purchase, the quantity, and where to ship the product. The Provider handles the purchase and reports status updates back to App. App may also cancel the order if the customer informs App before the product ships. The customer relationship is managed entirely in the App ecosystem.

{{replenishment_s2s_App}}

## App Hosted

In this flow, the replenishment system is managed entirely on App IT systems. A providers signs up for an account with a hosted, to be developed system. All of the management and relationship information is managed by App. The Provider provides list of capabilities (delivery, pick up only, etc) and products. As replenishment is needed, App will handle the allocation of quantities and inform the Provider through the system and through emails. Most of the work is handled by App, and the provider only has to manage their inventory through the hosted platform and then respond to orders.

*Note*: This is not yet built and is a theoretical option at this time. This is mostly relevant for smaller Providers.

{{replenishment_hosted}}
