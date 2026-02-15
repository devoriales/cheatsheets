# Automating Startup Tasks on macOS with LaunchAgents

As a DevOps engineer working on macOS, you probably find yourself running the same commands every time you log in: starting Docker/Colima, launching databases, spinning up local services, or firing up development tools. Let's automate that workflow using macOS LaunchAgents.

## What are LaunchAgents?

LaunchAgents are macOS's native way to run programs automatically. Think of them as systemd for macOS (but they've been around much longer). They live in `~/Library/LaunchAgents/` and are managed by `launchd`, Apple's service management framework.

**Why use LaunchAgents over Login Items?**
- More control over execution (timing, restarts, logging)
- Built-in monitoring and restart capabilities
- Standardized logging to stdout/stderr
- No GUI required - pure configuration

## The Anatomy of a LaunchAgent

LaunchAgents are defined in XML property list (plist) files. Here's a breakdown of the key components:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- Unique identifier for your service -->
    <key>Label</key>
    <string>io.example.myservice</string>
    
    <!-- Command to run with arguments -->
    <key>ProgramArguments</key>
    <array>
        <string>/path/to/executable</string>
        <string>--arg1</string>
        <string>value1</string>
    </array>
    
    <!-- Start automatically at login -->
    <key>RunAtLoad</key>
    <true/>
    
    <!-- Restart if the process dies -->
    <key>KeepAlive</key>
    <true/>
    
    <!-- Logging -->
    <key>StandardOutPath</key>
    <string>/tmp/myservice.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/myservice-error.log</string>
</dict>
</plist>
```

### Key Properties Explained

- **Label**: Unique reverse-DNS identifier (like `com.yourname.service`)
- **ProgramArguments**: Array of command and arguments (each argument is a separate string)
- **RunAtLoad**: Start when the agent is loaded (i.e., at login)
- **KeepAlive**: Restart if the process crashes
  - Can be `<true/>` for always restart
  - Or a dict with conditions: `{"Crashed": true}` to only restart on crashes
- **StandardOutPath/StandardErrorPath**: Where to write logs

## Real-World Example: Auto-starting Colima

I use Colima with a specific profile for running SQL Server containers with Rosetta emulation. Instead of running this command every time I log in:

```bash
colima start --profile mssql-amd64 \
  --vm-type vz --vz-rosetta \
  --cpu 4 --memory 8 --disk 50
```

I created this LaunchAgent:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>io.colima.start</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/colima</string>
        <string>start</string>
        <string>--profile</string>
        <string>mssql-amd64</string>
        <string>--vm-type</string>
        <string>vz</string>
        <string>--vz-rosetta</string>
        <string>--cpu</string>
        <string>4</string>
        <string>--memory</string>
        <string>8</string>
        <string>--disk</string>
        <string>50</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/colima.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/colima-error.log</string>
</dict>
</plist>
```

**Important notes:**
- Each command-line argument gets its own `<string>` element
- Use full paths to executables (find with `which colima`)
- `KeepAlive` ensures Colima restarts if it crashes

## Managing Multiple Services

Let's say you want to auto-start multiple services. Create separate plist files for each:

### PostgreSQL
`~/Library/LaunchAgents/homebrew.mxcl.postgresql@14.plist`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>homebrew.mxcl.postgresql@14</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/opt/postgresql@14/bin/postgres</string>
        <string>-D</string>
        <string>/opt/homebrew/var/postgresql@14</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardErrorPath</key>
    <string>/opt/homebrew/var/log/postgresql@14.log</string>
</dict>
</plist>
```

### Redis
`~/Library/LaunchAgents/homebrew.mxcl.redis.plist`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>homebrew.mxcl.redis</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/opt/redis/bin/redis-server</string>
        <string>/opt/homebrew/etc/redis.conf</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key>
        <false/>
    </dict>
    <key>WorkingDirectory</key>
    <string>/opt/homebrew/var</string>
    <key>StandardErrorPath</key>
    <string>/opt/homebrew/var/log/redis.log</string>
</dict>
</plist>
```

## Managing Your LaunchAgents

### Load (enable) an agent
```bash
launchctl load ~/Library/LaunchAgents/io.colima.start.plist
```

### Unload (disable) an agent
```bash
launchctl unload ~/Library/LaunchAgents/io.colima.start.plist
```

### List running agents
```bash
launchctl list | grep -v "com.apple"
```

### Check a specific agent
```bash
launchctl list | grep colima
```

### Manually start/stop an agent
```bash
# Start
launchctl start io.colima.start

# Stop
launchctl stop io.colima.start
```

### Reload after editing
```bash
launchctl unload ~/Library/LaunchAgents/io.colima.start.plist
launchctl load ~/Library/LaunchAgents/io.colima.start.plist
```

## Advanced Configuration Options

### Environment Variables
```xml
<key>EnvironmentVariables</key>
<dict>
    <key>PATH</key>
    <string>/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string>
    <key>KUBECONFIG</key>
    <string>/Users/youruser/.kube/config</string>
</dict>
```

### Working Directory
```xml
<key>WorkingDirectory</key>
<string>/Users/youruser/projects</string>
```

### Conditional KeepAlive
Restart only on crashes, not clean exits:
```xml
<key>KeepAlive</key>
<dict>
    <key>Crashed</key>
    <true/>
</dict>
```

### Delayed Start
Wait 10 seconds after login before starting:
```xml
<key>StartInterval</key>
<integer>10</integer>
```

### Throttle Restarts
Prevent rapid restart loops:
```xml
<key>ThrottleInterval</key>
<integer>60</integer>
```

## Troubleshooting

### Check if your agent is loaded
```bash
launchctl list | grep your-label-name
```

If it shows a PID, it's running. If it shows "-", it's loaded but not running.

### Check logs
Always set up logging in your plist:
```bash
tail -f /tmp/colima.log
tail -f /tmp/colima-error.log
```

### Common issues

**Permission denied**: Ensure the executable path is correct and has execute permissions
```bash
ls -la /opt/homebrew/bin/colima
```

**PATH issues**: LaunchAgents don't inherit your shell's PATH. Use full paths or set `EnvironmentVariables`

**Agent not loading**: Validate your plist syntax
```bash
plutil -lint ~/Library/LaunchAgents/io.colima.start.plist
```

**Still not working?**: Check system logs
```bash
log show --predicate 'process == "launchd"' --last 10m | grep -i error
```

## Organizing Your LaunchAgents

I recommend a consistent naming convention:

- **Homebrew services**: `homebrew.mxcl.<service>.plist`
- **Custom scripts**: `local.<username>.<purpose>.plist`
- **Third-party apps**: Use their reverse DNS (e.g., `io.colima.start.plist`)

Keep a backup of your LaunchAgents in a dotfiles repo:
```bash
# In your dotfiles repo
mkdir -p LaunchAgents
cp ~/Library/LaunchAgents/*.plist LaunchAgents/

# Create a sync script
cat > sync-launchagents.sh << 'EOF'
#!/bin/bash
for file in LaunchAgents/*.plist; do
    filename=$(basename "$file")
    cp "$file" ~/Library/LaunchAgents/
    launchctl unload ~/Library/LaunchAgents/"$filename" 2>/dev/null
    launchctl load ~/Library/LaunchAgents/"$filename"
done
EOF
chmod +x sync-launchagents.sh
```

## When NOT to Use LaunchAgents

LaunchAgents are great, but not for everything:

- **One-off tasks**: Just run them manually or use shell aliases
- **Interactive programs**: LaunchAgents run without a GUI context
- **Tasks that need user input**: No stdin available
- **Scheduled tasks**: Use `launchd` calendar intervals or `cron` instead

For scheduled tasks, add this to your plist:
```xml
<key>StartCalendarInterval</key>
<dict>
    <key>Hour</key>
    <integer>9</integer>
    <key>Minute</key>
    <integer>0</integer>
</dict>
```

## Conclusion

LaunchAgents are a powerful way to automate your development environment on macOS. They're more reliable than Login Items, provide better logging, and integrate deeply with the OS. Whether you're auto-starting Docker, databases, or custom dev tools, LaunchAgents give you fine-grained control over your startup sequence.

The key principles:
1. One service per plist file
2. Use full paths to executables
3. Each argument is a separate string element
4. Always configure logging
5. Use descriptive Label names
6. Version control your agents

Now stop typing the same startup commands every morning and let launchd handle it!

---

**Resources:**
- [Apple's launchd documentation](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html)
- [launchd.info](https://www.launchd.info/) - Community resource for LaunchAgent examples
- Your existing agents: `ls ~/Library/LaunchAgents/`

---

*Questions or improvements? Drop a comment below or reach out on [your social media links].*
