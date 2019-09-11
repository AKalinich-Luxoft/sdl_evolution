# Improve iAP2 over BT performance during apps registraion

* Proposal: [SDL-NNNN](NNNN-improve-iap2-bt-performance.md)
* Author: [Andrii Kalinich](https://github.com/AKalinich-Luxoft)
* Status: **Not created**
* Review manager: N/A
* Impacted Platforms: [SDL core]

## Introduction

This proposal will be to introduce one of the way of improvement IAP2 over BT performance by reducing BT channel bandwidth load during applications registration. This will be done by postponing OnPermissionChange notification sent to mobile apps upon registration.

## Motivation

BT transport is not so fast for a data transfer as for example USB. BT transport has a quite limited bandwidth and when user is trying to register multiple applications over BT, it may cause a big delays between sent request and received response which might be a performance issue. To speed up applications registration process, SDL should send less information in a short time periods to reduce a channel bandwidth load.

## Proposed solution

To reduce the channel bandwidth load, were analyzed all RPCs sent by SDL to mobile apps during app registration process.
One of the way for improvement is to postpone some RPCs during apps registration and send them later as it's not so critical to send them right after registration.
Below is the image showing the RPCs between mobile app and SDL during app registration on Sync3.2v2:

![RPC sizes](https://github.com/AKalinich-Luxoft/sdl_evolution/blob/feature/improve_IAP2_BT_performance/assets/proposals/RPC_size.png)

From the image above can be seen that one of the biggest RPCs which can be postponed by SDL is OnPermissionChange.
All other RPCs are either received from mobile (and can't be controlled by SDL) or should be sent right after registration.
By that reason, the proposed solution is to postpone OnPermissionChange sending for a some time which can be configured in ini file.


## Detailed design

Proposed to add "PendingPermissionsCache" and "PermissionsSender Timer" entities to PolicyManager component.
Also add "AfterRegistrationTimeout" (called timeout_X on draft) and "BetweenAppsTimeout" (called timeout_Y on draft). These timeout should be configurable in SDL ini file.
Below are the possible use cases and sequence diagrams describing in details how proposal works.

### Use case 1.

When application 1 is registered, PolicyManager adds this application to internal FIFO-queue "PendingPermissionsCache" and starts "PermissionsSender Timer" with timeout_X. When timeout is expired and no other applications were registered, SDL removes application 1 from queue and sends OnPermissionChange notification to application 1.

![Case 1](https://github.com/AKalinich-Luxoft/sdl_evolution/blob/feature/improve_IAP2_BT_performance/assets/proposals/Case4.png)

### Use case 2.

When application 1 is registered, PolicyManager adds this application to internal FIFO-queue "PendingPermissionsCache" and starts "PermissionsSender Timer" with timeout_X.
When application 2 is registered, PolicyManager adds this application to internal FIFO-queue "PendingPermissionsCache" and restarts "PermissionsSender Timer" with timeout_X (that's it - timeout will be reset).
When timeout is expired and no other applications were registered, SDL removes application 1 from queue and sends OnPermissionChange notification to application 1.
After that PolicyManager starts "PendingPermissionsCache", but with timeout_Y.
When timeout is expired and no other applications were registered, SDL removes application 2 from queue and sends OnPermissionChange notification to application 2.
Two separate timeouts (timeout_X and timeout_Y) were declared because first one can have a quite big value for example 10 sec and there is no sense to wait 10 secs between timer iterations. By that reason timeout_Y (which can be 1 sec) will be used to send OnPermissionChange notifications one by one if no other apps were registered.

![Case 2](https://github.com/AKalinich-Luxoft/sdl_evolution/blob/feature/improve_IAP2_BT_performance/assets/proposals/Case1.png)

### Use case 3.

When application 1 is registered, PolicyManager adds this application to internal FIFO-queue "PendingPermissionsCache" and starts "PermissionsSender Timer" with timeout_X.
When application 2 is registered, PolicyManager adds this application to internal FIFO-queue "PendingPermissionsCache" and restarts "PermissionsSender Timer" with timeout_X (that's it - timeout will be refreshed).
When user activates application 1 on HMI, SDL will remove application 1 from "PendingPermissionsCache" and send OnPermissionChange notification for this app immediately. Note that there is no need to restart the timer.
When timeout is expired and no other applications was registered, SDL removes application 2 from queue and sends OnPermissionChange notification to application 2.

![Case 3](https://github.com/AKalinich-Luxoft/sdl_evolution/blob/feature/improve_IAP2_BT_performance/assets/proposals/Case3.png)

### Use case 4.

When application 1 is registered, PolicyManager adds this application to internal FIFO-queue "PendingPermissionsCache" and starts "PermissionsSender Timer" with timeout_X.
When application 2 is registered, PolicyManager adds this application to internal FIFO-queue "PendingPermissionsCache" and restarts "PermissionsSender Timer" with timeout_X (that's it - timeout will be refreshed).
When timeout is expired and no other applications was registered, SDL removes application 1 from queue and sends OnPermissionChange notification to application 1.
After that PolicyManager starts "PendingPermissionsCache", but with timeout_Y.
When application 3 is registered, PolicyManager adds this application to internal FIFO-queue "PendingPermissionsCache" and starts "PermissionsSender Timer" with timeout_X (that's it - timeout_X will overwrite the current timeout_Y)
When timeout is expired and no other applications was registered, SDL removes application 2 from queue and sends OnPermissionChange notification to application 2.
When timeout is expired and no other applications was registered, SDL removes application 3 from queue and sends OnPermissionChange notification to application 3.

![Case 4](https://github.com/AKalinich-Luxoft/sdl_evolution/blob/feature/improve_IAP2_BT_performance/assets/proposals/Case2.png)

## Impact on existing code

SDL policies component may be affected by this proposal. By that reason, policies component should be tested thoroughly.

## Alternatives considered

It was considered to postpone another RPCs which were added in the latest SDL releases but was not present in Sync3.2v2.

Also, this proposal is just for IAP2 performance improvement so if there is no any performance issues observed on the latest releases - SDL implementation can be kept without proposed changes.
