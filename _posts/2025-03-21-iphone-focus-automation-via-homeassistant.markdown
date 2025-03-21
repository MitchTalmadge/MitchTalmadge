---
layout: post
title:  "iPhone Focus Automation via Home Assistant"
categories: home-automation
---

I have a simple goal: **Make my iPhone go into Sleep focus via some automation in Home Assistant.**

*(TL;DR: I don't have a solution yet. Still trying to figure this out.)*

My use case is an [Arre Smart Button](https://arrehome.com/pages/arre-smart-button) on my bedside table that I click to activate my "good night" routine: turn off the lights, set the thermostat, and (ideally) set my phone to Sleep focus so that notifications don't make sounds.

That last bit is proving to be a lot harder than I expected. The only way to turn on the Sleep focus without relying on time-based automation is to use the built-in **Shortcuts** app. What I need is an automation trigger that can be invoked from another server. Here are all of the automation triggers that Apple has implemented:

![Screenshot 1/4 from iPhone of various available automation triggers](/assets//images/2025-03-20-iphone-automations-1.jpeg){: width="300" } 
![Screenshot 2/4 from iPhone of various available automation triggers](/assets//images/2025-03-20-iphone-automations-2.jpeg){: width="300" } 
![Screenshot 3/4 from iPhone of various available automation triggers](/assets//images/2025-03-20-iphone-automations-3.jpeg){: width="300" } 
![Screenshot 4/4 from iPhone of various available automation triggers](/assets//images/2025-03-20-iphone-automations-4.jpeg){: width="300" }

<!-- | Trigger Name         | Example                                    |
|:---------------------|:-------------------------------------------|
| Time of Day          | At 8:00 AM, weekdays                       |
| Alarm                | When my alarm is stopped                   |
| Sleep                | When Wind Down starts                      |
| Arrive               | When I arrive at the gym                   |
| Leave                | When I leave work                          |
| Before I Commute     | 15 minutes before I leave for work         |
| CarPlay              | When CarPlay connects                      |
| Email                | When I get an email from Jane              |
| Message              | When I get a message from Mom              |
| Transaction          | When I tap a Wallet Card or Pass           |
| Apple Watch Workout  | When I start a workout                     |
| Wi-Fi                | When my iPhone joins home Wi-Fi            |
| Bluetooth            | When my iPhone connects to AirPods         |
| NFC                  | When I tap an NFC tag                      |
| App                  | When 'Weather' is opened or closed         |
| Airplane Mode        | When Airplane Mode is turned on            |
| Low Power Mode       | When Low Power Mode is turned off          |
| Battery Level        | When battery level rises above 50%         |
| Charger              | When my iPhone connects to power           |
| Do Not Disturb       | When turning Do Not Disturb on             |
| Work                 | When turning Work on                       |
| Driving              | When turning Driving on                    |
| Reduce Interruptions | When turning Reduce Interruptions on       |
| Wind Down            | When turning Wind Down on                  |
| Sound Recognition    | When my iPhone recognizes a doorbell sound | -->

Right away, the only triggers that I can see being useful here are **Email** and **Message**. If I could have Home Assistant either email or text my phone, that should be enough to trigger my shortcut and turn on my Sleep focus...right?

# Attempt 1: Email

My main concern with this idea is that emails on my iPhone have **never** arrived quickly. If an app sends me a 6-digit code to sign in, for example, I always have to go into my Mail app and force a refresh before I get the email. Otherwise I'd be sitting there for 5-15 minutes before I get the email. 

The primary reason for slow mail is that only iCloud email accounts support "Push" notifications in the Mail app; all others must "Fetch" the mail every 15+ minutes. This is why my Gmail messages take forever to come in. From the little research I've done, it sounds like Google does not support Push notifications on third party apps, and instead only allow it on their own Gmail app. Lame...

So, to make this work, I need to send an email to my iCloud address. That should get it to my iPhone the fastest. To test this idea, I set up an automation to listen for emails on my iCloud address, and turn on Sleep focus when one arrives. Then, I sent an email to my iCloud address from my Gmail address. 

Unfortunately, it just straight up doesn't work. Not because the email is too slow (I mean, it takes about 5-10 seconds, which is slow but not horrible)... but because the automation just doesn't run at all. I double and triple checked that everything was correctly configured, but it would never even try to run the automation. [I found a thread on Reddit](https://www.reddit.com/r/shortcuts/comments/1d8g1n7/icloud_mail_doesnt_trigger_automation/) where people were complaining of this happening for iCloud accounts. I have to assume it's just an iOS 18 bug.

Moving on to my next idea... Messages?

# Attempt 2: SMS Messaging via Email

So I can't detect an email. Maybe I can detect an SMS? To send SMS messages to my phone, I would likely need to pay per message with a service like Twilio. It's not a ton of money ($0.0079 per message -- about 50 cents per month for two messages per day), but it's more the principle of it that hurts me. We live in an era where there are better, faster, and completely free options for push-notifying a cell phone than with SMS messages. Yet, here we are ...

First, though, I want to try a trick that I've messed with in the past. It's a little bit hacky, though. Did you know you can email a phone number? For example, with a T-Mobile number, you could send an email to e.g. `1234567890@tmomail.net` and it will arrive as a text message to `(123) 456-7890`. Verizon and some other carriers have killed this antiquated feature, but mine still supports it so I thought I'd at least try it.

This kinda worked. The first few messages came through. It was a little slow, around 5 seconds, but it did work. However, it quickly stopped working. Right away, the email server sends back a message saying that the email was blocked. It seems like they have some very aggressive spam filtering and are able to detect automated messages like "Goodnight!". Maybe I could find a way around this, but frankly I am in search of a more reliable solution, and this feels gross. Plus, it's probably against their TOS to automate such a thing, and I'd like to keep my phone number alive.

# Attempt 3: Apple Home Automation

While playing with the previous two ideas, I remembered that I can create automations in Apple Home. You can do things like turning on a light when a sensor detects too much CO2 in the air, or whatever. I have my Home Assistant and Apple Home services connected, so I can expose devices in Home Assistant into my Apple Home. Maybe I can use this to accomplish my goal?

I was able to create a virtual "switch" in Home Assistant that represents whether I am sleeping or not. I put this into Apple Home, and began creating an automation on it:

![](/assets/images/2025-03-20-iphone-home-automation-1.png)

...but, it's pretty useless. You can't set a focus. You can't run another shortcut. You can't really do *anything* that involves the iPhone itself. I assume that this automation would run on a remote server somewhere, perhaps even on my Apple TV (which tends to serve as a HomeKit hub), and therefore would have no direct access to any of the iPhone features. Lame!

# Attempt 4: SMS Messaging via Twilio

Ok, fine. I'll pay the 50 cents per month to solve this minor annoyance.

I rented a "Local" phone number on Twilio and installed the [Twilio integration](https://www.home-assistant.io/integrations/twilio/) on Home Assistant. Then I added a `notify` section to my `configuration.yml` as such ([docs for this are here](https://www.home-assistant.io/integrations/twilio_sms))...

```yml
# Example configuration.yaml entry
notify:
  - name: send_sms
    platform: twilio_sms
    from_number: "+12223334444" # Twilio number goes here
```

I rebooted Home Assistant and sent my SMS with the following action:

```yml
action: notify.send_sms
data:
  message: Good Night!
  target: 19998887777 # Personal number goes here
```

and!!! nothing happened. I had to check the Twilio error logs and saw this:

> Messages sent to US numbers will not be delivered if they are sent from numbers that are not associated with an approved A2P 10DLC Campaign

??? wat? fine. I guess this is [a new thing](https://www.youtube.com/watch?v=KWvbRToRnGg). I don't remember having to do this in the past when using Twilio for SMS. 

After some research, I learned that this is some new thing to cut down on spam. I have to register with "The Campaign Registry (TCR)" as a sole proprietor, then tell them what I plan to do with the number, who I am messaging, etc... And I have to pay \$2/month to maintain my registration. So this project is now coming out to about \$0.50 for messages, \$2 for the registration, and \$1.15 for the phone number, for a total of \$3.65/month. All so that I can have my iPhone go into sleep mode!! This is so dumb.

If I instead get a "Toll Free" number for \$2.15/month, there is a simpler registration process with no additional monthly fee, so it's more like \$2.65/month for this project instead. So I tried that. But they want a legal entity name, which I do not have. I'm just a person try'na get my phone to change focus!!

Screw this.

# Attempt 5: ??? TBD

I am currently out of ideas. If I figure something out, I will update this post. If you know of a solution, feel free to write to me: contact@mitchtalmadge.com

Thanks!