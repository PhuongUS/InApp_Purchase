# Testing Auto-Renewable Subscriptions on iOS

Subscriptions work differently in TestFlight, the sandbox, and live on the App Store, so testing them requires knowledge of those differences. This is my attempt to help sort out the mess and document it for others to reference. 

##General Notes

Subscription length has been significantly shortened for testing purposes. This allows users to quickly test multiple renewals and expirations via TestFlight or with sandbox users.

Actual subscription duration -> Test duration

1 week -> 3 minutes  
1 month -> 5 minutes  
2 months -> 10 minutes  
3 months -> 15 minutes  
6 months -> 30 minutes  
1 year -> 1 hour  

The subscription will automatically renew 6 times per account per 8 hour window, then the subscription will automatically expire at the end of each subscription period. These renewals happen automatically whether the app is open or not, just like renewals on the App Store. Unlike the App Store, there’s no option to cancel, so there’s no way to directly test cancelation. There’s also no way to test subscription management while using TestFlight or sandbox users.

Each automatic renewal sends a transaction to the app. The transaction, or transactions, depending on how much time has passed, is processed the next time the app is opened. Validating these transactions triggers yet another password prompt. When the app is live on the App Store it shouldn’t trigger these additional password prompts.

##TestFlight Testing
First off, subscribing in a TestFlight distributed app prompts the user 3 times for their Apple ID password and does not allow the use of Touch ID to complete the transaction. Sometimes, after starting a new subscription multiple times on the same device (after the first 6 auto-renewals), only 2 password prompts are displayed ¯\_(ツ)_/¯. It would be nice if the purchase flow while testing mirrored that of a live app on the App Store, but that’s just not the case for whatever reason. It’s a bit disconcerting to see so many password requests while testing, but rest assured, once the app is live on the App Store the purchase flow will work as expected (which is a single, Touch ID enabled purchase sheet on iOS 11).

Testing renewals and expiration:

1. Subscribe to a monthly subscription
2. Close the app and set a 5 minute timer
3. Launch the app
4. If prompt is displayed, enter password

At this point the app should continue to operate in the subscribed state. Repeat steps 2-4 several more times (or just close the app and wait) until 35 minutes has passed (6 renewals at 5 minutes each plus the original 5 minute subscription). The app should now revert to the un-subscribed state and allow the user to re-subscribe.

Test restoring purchases after expiration:

1. Subscribe to a monthly subscription
2. Close the app and wait 35 minutes
3. Launch the app
4. If prompt is displayed, enter password
[app should revert to the un-subscribed state]
5. Tap “Restore Purchases” button

No active subscription should be found and the user should be shown a message to that effect.

Test restoring purchases during active subscription:

1. Subscribe to a monthly subscription
2. Delete app and reinstall within 5 minutes
3. Launch the app
4. Tap “Restore Purchases” button

Active subscription should be found and the app should change to the subscribed state.

Test restoring purchases across devices:

1. Subscribe to a monthly subscription
2. Install the app on a different device before the subscription expires
3. Launch the app
4. Tap “Restore Purchases” button

Active subscription should be found and the app should change to the subscribed state.

##TestFlight Caveats
Note that since the subscription is only auto-renewed 6 times in an 8 hour window, some of the procedures above will change if those 6 renewals have already happened.

If the subscription is associated with an account and the subscription status maintained server-side, the testing procedures get much more complicated and are beyond the scope of this post.

One thing to keep in mind is that with subscriptions being so hard to initiate (3 password prompts, no Touch ID) and renew (another password prompt, no Touch ID), the subscribed state is likely to get significantly less testing from beta users than the un-subscribed state. Consider creating a secret unlock code/gesture to allow beta testers access to the subscribed state for longer periods of time without having to jump through hoops.

##Production Testing
For an app that has yet to be released on the App Store, getting an early version of the app approved is a great way to test subscriptions:

1. Submit a beta version of the app to App Review. Make sure to set “Version Release” to “Manually release this version” so that the app is not released on the App Store.
2. Generate promo codes for the app. This can be done for free apps that are approved, but not yet on the App Store.
3. Download the app from the App Store using a promo code.
4. Subscribe.

Since this app has gone through approval, subscriptions will perform exactly as they will when the app is live on the App Store, including charging testers who subscribe and allowing testers to manage the subscription in the App Store app. Promo codes can be given to testers so that they can test the app for free. Subscriptions paid for via promo code work exactly like paid subscriptions, except that they don’t auto-renew.

##Sandbox Testing
Finally, it’s also possible to test auto-renewable subscriptions via sandbox test accounts, but this form of testing has been well documented, so I won’t bother rehashing how that works.

##Rejection
After going through all the testing mentioned above, my app was rejected for not properly disclosing the subscription in the app and in the App Store description. To save others time, here’s the relavent section from the rejection notice:

We noticed that your app did not fully meet the terms and conditions for auto-renewing subscriptions, as specified in Schedule 2, Section 3.8(b), included below:

- You clearly and conspicuously disclose to users the following information regarding Your auto-renewing subscription:
    - Title of publication or service – Length of subscription (time period and/or content/services provided during each subscription period)   
    - Price of subscription, and price per unit if appropriate   
    - Payment will be charged to iTunes Account at confirmation of purchase   
    - Subscription automatically renews unless auto-renew is turned off at least 24-hours before the end of the current period   
    - Account will be charged for renewal within 24-hours prior to the end of the current period, and identify the cost of the renewal   
    - Subscriptions may be managed by the user and auto-renewal may be turned off by going to the user’s Account Settings after purchase   
    - Links to Your Privacy Policy and Terms of Use   
    - Any unused portion of a free trial period, if offered, will be forfeited when the user purchases a subscription to that publication, where applicable  

##Solution
Using The Atlantic app and the above text as a reference, here’s the text I used in my app:

Information about Weather Atlas Pro subscriptions:
- Payment will be charged to your iTunes account at confirmation of purchase.
- Your subscription will automatically renew unless auto-renew is turned off at least 24-hours before the end of the current subscription period.
- Your account will be charged for renewal within 24-hours prior to the end of the current subscription period. Automatic renewals will cost the same price you were originally charged for the subscription.
- You can manage your subscriptions and turn off auto-renewal by going to your Account Settings on the App Store after purchase.
- Read our terms of service and privacy policy for more information.
