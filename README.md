```
██╗  ██╗ █████╗ ███╗   ███╗██████╗ ██████╗
╚██╗██╔╝██╔══██╗████╗ ████║██╔══██╗██╔══██╗
 ╚███╔╝ ███████║██╔████╔██║██████╔╝██████╔╝
 ██╔██╗ ██╔══██║██║╚██╔╝██║██╔═══╝ ██╔═══╝
██╔╝ ██╗██║  ██║██║ ╚═╝ ██║██║     ██║
╚═╝  ╚═╝╚═╝  ╚═╝╚═╝     ╚═╝╚═╝     ╚═╝
              C L I   M A N A G E R
```

# XAMPP CLI Manager

A terminal launcher and dashboard for **XAMPP on macOS**. Type `xampp` and you can start, stop and check Apache, MySQL and ProFTPD from the same terminal window.

## Why this exists

Since the **macOS 27 update**, the XAMPP manager app that ships with XAMPP (`/Applications/XAMPP/manager-osx.app`) doesn't open on Apple Silicon Macs. When you double-click it, nothing happens.

The reason is the app's launcher script (`Contents/MacOS/installbuilder.sh`), which was written in 2018. It only accepts Intel CPUs:

```sh
machine_platform=`uname -p`          # "arm" on Apple Silicon
elif [ "${machine_platform}" == "i386" ];then
...
echo "The current OS X version is not supported"
```

XAMPP itself still works. Apache, PHP and MySQL are Intel builds that run through Rosetta 2. Only the graphical manager is broken.

**This project is just a launcher from the command line that replaces that manager.** It doesn't replace or modify XAMPP. It drives XAMPP's own control script (`/Applications/XAMPP/xamppfiles/xampp`) and adds a status dashboard, health checks and a few fixes for XAMPP bugs on macOS.

## Features

- **Animated intro:** the XAMPP logo wipes in when the dashboard opens.
- **Live service status:** green (running), yellow (running but its port isn't answering) or red (stopped) for Apache, MySQL and ProFTPD.
- **One-key actions:** start, stop and restart everything or each service separately.
- **Readable results:** `✔ started`, `● already running`, `✖ failed to start`, with a spinner while working.
- **Logs only when something fails:** the last 8 lines of the relevant log, formatted and color-coded, plus a warning if another app is using the port (for example, "Port 80 is taken by httpd").
- **Health check:** ports, website and phpMyAdmin HTTP responses, MySQL connection, MySQL system tables and recent Apache errors.
- **Resources panel:** CPU (as % of the whole Mac), memory and macOS Energy Impact for each service, plus a total.
- **Safe MySQL upgrade:** checks whether `mysql_upgrade` is needed, warns you and asks for confirmation, then backs up, verifies the backup, upgrades and checks every table.
- **Shortcuts:** open the website, phpMyAdmin or the `htdocs` folder, or follow the Apache error log live.

```
  ╭─ Services ─────────────────────────────────────────────────────╮
  │ ●  Apache    running      :80     http://localhost             │
  │ ●  MySQL     running      :3306   localhost/phpmyadmin         │
  │ ●  ProFTPD   stopped      :21                                  │
  ╰────────────────────────────────────────────────────────────────╯
  ╭─ Actions ──────────────────────────────────────────────────────╮
  │ All       1  Start         2  Stop          3  Restart         │
  │ Apache    4  Start         5  Stop          6  Restart         │
  │ MySQL     7  Start         8  Stop                             │
  │ ProFTPD   9  Start         0  Stop                             │
  │ Open      w  Website       p  phpMyAdmin    h  htdocs          │
  │ Tools     c  Health check  u  Upgrade DB    l  Live log        │
  │           r  Refresh       q  Quit                             │
  ╰────────────────────────────────────────────────────────────────╯
  ╭─ Resources ────────────────────────────────────────────────────╮
  │ SERVICE      PROCS       CPU      MEMORY    ENERGY             │
  │ Apache           7      0.1%    102.0 MB       0.4             │
  │ MySQL            1      0.0%     56.0 MB       0.2             │
  │ ProFTPD          –         –           –         –             │
  │ this panel       1      0.0%      2.3 MB       0.0             │
  ├────────────────────────────────────────────────────────────────┤
  │ Total                   0.1%    160.3 MB       0.6             │
  │ CPU = % of whole Mac (10 cores) · energy = macOS Energy Impact │
  ╰────────────────────────────────────────────────────────────────╯
```

## Requirements

- macOS on Apple Silicon or Intel. It was tested on macOS 27 with an M-series Mac.
- XAMPP 8.x installed at `/Applications/XAMPP`. It was tested with XAMPP 8.2.4 (MariaDB 10.4.28).
- **Rosetta 2** on Apple Silicon, because XAMPP's binaries are Intel-only. If it's missing, install it with `softwareupdate --install-rosetta`.
- An admin account. Starting and stopping services asks for your password once per window, because Apache uses port 80 and XAMPP's files belong to `root`.

It uses only tools that come with macOS (zsh, `top`, `netstat`, `curl`, `ditto`), so there's nothing else to install.

## Install

```sh
git clone https://github.com/Reymlgz/xampp-cli-manager.git ~/xampp-cli-manager
chmod +x ~/xampp-cli-manager/xampp
mkdir -p ~/.local/bin
ln -s ~/xampp-cli-manager/xampp ~/.local/bin/xampp
```

If `~/.local/bin` isn't in your `PATH` yet, add this line to `~/.zshrc`, then open a new terminal:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

## Usage

```sh
xampp                 # open the dashboard
```

| Command            | What it does                                          |
|--------------------|-------------------------------------------------------|
| `xampp start`      | Start Apache, MySQL and ProFTPD                       |
| `xampp stop`       | Stop all services                                     |
| `xampp restart`    | Restart all services                                  |
| `xampp status`     | Show the Services box                                 |
| `xampp check`      | Run the health check                                  |
| `xampp usage`      | Show CPU, memory and energy for each service          |
| `xampp upgrade-db` | Check whether MySQL needs `mysql_upgrade`, and run it safely |
| `xampp web`        | Open http://localhost                                 |
| `xampp pma`        | Open phpMyAdmin                                       |
| `xampp htdocs`     | Open the `htdocs` folder in Finder                    |
| `xampp logs`       | Follow the Apache error log live                      |

To skip the startup animation, add `export XAMPP_NO_INTRO=1` to `~/.zshrc`.

## Fixes for XAMPP bugs on macOS

While building this, we found a few XAMPP problems. The manager works around them:

- **"Stop MySQL" doesn't stop MySQL.** XAMPP's `bin/mysql.server` stops MySQL by running `su - mysql -s /bin/sh -c "kill …"`. That always fails on macOS, because macOS's `su` has no `-s` option and the `_mysql` account can't log in. XAMPP then concludes that MySQL "isn't running", **deletes the PID file**, reports success, and leaves MySQL running. This affects the original manager app too. This manager sends MySQL the normal shutdown signal (`kill -TERM`) directly and waits until the process has really exited.
- **Missing PID files.** XAMPP decides whether a service is running only from its PID file. If that file disappears while the service is still running, XAMPP can't stop it. Before every action, the manager restores missing PID files from the running processes.
- **MySQL log flooding.** Checking whether a port is open by connecting to it makes MySQL log an `Aborted connection` warning every time. The manager asks macOS which ports are listening (`netstat`) instead, without connecting.

## MySQL upgrade (`u` / `xampp upgrade-db`)

After XAMPP or MariaDB has been updated, MariaDB's internal tables can be out of date. MySQL then fills its log with errors like `Incorrect definition of table mysql.column_stats`. `mysql_upgrade` fixes this. The manager:

1. **Checks whether an upgrade is needed.** It looks at whether `mysql_upgrade` has ever run, whether the table version matches the server version, and whether system-table errors appeared since the last start. If none applies, it tells you that and does nothing.
2. **Shows what will happen and asks `y / N`:**
   - MySQL is stopped for about a minute.
   - A full backup is made first.
   - Only internal tables change.
   - Don't close the window until it says Done.
3. **Runs these steps in order, stopping at the first failure:**
   - stop MySQL
   - copy the whole data folder to `~/Backups/xampp-mysql-<date>` and compare it file by file with the original
   - start MySQL
   - run `mysql_upgrade`
   - restart MySQL to confirm it starts cleanly
   - check every table with `mysqlcheck`

If anything fails, it shows the MySQL log, where your backup is, and the exact command to restore it. Your databases' data and your `htdocs` files are never modified.

## Optional: repair the original manager app

If you'd rather use XAMPP's graphical manager, this one-line patch makes its launcher accept Apple Silicon. It keeps a backup as `installbuilder.sh.bak`:

```sh
sudo sed -i.bak 's/elif \[ "${machine_platform}" == "i386" \];then/elif [ "${machine_platform}" == "i386" ] || [ "${machine_platform}" == "arm" ];then/' \
  /Applications/XAMPP/xamppfiles/manager-osx.app/Contents/MacOS/installbuilder.sh
```

The "Stop MySQL" bug described above still affects the graphical manager.

## Notes

- **Your password:** the manager only runs XAMPP's own control script and the MySQL steps above as admin. It never stores your password. To approve with your fingerprint instead of typing the password, you can turn on Touch ID for `sudo` (see `/etc/pam.d/sudo_local.template`).
- **Rosetta's future:** Apple is reducing Rosetta support starting with macOS 27. If a future macOS stops running Intel apps, XAMPP itself will stop working. At that point, a native setup such as Homebrew's `httpd`, `php` and `mariadb` is the way to go.
- **Services keep running after you quit.** Pressing `q` only closes the dashboard.

## Uninstall

```sh
rm ~/.local/bin/xampp
rm -rf ~/xampp-cli-manager
```

XAMPP itself isn't affected.

---

Released under the [MIT License](LICENSE). Not affiliated with Apache Friends or Bitnami. XAMPP is a project of Apache Friends.
