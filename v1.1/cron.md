The pipeline will run every day at 8:00 AM, sourcing your .zshrc for the API key and writing the briefing to your vault.

  

  **To manage it:**

  launchctl list com.news-agent.daily   # check status

  launchctl unload ~/Library/LaunchAgents/com.news-agent.daily.plist  # stop

  launchctl load ~/Library/LaunchAgents/com.news-agent.daily.plist    # restart

  **To change the time,** edit ~/Library/LaunchAgents/com.news-agent.daily.plist — change Hour (0–23) and Minute, then reload.

  

  **Logs:** ~/Library/Logs/news-agent.log and news-agent.err.