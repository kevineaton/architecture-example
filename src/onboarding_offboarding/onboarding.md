---
  title: Onboarding / Offboarding
  summary: Describes onboarding and offboarding flows and calls
  tags:
    - device
    - onboarding
    - offboarding
---

# Onboarding and Offboarding

## Onboarding

Onboarding may require pre-registration. If it does, then you need to first pre-register the device with a call prior to activating the device like normal. Some devices require a plan during pre-registration, others may not. For pre-registered devices that require a subscription, it is assumed a payment method is already configured.

[API Docs](https://apidocs.api.App.com/#tag/Devices/operation/Devices%20-%20Preregister)

{{device_onboarding_pre_reg}}

{{device_onboarding_pre_reg_code}}

## Offboarding

Offboarding a device is simpler. For devices with a Thing Shadow connection, it is a two-step process. For these devices, the device must be told it needs to reset. This is best accomplished with an action of `unlinkDevice` being sent. Note that if the device is corrupted and a developer needs to offboard it manually, this can be accomplished by using curl or Postman and sending the responses on behalf of the device. Just be sure to flash the device, as the server will assume it's already been wiped.

[API Docs - Deactivate](https://apidocs.api.App.com/#tag/Devices/operation/Device%20-%20Deactivate)

[API Docs - Update Action](https://apidocs.api.App.com/#tag/Actions/operation/Action%20Update)

{{device_offboarding_with_ts}}

{{device_offboarding_with_ts_code}}
