# com.news-agent.daily.plist

Runs `news-agent.js` daily at 8:00 AM via launchd. Sources `.zshrc` for `ANTHROPIC_API_KEY`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.news-agent.daily</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/zsh</string>
        <string>-c</string>
        <string>source ~/.zshrc &amp;&amp; /opt/homebrew/bin/node "/Users/liz1233/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian001/sync/Market/News/news agent 1.1/news-agent.js" &gt;&gt; "/Users/liz1233/Library/Mobile Documents/iCloud~md~obsidian/Documents/Obsidian001/sync/Market/News/news-agent.log" 2&gt;&amp;1</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>8</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/Users/liz1233/Library/Logs/news-agent.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/liz1233/Library/Logs/news-agent.err</string>
    <key>RunAtLoad</key>
    <false/>
</dict>
</plist>
```

## Management

```bash
launchctl list com.news-agent.daily                              # check status
launchctl unload ~/Library/LaunchAgents/com.news-agent.daily.plist   # stop
launchctl load ~/Library/LaunchAgents/com.news-agent.daily.plist     # restart
```

To change the time: edit `Hour` (0–23) and `Minute`, then unload and reload.
