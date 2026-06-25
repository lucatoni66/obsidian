---
title: "I Built a FREE TradingView Alternative Using AI (Any Indicator)"
source: "https://www.youtube.com/watch?v=lilVaCcfzJY"
author:
  - "[[Power of Trading]]"
published: 2026-05-24
created: 2026-06-22
description: "I built my own free, multi-chart trading dashboard using AI — and it replaces the expensive charting subscriptions most traders pay $40+ a month for. Live da..."
tags:
  - "clippings forex"
---
![](https://www.youtube.com/watch?v=lilVaCcfzJY)

## Transcript

**0:00** · This setup costs almost $500 a year. I rebuilt the whole thing in an afternoon for free. And by the end of this video, you'll have it, too, running on your own laptop. Because here's the deal with most charting platforms: one chart at a time, two indicators. The good stuff, fair value gap, volume profile, locked, pay up, or stay limited. But you're looking at six charts, eight if I want, live data, every indicator I need, stocks and crypto side by side, $0.

**0:29** · And the best part, you can add any indicator in the world, anything you've ever seen.

**0:34** · Bring it in, drop it on.

**0:37** · Now, here's what nobody tells you. The chart engine, the live data, the layout, it's all open source, all free. The only thing those platforms charge you for is wiring it together. And I didn't write one line of that wiring. And AI did, a tool called Antigravity. A few prompts, that's it. Did it work first try? No, it crashed. Screen went black more than once, but I'll show you every mistake fix, so yours doesn't. By the end, you'll have your own version. Any market, any broker, any indicator. No subscription, no code, just you and a few prompts. Let's build it.

**1:09** · All right, let's actually build this thing.

### Setting Up the Project with AI (Project IDX)

**1:14** · So, I'm in Antigravity here, and the first thing I do is make in a new folder. I'll call it trading chart.

**1:20** · Simple.

**1:22** · Open it up, and this is where it gets interesting. I'm going to paste in one prompt, just one, and I want you to watch what happens here.

**1:30** · That's it. That's the whole instruction.

### Writing the Perfect AI Prompt for Trading Charts

**1:32** · Now, let me tell you what I just asked it, cuz this little prompt is doing a lot. I told it, "Build me a multi-chart trading dashboard, the kind of setup you'd normally pay a monthly subscription for.

**1:43** · Pull live data from Alpaca, that's a free brokerage account, so US stocks and crypto streaming in real time. And for the indicators, hook into a library called Pandas Ta, over 150 built-in indicators, all free. and look, it's already moving. It's writing the files by itself.

**2:02** · Um back end, the data layer, all of it.

**2:05** · I'm not touching a thing. Now, it'll start asking for permission. Uh allow running the command.

**2:10** · You just hit submit, and it'll ask again and again. Don't overthink it, just keep approving. Let it work. And there it is.

**2:16** · The verification runs, and it prints the two words I was waiting for. Set up okay, every package installed, clean, stage one done. But here's the thing.

**2:25** · Right now, there's nothing on screen yet, no charts, no data, just a skeleton. So, the real question is, can this thing actually pull live market data?

### Connecting FREE Real-Time Market Data

**2:35** · Let's find out. Okay, so before the next prompt, we need data. And for that, we're using Alpaca.

**2:41** · Just head to Alpaca, create a free account, log in like you would anywhere else.

**2:46** · And once you're in, go to the demo account, the paper trading one, and you'll find two things.

**2:51** · An API key and a secret key.

**2:55** · Copy both.

**2:56** · Now, these are your own private keys, so I'm keeping mine off screen. You'll just drop yours straight into the prompt, right here.

**3:02** · Key in one spot, secret in the other, and that's the prompt we take into antigravity.

**3:08** · And again, it just starts. Files building, files updating, the whole data layer coming together on its own. It'll ask permission, allow the command, Python data source, submit.

**3:19** · Couple more come up. Approve those, too.

**3:22** · Just keep it moving.

**3:24** · And watch this. This is the part I actually cared about. It's pulling real data, stocks and crypto, both of them, live. And it's formatting everything perfectly for the chart engine, the exact shape it needs, down to the timestamps. It even tells me stage two ready. What's next? So, the data's flowing, but it's flowing into nothing right now. It's just sitting in the back end. We need a bridge, something that takes this data and actually serves it up to a webpage.

### Building a Live Backend Server Like TradingView

**3:51** · So, next prompt. This one builds the Flask back end. In plain English, it's the little server that connects our data to the screen.

**4:00** · It sets up the routes, the symbols, the live prices, all of it.

**4:05** · Same thing. Files rebuilding, new files appearing, permissions popping up.

**4:10** · Approve them as they come. This one takes a little longer, so give it a second.

**4:15** · And there.

**4:17** · The server's live. It's connecting to our data source, and it's handing back clean formatted data through every route I asked for. So, now I've got live data and a server feeding it out. Like, which means we're finally ready for the part you actually came here to see.

**4:31** · Charts on the screen. Let's go.

### Creating a Professional 4-Chart Dashboard UI

**4:33** · Now, the third prompt.

**4:35** · And this is the one that brings it to life. I'm telling it build the full front end. Get four charts on the screen at once, the multi-chart dashboard, the layout, all of it. No indicators yet, just the charts. Three files, the page, the style, the chart logic.

**4:50** · And then, it works.

**4:52** · Quietly. For a good 2 minutes, it's just building the UI, the files, the layout, everything coming together. It asks for a couple of commands, I approve them, it processes for a bit longer, and then it hands me a link, localhost port 5000.

**5:06** · So, I copy it, open my browser, paste it in, and there it is. Four charts all at once, Apple, Tesla, Bitcoin, Nvidia, Nvidia, live on one screen, the exact thing people pay monthly for. And just to be sure it's real, I switch Bitcoin to a different time frame, and it reloads perfectly. No lag, no glitch, it just works. I'll be honest, this is the moment it clicked for me. One prompt, and I've got a multi-chart dashboard running on my own machine for free. But, there's still something missing.

**5:35** · Look closely, the charts are there, but they're frozen. The candles aren't moving, there's no live heartbeat yet.

**5:43** · And fixing that, that turned into the hardest part of this whole build. Let me show you.

### Real-Time Crypto Streaming Without Lag

**5:49** · So now I want the charts to actually move, tick live like the real thing.

**5:52** · That's prompt prompt for add real-time streaming, pull the live feed from Alpaca, and keep the keys safe on the back end where nobody can see them.

**6:01** · It runs. New files, updates, the usual.

**6:04** · Three, four minutes go by. It asks for permissions, I approve them, and it finishes. So I go back to my browser, refresh, and black.

**6:13** · Everything's gone. All processing done.

**6:17** · And I go back to the browser one more time.

**6:19** · And there it is. Bitcoin. Live, ticking, moving on its own. After all that, it's finally breathing. But And of course there's a but. It wasn't smooth. The candle was only updating every 12 seconds or so. Choppy. Because crypto trades on this feed just don't come in fast enough, and I wanted it smooth, like actually smooth.

### Fixing AI Errors & Token Limit Problems

**6:39** · TradingView smooth.

**6:41** · So one more prompt. Pull crypto from a faster live source, subscribe to Bitcoin, Ethereum, Solana, and let it stream properly. Enter.

**6:49** · Permissions.

**6:50** · Wait for it to finish, back to the browser.

**6:53** · And now, now it's beautiful. The candle's moving in real-time, up, down, ticking every half second. The price is live. It's exactly what I was chasing.

**7:02** · And honestly, getting here was the hardest part of the entire build, but it's done. The charts are alive, which means it's finally time for the part that makes this actually useful.

**7:12** · Indicators. Okay, charts are live.

**7:15** · Now let's make this thing actually useful. Indicators. So I send the prompt. Build a proper TradingView style indicator UI for every chart. The back-end math, the popover, EMA, Bollinger Bands, RSI, MACD, and focus on making the experience clean.

**7:31** · And instead of just charging in, it does something I liked. It hands me a full plan first. How it'll add the oscillators, how the back end calculates everything, how it all fits together.

### Making Charts Smooth & Ultra Fast

**7:41** · So I read it and it's solid. I tell it one important thing, though.

**7:47** · Draw the indicators from the historical data and don't recalculate them on every single tick.

**7:52** · Because if it tries to recompute everything every half second, it'll choke and I'll lose that smooth live candle I fought so hard for. Protect the smoothness, that's the rule. It agrees, starts building.

**8:03** · I approve the permissions, it creates everything and I open the dashboard, approve, retry again, but you push through it and eventually it pulls together. The button's there, then one more push, expand the indicators. I tell it, switch over to a bigger library, Pandas-ta, and load in around 35 popular indicators organized by category. It works through it and then yeah, my quota runs out again, second time. So, same move as before, swap to another account, fresh email, and we're back. No drama.

### Adding RSI, MACD & EMA Indicators with AI

**8:35** · And now it's finally ready. So, I open the browser and there it is, the indicator section sitting right where I wanted it. I open the Bitcoin chart.

**8:42** · Okay, so now the indicator panel's full.

**8:44** · RSI, Stochastic, MACD, all the standard ones are right here and you can drop them on any chart, any market.

**8:51** · That alone already replaces what most people pay for.

**8:55** · But that's not the part I'm excited about, because here's what I really want to show you.

**8:58** · With this setup, you can add literally any indicator in the world.

**9:04** · I mean it.

**9:05** · Any open-source indicator you've ever seen anywhere, you can pull it in, put it on your chart and run it across as many charts as you want, uh different markets, different time frames, all at once. Let me actually show you how.

### Fixing Black Screen Errors & Boosting Performance

**9:18** · Um so, I'll pick one.

**9:20** · There's this open-source indicator I genuinely love. It's called Swing Arm ATR. Think of it as a smarter, more advanced version of Supertrend. It's open, the source code's right there. So, I just copy it, I make a folder, drop the source code into a text file, done. And here's a little trick to make this work even better. I also grab two or three screenshots of the indicator just so the AI can see exactly what it's supposed to look like. The code tells it the math, the pictures tell it the look. So, I take the code, the images, and one short prompt.

**9:50** · And the prompt is basically just here's the source code, here's what it looks like, build me this exact indicator into my dashboard.

**9:58** · That's it.

**9:59** · That's the whole instruction. Hit enter and it goes to work.

**10:05** · And look at this.

**10:07** · There it is, the same indicator running on my chart. Same lines, same behavior, same to same. Now, the colors might come out slightly different, but it works exactly the same. And if you want, you just tell it match these colors, clean up the look, and it'll fix that, too.

**10:22** · But the indicator itself, perfect. And now, watch what this unlocks. I could open all eight charts at once and slap this same indicator on every single one. Eight markets, eight time frames, one screen, all running the indicator I love. That's a setup serious traders pay real money for, and I just built it for free. But here's the thing I really want you to hear. I only showed you one, one indicator. The whole world is open to you now. Every open-source indicator out there, it's yours.

**10:50** · You bring it, you drop it in, and it just works.

### Add Any Open-Source Indicator in Seconds

**10:56** · I showed you the door. What you build behind it, that's up to you. This Now, let me show you something cuz I didn't build this just to show off. I built it for myself. This is what I actually trade on.

**11:06** · So, let me walk you through how I really use it.

**11:08** · First, the layout. If I want two charts, I click two. Boom, two charts. Want all eight? Click eight.

**11:17** · There they are. But for my actual strategy, I run six, three on top, three on the bottom. So, I hit six. Now, me, I trade crypto and mostly Bitcoin. So, the first thing I do is load Bitcoin onto every single chart, all six.

**11:31** · Quick.

**11:32** · All six there, Bitcoin everywhere. Then I set my time frames, and this is the whole point of a multi-chart setup. I get to see every time frame at once. Top row, 5-minute, \[snorts\] 15-minute, 1-hour. Bottom row, same thing. 5, 15, 1-hour.

**11:45** · Now, the indicators. On the top three, I add my first one, Supertrend. There it is on the first, the second, and the third. All three, Supertrend up top. And on the bottom three, I add MACD. 1 2 3.

**11:59** · Done. So, now every top chart has Supertrend, every bottom chart has MACD.

**12:04** · And this This is my actual TradingView, every time frame side by side in one glance. So, here's how I read it. Say I'm looking for a buy on the 5-minute watch. According to this setup, the 5-minute already gave me the buy signal early. The MACD lines up, the 15 confirms, the 1-hour confirms, and once all of it agrees, that's my entry on the 5-minute. That's it. That's a real multi-time frame trading setup, the kind people build entire paid platforms around. And you just watch me put it together with free tools uh on my own machine.

**12:36** · And the best part, this is just my setup. Yours can be anything.

**12:40** · Any market, any indicators, any tiny time frames, whatever fits how you trade. No subscriptions, no fees, nothing. You're in full control. I've shown you the entire process step by step, nothing hidden. Now, it's your turn to build yours. So, you want this exact setup? Cool. Here's the deal.

### My Personal Multi-Timeframe Trading Strategy

**12:58** · Three quick things and it's yours.

**13:00** · One, tap the hype button right by the comments. I just need 50 people. 50.

**13:06** · Takes one tap. Two, like the video. I literally just saved you money. That like is the least you can do, and it tells me to keep building this stuff.

**13:13** · Three, drop a comment. Just two words, build it. That's it. And tell me what you're stuck on in your trading. I read every one, and I'll build it for you.

**13:22** · Free. You ask, I deliver. Now, the rewards. Hit 1,000 likes, I drop every single prompt I used.

**13:30** · The whole thing.

**13:31** · 3,000 likes, you won't build anything at all. I'll put the entire project on GitHub. You just drop the link into Anagravity, say build me this, and 2 minutes later, it's done. The exact setup you just watched. That's the trade. 50 hypes, 1,000 likes. It's nothing for what you're getting. For the link and everything else.

**13:51** · Join my WhatsApp channel down in the description.

**13:55** · Quick heads-up, this is for learning and building your own tools, not financial advice. How you trade is on you. All right, you know what to