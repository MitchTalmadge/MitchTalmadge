---
layout: post
title: "Tough Luck: Using Network Traffic Analysis to Expose a Rigged Coupon Wheel-of-Fortune"
author: "Mitch Talmadge"
date: 2018-05-29T20:45:13.530-0600
last_modified_at: 2020-03-31T20:45:33.752-0600
categories: ["Mitch Talmadge"]
tags: ["marketing","hacking","psychology","emotions","networking"]
description: "I dig into the network traffic behind a Shopify app called \"Wheelio\" that tricks users into believing they won a coupon by luck."
image:
  path: /assets/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*bk5y0ntuO-YtWltyBe4uRw.png
---

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/0*05Unn77opyN0qkWl.png)

While doing productive work on the internet like [watching cats slap each other](https://reddit.com/r/CatSlaps) , I happened upon a company called [_Hush Blankets_](https://hushblankets.com/) that creates weighted blankets to help with sleep and anxiety. What an awesome idea! I spent a couple minutes browsing through the website when — all of a sudden — a giant wheel of fortune came popping out of the left side of the screen.

> “You unlocked a **special bonus,” it proclaimed!** 

Gasp! The wheel was covered in coupons promising $10, $15, $20, $30, OFF— or even a FREE blanket! Considering that these blankets go for nearly $200 each, a free blanket would be pretty darn cool. All I had to do was enter my email and press the button! My mom always told me I was special. 😋 _(I’m not sure who this “Ed” guy is though. )_

“Okay,” I thought, “I have nothing to lose.” So, naturally, I entered a fake email address and hit enter. The wheel instantly came to life! It spun slower and slower until it began to get **STUCK _RIgHt ON tHe EdGE_** of “Almost” and “$10 OFF"!!…

gasp!!! **I WON!**

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*FWyh1ZaslsdekoazknsKGg.png)

**Or did I?**

As both a software engineer and an entrepreneur, I know two things very well:
1. Randomness (so called “luck” 🙄) and computers don’t mix.
2. No company would ever give away a $200 product for free just because some lucky bastard clicked a button. Not if they can help it.

Needless to say, I was skeptical. I was also intrigued. I wanted to know how the wheel _really_ worked. Was the prize predetermined? Could you _really super truly_ win a FREE blanket?! I started digging.

Betting on my intuition that a blanket company wouldn’t code up a custom wheel of fortune just for coupons, I hopped right into the Chrome Debugger’s “Sources” section. This allows me to see all of the scripts currently loaded on the page. Glancing through the URLs, I quickly noticed one with the word “wheelio.” Hmm… wheelio… “wheel”… yeah, that seems like a good place to start.

![wheeeeelio](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*F7Ee_kjhnfW14XIFHpTNtg.png)

wheeeeelio

Without even looking at the source code, I punched “Wheelio” into Google, and… oh. That was easy.

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*5e1l1Ys3KO_KremXCwTZUg.png)

**“They spin, you win.”** That tagline doesn’t sit well with me. Feels kinda disturbing. Let’s dig deeper. Here’s an explanation on the website of why you should be using Wheelio:

> Purchase behavior psychology is a strong thing… 

> People don’t really like easy free stuff. Not when it comes to coupons and usage of coupons. They need to feel like they “won” and the coupon was hard earned. When they feel like they have the upper hand, the usage of coupons is 10X higher than as of an ordinary coupon. 

**TL;DR:** You click the button, it makes you feel like you’ve won, so you’re more likely to use the coupon.

_Why does this feel emotionally manipulative to me?_ Back to that in a second. Let’s keep going, because I’m really eager to get my hands on a free blanket.

I was curious if the prize was determined in my web browser or on a remote server somewhere. Could I trick my browser into giving me any prize on the wheel? I figured that the best way to find out would be to monitor the network traffic. I refreshed the page and the data came flowing in:

![Ooh, pretty colors](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*bk5y0ntuO-YtWltyBe4uRw.png)

Ooh, pretty colors

I filtered down the results to only those that included “wheelio”:

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*57ff-FAfsgPeY92hSTABpg.png)

Oh! A websocket? This means that the server and browser are having an ongoing conversation. Let’s listen in. 😈

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*IyxmabMu8SqotciVSoZlVQ.png)

Here’s what the server and browser are saying to each other. Most of these frames (messages) look pretty useless to me, but one of them is being cut off. Judging by the length, it probably contains some  _juicy details._ A little copying there, some formatting here, and…

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*Rzrz_mxbgqGQ9-eiLhFLIQ.png)

Check it out! This must be the information that is used to populate the wheel with all the labels, like “$10 OFF Coupon”. Not only can you see the labels, but you can see the **coupon code** and the **“gravity”** (i.e. probability) for each coupon.

Now about that free blanket… The gravity for the free blanket coupon is 0! _That means you can never land on it!_ In fact, this is true of the $30 and $20 coupons too!

I even tried checking out with their respective codes: `000-000-000` , `30OFF` , and `WELCOMEBACK` , but only the $20 coupon worked.

What really irks me is that they outright say “You have a chance to win up to $30 OFF.” This is a blatant lie. Not only this, but there is a progress bar on the popup which says “12 coupons left. Hurry up!”, but this number is hard-coded into the popup! It says 12 for everyone, forever! Another lie. 😡

So, back to the whole emotional manipulation thing. I fail to understand how a CEO could sleep at night knowing that they are purposely toying with their customers’ emotions by using these kinds of tactics. Not to mention that the target market for Hush Blankets may already be struggling with their emotions to begin with. A wheel of fortune is supposed to have near-equal probability for all landing spaces. Wheelio has intentionally designed this app to be rigged, and they boast about it!

Hush Blanket isn’t the only company doing this, either. Take a stroll through the [over 350 reviews on the Shopify app page](https://apps.shopify.com/wheelio-first-interactive-exit-intent-pop-up#reviews-heading) and you’ll find many many more companies using this tactic. Companies like [_Hey Casey!_](https://www.heycasey.co.za) which lists a 20% off coupon with 0 gravity, or [_Wavy Prints_](https://wavyprints.com) which has a free print coupon with 0 gravity. Interestingly, whether by accident or not, _Wavy Prints_ did not actually disable their free print coupon, so I took the liberty of ordering their largest print of **Kanye Crossing the Alps:**

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*jyMelakbUqTKVFZ3umDkGA.png)

Two days later, Wavy Prints took their website offline, presumably because they thought they were hacked.

Karma is a bitch.
### Update Mar. 31, 2020: Privy Enters the Arena

Let’s shame another company! Today one of my readers came across a clothing company called [Pearlfeet](http://pearlfeet.com) which uses a very similar coupon wheel. This one has options like “ **100% Off** ,” “ **Free shipping** ,” and more! However, what I quickly found out is that this wheel is rigged just like the rest. This wheel is not hosted by Wheelio like the other sites, instead it uses a new service called [Privy](https://www.privy.com/spin-to-win-examples) , which also works on Shopify sites.

In this case, you enter your email to get spammed by their junk newsletter, and then the wheel always lands on the “ **10% Off** ” coupon. Check it out:

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*fASla9qnBuiAyuvVqwHjxw.gif)

Once it’s done spinning, a “ **Thanks!** ” dialog box pops up with the code “ **PF10** ”:

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*HCn7DrnFGX1AJbNpAaPZbg.png)

I did some quick digging to see what was going on here. First of all, while I cannot view the coupon codes of the wheel since the developers of Privy were smarter than Wheelio and decided to keep that information server-sided rather than download it all into the client’s browser, I did notice that the returned prize had a win ratio of `1` (signifying a 100% probability that it will land on the 10% off coupon… typical) and even more interesting, the coupon code is “ **EXAMPLE** ,” _not_ “ **PF10** ” like the picture shows.

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*Jp1A0-8ZA9TuV5SKZbpKlg.png)

So what does this mean? It probably means that none of the other coupons exist, and in fact I would bet that every slice of the wheel is filled in with “ **EXAMPLE** .” It also means that “ **PF10** ” is hard-coded somewhere. I looked back at the metadata for the “campaign,” which is downloaded into the browser before you even get a chance to see the wheel, and this is what I saw:

![](/assets/images/2018-05-29-how-companies-like-hush-blankets-are-taking-advantage-of-your-emotions-for-profit/1*t-xEdvKsp0pHiZP8TqTXSA.png)

What you’re seeing here is that the “Thanks!” dialog box from before has the code “ **PF10** ” hard-coded into it!

This is why the wheel has no actual coupon codes; because the code you get does not depend on the wheel at all. The wheel is just a fancy way to get you to enter your email and forever be bombarded with digital envelopes full of garbage. Once that email is submitted and you’ve been entertained by the wheel, the 10% off coupon dialog box is displayed so that you can feel like you’ve won and need to use the coupon.

Very manipulative.

