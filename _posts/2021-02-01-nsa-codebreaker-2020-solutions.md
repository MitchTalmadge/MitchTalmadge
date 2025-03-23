---
title: "NSA Codebreaker 2020 Solutions"
author: "Mitch Talmadge"
date: 2021-02-01T19:57:22.871-0700
last_modified_at: 2021-02-01T19:57:22.871-0700
tags: ["programming","hacking","ctf","capture-the-flag","cryptography"]
description: "I explain my methods to solving the 2020 NSA Codebreaker CTF"
image:
  path: /assets/2021-02-01-nsa-codebreaker-2020-solutions/0*09tAanAlUKxktlBU.JPG
---

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/0*09tAanAlUKxktlBU.JPG)

In this post I go through how I solved challenges 1 through 5 of the 2020 NSA Codebreaker CTF (Capture the Flag) . This is a cybersecurity challenge involving reverse engineering, cryptography, and many other technical skills.

In many cases, my solutions will be different from your own. Where possible, the task files are subtly changed for each user so that the resulting answers are slightly different. The premise is the same, though. :)
## Intro

This CTF takes place around a **fictitious** (read: fake) story about a kidnapping of a journalist. Here is the intro prompt from the challenge, if you are curious:

> Two days ago, a renowned American journalist went missing while on assignment abroad. Although the city where the journalist was last seen has very few surveillance cameras on its streets, local authorities were able to provide us with some surveillance footage taken near the journalist’s hotel during the days leading up to the disappearance. From the footage, we see that the journalist was kidnapped from the front of their hotel and taken away in an unmarked van. Unfortunately, we can also observe the kidnappers destroying the journalist’s mobile phone making it impossible to track their route via GPS. Locals have also reported drone activity in the area leading up to the event. A criminal organization, well known for using drones in their kidnap-for-ransom schemes, has claimed responsibility for the incident. 
> 
> Your mission is to: 
> 1. Locate the missing journalist and hostage takers’ current position.
> 2. Facilitate the recovery of the journalist and take actions to prevent another incident from happening. 
> 
> Each task in this year’s challenge will require a range of skills. We need you to call upon all of your technical expertise, your intuition, and your common sense to help us locate and rescue the journalist! 
> 
> Good luck. We hope you enjoy the challenge! 

Sounds fun! Let’s get started.
## Task 1: What’s On the Drive?
### 10 Points | Computer Forensics, Command Line, Encryption Tools

> In accordance with USSID18, a collection on an American citizen is permitted in cases where the person is reasonably believed to be held captive by a group engaged in international terrorism. As a result, we have obtained a copy of the home directory from the journalist’s laptop and are hoping it will contain information that will help us to locate and rescue the hostage. Your first task is to analyze the data and files available in the journalist’s home directory. 

#### Goals:
1. Find the journalist’s username.
2. Find the encrypted file in the journalist’s home directory.

#### Goal 1: Find the username

We are provided a file called `home.zip` which is an “archive of data from the journalist’s computer.” This is clearly a Linux-based home directory upon extraction. Here is the file structure, which I printed with the `tree` command:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*wOaZaOYLexLTsMiq5SRAxA.png)

In Linux, each standard user gets their own directory inside `home` , which is named by their username. Thus, the username of this journalist is `NapoleonDovetail374` .

#### Goal 2:** **Find the encrypted file

There’s a few ways you could go about this, but I just went the common-sense route. Which of the files in the above picture would you most likely want to encrypt? Probably the file named `pwfile` , which presumably contains a bunch of passwords. To verify that this is the encrypted file and determine how it is encrypted, we can use the `file` command.

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*wsxp6ItI7monMULx2B9hZw.png)

Yep, that’s the one!
## Task 2: Social Engineering
### 40 Points | Computer Forensics, Metadata Analysis, Encryption Tools

> The name of the encrypted file you found implies that it might contain credentials for the journalist’s various applications and accounts. Your next task is to find information on the journalist’s computer that will allow you to decrypt this file. 

#### Goal: Find the password to decrypt `pwfile`

Ok, this is a very clear goal. We just need to figure out how to decrypt this file. My very first intuition was to look at the `pwHints.txt` file. Who wouldn’t want some password hints? Let’s open it up:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*WMNMbbaRuOoUUixfKxTg3Q.png)

There’s lots of different hints in here, but the important one is the `keychain` hint. The terms “keychain” or “keyring” are pretty synonymous with a password storage these days, so it makes sense that this would go to our encrypted `pwfile` . We need to find the name of this person’s pet, and the pet’s birthday.

Let’s start in the `Pictures/Pets` directory. I found an image there called `shenanigans.jpg` which looks pretty promising:

![animals wearing party hats and having a birthday party](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*3TFEw92iZWK1rDOeszmIug.png)

just wow

Let’s assume that this was taken with a camcorder even though it’s clearly photoshopped (it’s a fictitious story, let’s cut them some slack) . In that case, assuming this picture was taken on this journalist’s pet’s birthday, we can hopefully get the capture date from the EXIF data of the image. Let’s run `exif` on the file:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*yEEr1JQ3F1t8ITzSgU7tmA.png)

Yep, there it is! This picture was taken on December 28th, 2019 (12/28/19) . We don’t know the birth year of this pet (which pet owners do, honestly?), so it’s probably a safe bet that the password only involves the month and day of the birthday; 12/28.

Now we need to find the pet’s name. Nothing in the file names or pictures revealed anything here, so I started reading their blog entries found in `Documents/Blog-Articles` . Check out the one called `blogIntro.txt` :

> Journalist by day. New Blogger by night. Traveler when able. Pet lover…always. 
> 
> Napoleon here. Welcome to the ‘Diary of an Exotic Furry Voyager’. Outside of work, my two favorite things are traveling the world and getting to come home to **my favorite furry little friend, and the best friend on the planet, Pearl.** This blog is going to account for some of my travels and more importantly, the animals I meet on the way. Hope you enjoy! 

Bingo! This journalist’s pet name is “Pearl.”

The password hint was `pet name + pet bday` , so let’s try `Pearl1228` :

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*KTqgr-zYU_2ILj7ZwnTeWQ.png)

Success! Opening `pwfile.dec` with `vim` reveals that this is an SQLite database. Task 2 complete.

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*3IY02HmtbDxG_c9rlPAk2A.png)

## Task 3: Social Engineering
### 150 Points | Computer Forensics, Metadata Analysis, Encryption Tools

> Good news — the decrypted key file includes the journalist’s password for the Stepinator app. A Stepinator is a wearable fitness device that tracks the number of steps a user walks. Tell us the associated username and password for that account. We might be able to use data from that account to track the journalist’s location! 

#### Goal: Find the username and password for the Stepinator account

Well this sounds like it’ll be easy now that the `pwfile` is decrypted. We know that it’s an SQLite database, so let’s use the `sqlite` command to do some queries:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*TSGCzJc7pGUAT_QZJ1WfLg.png)

We can see that there are two tables in this database. Let’s jump into the `passwords` table and see what we have:

![pro tip: use **.explain on** to add column headers to outputs](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*0iuRHAEHbcF2LK5--4DZ-A.png)

pro tip: use **.explain on** to add column headers to outputs

There’s lots of usernames and passwords in here, but it’s not clear which one is the Stepinator account. There is a `service` ID for each account, so let’s check the `services` table:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*avlX5CrebwuvvB214AYimA.png)

Awesome! Stepinator has `service` ID 8. Thus, our username and password appear to be `Pearl_Dovetail_1228` and `<~:gn0OB6%QpDGXnQEbmHb0JbC?0J`~>` … but hang on, notice how all of the passwords look a bit _odd?_ Based on the `pwHints.txt` file, we expect it to look something like `color + petName + anniversary + fdate` . I had never seen this format before, so I googled `<~ ~>` and found this helpful piece of information:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*tb5I7gPzYcxdBZVSc_kjxA.png)

So they’re just ascii85-encoded. Cool! I used [an online tool](https://cryptii.com/pipes/ascii85-encoding) to decode the password, and it turns out to be `PalegreenPearl11030503` . That’s much better — and it was the correct answer!
## Task 4: Follow That Car!
### 500 Points | Graph Algorithms, Computer Science

> By using the credentials in the decrypted file, we were able to download the journalist’s accelerometer data from their Stepinator device from the time of the kidnapping. Local officials have provided us with a city map and traffic light schedule. Using these along with the journalist’s accelerometer data, find the closest intersection to where the kidnappers took their hostage. 

#### Goal: Find the intersection where the kidnappers took the journalist

For this task we are provided 3 files:
- Relevant information for solving the problem ( `README.txt` )
- Acceleration data ( `stepinator.json` )
- City map and traffic light schedule ( `maps.zip` )

Here is the `README.txt` contents, if you are interested. Scroll past it for a TL;DR.

> README — Additional Instructions
> 
> The jounalist was wearing a fitness tracker. Although this device does not capture GPS data, it does capture acceleration data. Perhaps we can use this data to locate and save our victim! 
>
> Security cameras show where the the journalist was kidnapped and it has been marked on the included city map. The kidnappers then headed directly East. After that, we lost track of them. 
>
> We have obtained the journalist’s accelerometer data from their fitness tracker app thanks to your success in decrypting their password database. A list of lateral acceleration values (in m/s²) for each second starting at the time of the kidnapping can be found in the `stepinator.json` file. 
>
> Unfortunately, after about two and a half minutes, the tracker stopped collecting data. 
>
> We have provided a map of the city streets showing the kidnapping location. Each city block is about 100m long. This map is available as a shapefile as well as a .png image file. 
>
> Finally, we have obtained the traffic light information for the city. Each intersection has a light, and each light has a cycle of 30 seconds green, 30 seconds red. If a light is green for vehicles traveling through the intersection vertically, it will be red for those traveling horizontally (and vice versa) . The victim was kidnapped just as the lights changed color. Maps showing the traffic lights at certain time intervals are attached. 
>
> We suspect that the kidnappers obeyed all traffic laws and stayed close to the speed limits. Right turns on red lights are not allowed. The speed limit in the city is 13 m/s. The kidnappers likely slow down when they take a right or left turn. We also believe that the kidnappers would not loop back when driving away — you may assume that once an intersection was traversed, that intersection was not revisited. 
>
> Finally, since the journalist was kidnapped in the middle of the night, it is safe to assume that there were no other cars on the road. 
>
> Determine the closest intersection to where the journalist was located when their fitness tracker stopped recording data. 

**TL;DR:**
- We have the acceleration data for the moving vehicle because of the Stepinator.
- We know where the kidnapping took place.
- We have maps of all intersections and the schedule at which the lights change.
- We know that each city block is about 100m long.
- We know the vehicle started moving east right as the lights changed.
- We know that the speed limit is 13 m/s and that the kidnappers obeyed traffic laws.
- The kidnappers slow down to make a right or left turn, but do not turn on red.

Here are the maps:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*l_ZBlQxX2V1N1q9Y7C0sJQ.png)

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*85ZGlN1V0hCFCO9jPSTpUw.png)

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*rqRlfHfi89FkXXLRRoU9Zg.png)

And here is the acceleration data:
```
[1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 0, 0, 0, 0, 0, 0, 0, 0, -0.833, -4.5, -4.5, -3.667, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 0, 0, 0, 0, 0, 0, 0, 0, -0.833, -4.5, -4.5, -3.667, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 0, -0.846, -4, -4, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 0, -2.345, -4.5, -4.5, -2.309, 0, 0, 0, 0, 0, 0, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 1.5, 0, 0, 0, 0, 0, 0, 0, 0, 0, -3.308, -4, 1.5, 1.5, 1.5, 1.5, 1.5, 0, 0, -2.426, -4, 1.5, 1.5, 1.5, 1.5, 0, 0, 0, 0, 0, 0, 0, 0, 0, -0.266, -2.761, -4.5, -4.5, -1.239, 0]
```

How are we going to put this whole puzzle together? My first idea was to plot this acceleration data in Excel. Using some formulas I was able to derive the acceleration, velocity, and distance travelled at each second:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*HDBR6r0QguoV9UL3seNXiA.png)

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*OcCUI3e5PpznUhqfKYJErA.png)

There’s some pretty valuable information in these graphs.

First of all, with the distance graph we can be certain that a total of 11 blocks were travelled, since the data ends at 1100m from the start.

Second, the velocity graph shows the behavior of the car at every second. Look at the first “hump:”

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*_Py9sRj3X1Vgj3e8bFyMRQ.png)

Here we can see that the car speeds up, moves at a constant rate of about 13m/s, and then slows down to a stop. This is indicative of driving, then stopping for a red light.

But how do we know that they went straight, and did not turn? The answer is that we know that the car will slow down for a turn. In this case, the car goes a constant speed. A turn would look more like this:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*Ual_IiSFESAos3SDZs3Dyg.png)

Here, the car speeds up from a stop to 13m/s, then slows down to 5m/s, before quickly speeding back up to 13m/s, and then coming to a stop.

They sped up, slowed down to turn, then sped back up before running into a red light.

Now all that is left to do is open OneNote on my Surface, and start drawing a path that aligns with the data. This took a while, but I ended up with the following path, which turned out to be correct!

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*qa9sXf-mejUKsBqFE18uXw.png)

You might be wondering how I decided that they made a right or left turn. Luckily, at each turn, the opposite turn would have run me into a wall or caused other problems which made that turn impossible; so the choice was obvious at each step. However, even if you had only narrowed down the choices to three or so, you do get multiple chances to answer the question, so it’s no big deal.
## Task 5: Where Has the Drone Been?
### 1300 Points | Reverse Engineering, Cryptography

> A rescue team was deployed to the criminal safehouse identified by your efforts. The team encountered resistance but was able to seize the location without causalities. Unfortunately, all of the kidnappers escaped and the hostage was not found. The team did find two important pieces of technology left behind: the journalist’s Stepinator device, and a damaged surveillance drone. An analyst retrieved some encrypted logs as well as part of the drone’s GPS software. Your goal for this task is to identify the location of the criminal organization’s base of operations. 

This one was **TOUGH.** Something that bothers me when most people give their CTF solutions is that you only see the successful bits. You never hear about them struggling to find the answer. You never hear about how they accidentally stay up all night working on the problem, throwing off their sleep schedule, and missing classes as a result, which only added to their depression and anxiety. You never hear about how the problem obsessed them and caused them to neglect their college assignments. You never hear about the stress and the imposter syndrome, many times wondering “am I good enough to do this?” I am here to tell you that I experienced all of this and more. Problems like these can be a huge undertaking, and it was NOT easy to solve this one. However, I learned so much from it, and I’d do it again in a heartbeat. Be strong; power through; it’s worth it.

Now — onto the problem. We are given two downloads:
- Interesting looking binary ( `gpslogger` )
- Recovered log files ( `logs.tgz` )

This includes four encrypted log files from the drone, and the software that produced them:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*VecDG2a4l2FpybjlVMB-gw.png)

Let’s run `file` on one of the log files and see what it knows:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*xW2JhYz-TlafdQoHcoBeXw.png)

Oh.. “data” ..that’s not useful. This is definitely a type of encryption that does not include metadata. Probably something simple like AES. Let’s also run `file` on the binary:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*3idkxhJGl1i6Oulb-XW_tw.png)

This is an aarch64 ELF executable. In other words, it was compiled for a 64 bit ARM processor, which I wouldn’t be surprised to find inside a drone. This is a similar processor to what powers the Raspberry Pi 4, or your smart phone — low on battery consumption with decent performance.

I tried many ideas to crack this encryption, and I will go through some of them here. First, I like to run `strings` to see what hardcoded information exists in the binary. If we are really lucky, the encryption key would be hardcoded. Here’s some of the things I found.
- These function names reveal that we are looking for a key and IV which are generated inside the code. `setup_cipher` also looks useful.

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*-bfvErwYDVF0NiK7cgIzng.png)

- These function names suggest that we will be working with AES CBC, ASM, or GCM encryption.. I wonder which.

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*r6lG6LHGNxonv9Mpi_n9oA.png)

- Ooh! Look at this one. The IV depends on some longitude coordinate, and the key depends on some latitude coordinate.

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*2r-6QsbD0rHTWONQXLqEGA.png)

That last one really got me thinking… where have I seen latitude and longitude coordinates before? Oh yeah, the map from Task 4!

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*rMkgf0elsvwArdCl443kTw.png)

I bet that will be useful later.

Okay, at this point I have some information but definitely not enough. I know that those coordinates play into the key and IV somehow, but I don’t know the exact details.

Let’s see if we can run the binary. Since it’s ARM64 based, I will need a device with this kind of processor. Remember when I mentioned phones? I have a spare OnePlus 6 Android phone laying around, so I installed [Termux](https://play.google.com/store/apps/details?id=com.termux) as well as [Andronix](https://play.google.com/store/apps/details?id=studio.com.techriz.andronix) , two apps which allow me to install and run Kali linux on my phone, even without root. I set up an SSH server and then remoted in from my desktop. After uploading the binary, I tried to run it:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*4v-7rXEFlvuEYTMt-jwCsQ.png)

Hmm.. missing libraries and symbols. Let’s see:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*SOByBlOV567CqEZAIzlFTA.png)

Ok, we’re missing `libgps.so` and `libc.musl-aarch64.so.1` . After installing these missing libraries, I still could not run the binary because of the `GPSNMEA: symbol not found` error. I think this binary won’t work unless I have an actual hardware GPS module hooked up, and I do not have one, and don’t know how to use the one my phone has built-in. I gave up on running the binary at this point.

Let’s finally disassemble the binary and look inside, something I was sort-of hoping to avoid. I used the command `objdump -D gpslogger > gpslogger.asm` . I opened the file and jumped to `main.main` which I knew about from `strings` :

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*Ba1FYVf1AzW8VvSc4vSxPQ.png)

Hey, do you see that? `<go.string.*+0x6fe0>` … This binary was written in golang! This is actually going to end up being obnoxious because golang does a lot of interesting things which make the assembly look fairly different from traditional c-lang type binaries, like c and c++ . Here’s some of the things I learned while I read through the assembly and scoured Google:
- golang passes all parameters into functions through the stack. They do not use registers for this, unlike c/c++ . ( [This will change in golang 1.16 or 1.17](https://github.com/golang/go/issues/40724) )
- golang functions can return multiple values.
- golang functions return their values on the stack, not through registers. They do this by placing the return values after all the parameters. So if you pass parameters into a function at `SP + 0x08` and `SP + 0x10` , then expect return values starting at `SP + 0x18` .
- Strings are returned as two words; a pointer and a length.

_I could go on forever, but instead [here’s a great page](https://dr-knz.net/go-calling-convention-x86-64.html) explaining more if you want to learn._

I decided to go take a look at the `main.generate_key` and `main.generate_iv` functions.

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*tQ0c7gcujHuEYkIuBuwQRw.png)

There’s a lot going on here. I can see something which I bet will be valuable, though: `strings.Repeat` . Let me narrow things down for you:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*zY9U_qjHrkinDTJlI27NuQ.png)

Here we store to `SP + 8` , `SP + 16` , and `SP + 24` . Then the function is called and we load from `SP + 32` and `SP + 40` . Remember what I was talking about with return values coming after the parameters? 32 comes directly after 24. So it looks like we pass in 3 parameters and get 2 out. Let’s look at the method signature for `strings.Repeat` :

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*ML5Bd9oqKJRcS8xHM8zQ4A.png)

[https://golang.org/pkg/strings/#Repeat](https://golang.org/pkg/strings/#Repeat)

So it takes in a string, and a count of how many times to repeat it. It returns a string. Remember also what I said about strings taking up two slots: a pointer and a length? So `SP + 8` is the input string pointer, `SP + 16` is the input string length, and `SP + 24` is the count.

We can see in the screenshot that `orr x0, xzr, #0x4` will cause `x0` to contain `0x04` , or simply `4` . `x0` is then stored in the count parameter position. So this string will repeat 4 times.

The return value is the repeated string; the pointer and the length. These are immediately returned from our `main.generate_key` function.

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*5ebEP5LcNQYk7XHTg33Rug.png)

`main.generate_iv` works very similar, except that we repeat 3 times, and then we load `runtime.rodata+0x3c7a0` , which turns out to be a single `0` character. This is appended onto the end of our repeated string via the `concatstring2` function call. I am guessing this is just a single character of padding, since IVs need to be the exact length of one plaintext block.

Where are `main.generate_key` and `main.generate_iv` called from? The answer is `main.setup_cipher` .

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*pN98SgGZNHrrIklzXb2_gQ.png)

Here I have jumped right into the middle of it, but you can see that it is calling both functions in succession.

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*iZRIzIrFPU1BKHYdM5veBA.png)

Still in `main.setup_cipher` , there are two very important function calls. One to `crypto/aes.NewCipher` and one to `crypto/cipher.NewCBCEncrypter` . This tells us that we are using AES CBC encryption! Now that we know which cipher and mode are being used, we can also infer some things:
- AES-128 requires blocks to be 16 bytes in length, AES-192 requires 24, and AES-256 requires 32. Our key has to match this length, which means that the string we repeat 4 times must be of length 4, 6, or 8 bytes. We don’t know yet whether 128, 192, or 256 is being used.
- CBC requires that the IV be the same length as one block. This means our IV has to be either 16, 24, or 32 bytes in length. To figure out which one we are using, let’s consider the equation `3x + 1 = y` . Here, `x` represents the length of our input string that is multiplied by 3. `1` is for our single added `0` char padding. `y` is either 16, 24, or 32. Also, `x` must be an integer. The only way to solve this equation is with `x = 5, y = 16` . Therefore, we know that our block size is 16 bytes, so we are using AES-128 with a 16 byte key and a 16 byte IV.
- To summarize, our key input must be 4 bytes long, and our IV input must be 5 bytes long.

This is really good information. All that’s left to do is figure out where these inputs come from. I already have a guess; remember the strings from the binary which mention that the key is based on latitude, and IV on longitude? Let’s keep that in mind.

Something I discovered while looking through `setup_cipher` is that it will branch to a panic if certain conditions fail. The panic message is quite valuable:

> !!! EXPECTED TO FIND NMEA $GNGGA HEADER !!! 

Wow, what a message. So much excitement in that error. It reveals something super useful to us, though! This `setup_cipher` function takes in a `$GNGGA` NMEA header. This is raw information that GPS modules receive from satellites. Here’s an example of one of these headers:
```
$GNGGA,054157.013,2307.1261,N,12016.4308,E,1,6,1.93,34.9,M,17.8,M,,*76
```

And here’s the formatting:

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*6YJXvAn6U5UtyJcC7hdXSQ.png)

Truncated a bit. Source: [https://gpsd.gitlab.io/gpsd/NMEA.html#_gga_global_positioning_system_fix_data](https://gpsd.gitlab.io/gpsd/NMEA.html#_gga_global_positioning_system_fix_data)

Look! Latitude and longitude. Look what else! 4 latitude characters (key) and 5 longitude characters (IV) . If you dig into the `main.setup_cipher` code a bit, you will find that it splits on `,` and `.` characters in a couple places. This allows it to extract the first 4 characters of the latitude and the first 5 characters of the longitude (stripping off the decimals) .

Keeping with the theme, I wrote a golang program using the same libraries in order to decrypt the log files. I plugged in `0513` as the latitude and `02459` as the longitude inputs, which come from the map image from many paragraphs ago. I then tried decrypting each log file. The newest two according to the dates in the file names were successfully decrypted!

![](/assets/images/2021-02-01-nsa-codebreaker-2020-solutions/1*R48b21LQozkOxJnjLCDk_w.png)

The very first block was messed up ( `$EHFKH` is not a valid NMEA header) and I guess that’s because my IV was incorrect somehow. I never took the time to figure out the real IV because it didn’t matter; all the rest of the data was correct and had the info I needed.

Let’s remember our goal: find the coordinates of the kidnappers’ headquarters. If I had to guess, I’d say that these two newer files are from when the drones were flying around looking for our journalist before the kidnapping. The two older files must be from flights taken at the headquarters.

I suppose that I could try graphing the NMEA files and see if that leads me to their coordinates, but I had an easier idea: bruteforce! It’s only a 4 character key and I don’t need the IV, so I just guessed all values from `0000` to `9999` and checked if the output looked right. Here’s what I wrote:

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"fmt"
	"io/ioutil"
	"strings"
)

const nmeaLat = "0521"
const nmeaLong = "02459"
const logName = "logs/20200628_153027.log"

func main() {
	decryptKnown()
	//bruteforce()
}

func decryptKnown() {
	ciphertext, err := ioutil.ReadFile(logName)
	if err != nil {
		panic(err)
	}
	plaintext := make([]byte, len(ciphertext), len(ciphertext))

	key := strings.Repeat(nmeaLat, 4)
	IV := strings.Repeat(nmeaLong, 3) + "0"
	block, err := aes.NewCipher([]byte(key))
	if err != nil {
		panic(err)
	}

	cbc := cipher.NewCBCDecrypter(block, []byte(IV))
	cbc.CryptBlocks(plaintext, ciphertext)

	err = ioutil.WriteFile(logName+".dec", plaintext, 0644)
	if err != nil {
		panic(err)
	}
}

func bruteforce() {
	ciphertext, err := ioutil.ReadFile(logName)
	if err != nil {
		panic(err)
	}
	plaintext := make([]byte, len(ciphertext), len(ciphertext))

	for i := 0; i < 9999; i++ {
		keyInput := fmt.Sprintf("%04d", i)
		key := strings.Repeat(keyInput, 4)
		IV := strings.Repeat("00000", 3) + "0"
		block, err := aes.NewCipher([]byte(key))
		if err != nil {
			panic(err)
		}

		cbc := cipher.NewCBCDecrypter(block, []byte(IV))
		cbc.CryptBlocks(plaintext, ciphertext)

		if string(plaintext[28:31]) == ",N," {
			fmt.Printf("Found key: %s\n", keyInput)
			return
		}
	}

}
```


In less than a second, I had the correct key to the older log files: 0521. Not far off from our original 0513! They must be nearby.

I opened the decrypted log file and obtained these decimal coordinates:

```
0521.073024 N 02418.706995 W
```

So now we had our answer; the CTF asks for the answer in the format `##°##'N ##°##'W`, so my answer was `05°21'N 24°18'W`. Auē, that was a lot of work.

---

## Tasks 6-9

I ran out of time for these, but there's lots of solutions on Google. I hope you enjoyed reading my writeup! If you have any questions, feel free to ask. I'm always happy to help.