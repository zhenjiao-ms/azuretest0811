---
title: Send Push Notifications to Authenticated Users (.NET Backend)
description: Learn how to send push notifications to specific
services: mobile-services,notification-hubs
documentationcenter: ios
author: krisragh
manager: dwrede
editor: ''

ms.service: mobile-services
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: article
ms.date: 07/21/2016
ms.author: krisragh

---
# Send push notifications to authenticated users
[!INCLUDE [mobile-services-selector-push-users](../../includes/mobile-services-selector-push-users.md)]

&nbsp;

[!INCLUDE [mobile-service-note-mobile-apps](../../includes/mobile-services-note-mobile-apps.md)]

> For the equivalent Mobile Apps version of this topic, see [How to: Send push notifications to an authenticated user](../app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#push-user).
> 
> 

In this topic, you learn how to send push notifications to an authenticated user on iOS. Before starting this tutorial, complete [Get started with authentication](mobile-services-dotnet-backend-ios-get-started-users.md) and [Get started with push notifications](mobile-services-dotnet-backend-ios-get-started-push.md) first.

In this tutorial, you require users to authenticate first, register with the notification hub for push notifications, and update server scripts to send those notifications to only authenticated users.

## <a name="register"></a>Update service to require authentication to register
[!INCLUDE [mobile-services-dotnet-backend-push-notifications-app-users](../../includes/mobile-services-dotnet-backend-push-notifications-app-users.md)]

## <a name="update-app"></a>Update app to sign in before registration
[!INCLUDE [mobile-services-ios-push-notifications-app-users-login](../../includes/mobile-services-ios-push-notifications-app-users-login.md)]

## <a name="test"></a>Test app
[!INCLUDE [mobile-services-ios-push-notifications-app-users-test-app](../../includes/mobile-services-ios-push-notifications-app-users-test-app.md)]

<!-- Anchors. -->
[Updating the service to require authentication for registration]: #register
[Updating the app to log in before registration]: #update-app
[Testing the app]: #test
[Next Steps]:#next-steps


<!-- URLs. -->
[Get started with authentication]: mobile-services-dotnet-backend-ios-get-started-users.md
[Get started with push notifications]: mobile-services-dotnet-backend-ios-get-started-push.md
[Mobile Services .NET How-to Conceptual Reference]: /develop/mobile/how-to-guides/work-with-net-client-library
