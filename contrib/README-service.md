# Running JankyBorders (this fork) as a background service

Use this to run your **custom fork** as a launchd service instead of the Homebrew `borders` binary, so you always get your build.

## 1. Stop the Homebrew service (if it’s running)

```bash
brew services stop borders
```

## 2. Build the fork

From the repo root:

```bash
make
```

This produces `bin/borders`.

## 3. Install the launchd plist

Copy the plist into your user LaunchAgents and point it at your built binary and a log directory.

**Option A – One-time setup with your paths**

Set the repo path and log dir (edit if needed), then run:

```bash
REPO="$HOME/Developer/Forks/JankyBorders"
LOG_DIR="$HOME/Library/Logs/JankyBorders"
mkdir -p "$LOG_DIR"

# Generate plist with your paths (HOME is set so bordersrc is found)
sed -e "s|BINARY_PATH|$REPO/bin/borders|g" -e "s|LOG_DIR|$LOG_DIR|g" -e "s|USER_HOME|$HOME|g" \
  "$REPO/contrib/com.jankyborders.fork.plist" \
  > "$HOME/Library/LaunchAgents/com.jankyborders.fork.plist"

# Load and start the service
launchctl load "$HOME/Library/LaunchAgents/com.jankyborders.fork.plist"
```

**Option B – Manual**

1. Copy `contrib/com.jankyborders.fork.plist` to `~/Library/LaunchAgents/com.jankyborders.fork.plist`.
2. Replace `BINARY_PATH` with the full path to your `bin/borders` (e.g. `/Users/you/Developer/Forks/JankyBorders/bin/borders`).
3. Replace `LOG_DIR` with a directory for logs (e.g. `$HOME/Library/Logs/JankyBorders`). Create it: `mkdir -p ~/Library/Logs/JankyBorders`.
4. Replace `USER_HOME` with your home directory (e.g. `/Users/you`) so the service finds `~/.config/borders/bordersrc`.
5. Load: `launchctl load ~/Library/LaunchAgents/com.jankyborders.fork.plist`.

**Option C – Install script (recommended)**

```bash
./contrib/install-service.sh
```

This builds, installs the plist (with `HOME` set so `~/.config/borders/bordersrc` is used), and loads the service.

## 4. Service commands

- **Start:** `./contrib/start-service.sh` or `launchctl start com.jankyborders.fork`
- **Stop:** `./contrib/stop-service.sh` or `launchctl stop com.jankyborders.fork`
- **Unload (disable at login):** `launchctl unload ~/Library/LaunchAgents/com.jankyborders.fork.plist`
- **Reload after editing plist:**  
  `launchctl unload ~/Library/LaunchAgents/com.jankyborders.fork.plist`  
  then  
  `launchctl load ~/Library/LaunchAgents/com.jankyborders.fork.plist`

## 5. After rebuilding

After you run `make` again, restart the service so it uses the new binary:

```bash
launchctl stop com.jankyborders.fork
launchctl start com.jankyborders.fork
```

## Configuration

- If you start the service with no arguments, `borders` will use `~/.config/borders/bordersrc` if that file exists (same as the Homebrew version).
- To pass options from the plist, add them as extra elements in the `ProgramArguments` array after the binary path, e.g.  
  `<string>active_color=0xffe1e3e4</string>`  
  `<string>inactive_color=0xff494d64</string>`  
  `<string>width=5.0</string>`  
  then reload the plist.

Logs: `~/Library/Logs/JankyBorders/borders.out.log` and `borders.err.log` (or whatever `LOG_DIR` you set).
