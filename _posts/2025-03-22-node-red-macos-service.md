---
layout: post
title:  "Run Node-Red on macOS as a Service"
---

As part of [automating the sleep focus of my iPhone]({% post_url 2025-03-21-iphone-focus-automation-via-homeassistant %}), I needed a way to run [Node-Red](https://nodered.org/) on my Mac as a service so that it would run in the background and start up when I logged into my MacBook. I also wanted to make sure that it ran under my user account instead of root to improve security.

*Note: These instructions are to set up a service that starts on login, and shuts down on log-out. I only have one user on my MacBook, and never log out, so keep that in mind if you've got multiple users.*

# Node-Red Installation

I installed Node-Red on my MacBook using [these instructions](https://nodered.org/docs/getting-started/local), which I will summarize here for posterity:

*Note: I originally tried installing Node-Red and setting up a service with `brew`, but it never would start for me. It kept crashing and I don't know why. Either way, the official install instructions recommend using `npm`, so I gave up on `brew` and went with that.*

1. Install via NPM: `sudo npm install -g --unsafe-perm node-red`
2. Test it out; restart your shell and run `node-red`.
3. Go to http://127.0.0.1:1880/ and it should be running.

# Service Config

Here is how you can set up the service:

1. Create the file `~/Library/LaunchAgents/org.nodered.plist`, which is a [launchd](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html) configuration file. Here's what mine looks like (remember to replace `mitchtalmadge` with your username):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>org.nodered</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/node-red</string>
        <string>--userDir</string>
        <string>/Users/mitchtalmadge/.node-red</string>
    </array>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin</string>
    </dict>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>WorkingDirectory</key>
    <string>/Users/mitchtalmadge/.node-red</string>
    <key>StandardOutPath</key>
    <string>/Users/mitchtalmadge/.node-red/node-red.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/mitchtalmadge/.node-red/node-red-error.log</string>
</dict>
</plist>
```

2. Activate the service with `launchctl load ~/Library/LaunchAgents/org.nodered.plist`
3. Start the service with  `launchctl start org.nodered`.
4. Go to http://localhost:1880/ and make sure it's running.
5. Reboot and log back in to make sure it starts up automatically.

# Don't Go to Sleep!

You may want to configure your sleep settings to prevent your Mac from sleeping while plugged in, so that Node-Red does not go down. This is in System Preferences -> Battery -> Options:

![Showing that the "Prevent automatic sleeping on power adapter" option is turned on](/assets/images/2025-03-22-node-red-macos-service/1742706740765.png)

---

Node-Red should now run in the background on your Mac and start up when you log in. It will also run as your own user, not root, which I think is much safer. If you need to stop it, you can do so with `launchctl stop org.nodered`.