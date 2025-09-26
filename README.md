# systeminfo

A highly configurable and performance-optimized shell script for displaying key system information upon login. 

Inspired by tools like `htop` and `atop`, `systeminfo` gives you a quick, comprehensive snapshot of your server's state the moment you connect.

## Example Output  
```
  DATE | 2025-09-20 17:47:50

  CPU  | AMD Ryzen 7 8840HS w/ Radeon 780M Graphics
  #CPU | 16
  LA   | 0.27 0.41 0.59
  CPU% | 0.3%
  TEMP | 42.0°C

  MEM  | tot     14.4G | used     6.4G | avail     8.0G
  SWP  | tot      4.0G | used       3M | free      4.0G

  ARCH | x86_64
  KERN | 6.14.0-29-generic
  OS   | Ubuntu 25.04
  HOST | betelgeuse

  UP   | 1 hour, 25 minutes

  DF   | /dev/mapper/ubuntu--vg-ubuntu--lv  935G  526G  362G  60% /
  DF   | /dev/nvme0n1p2                       2.0G  225M  1.6G  13% /boot
  DF   | /dev/nvme0n1p1                       1.1G  6.2M  1.1G   1% /boot/efi

  TRAF | wlp1s0             Rx 1.3G, Tx 80.4M
  TRAF | docker0            Rx 0B, Tx 0B

  I/O  | nvme0n1            r/s 336.00  w/s 260.00
  I/O  | loop15             r/s 4.39    w/s 0.00

  NET  | wlp1s0             192.168.50.128/24
  NET  | docker0            172.17.0.1/16

  PUBIP| 8.8.4.4

  PORTS| 22      (sshd)
  PORTS| 17500   (dropbox)

  WHO  | badrqst  seat0        Sep 20 16:50
  WHO  | badrqst  tty2         Sep 20 16:50

  TOP  |    PID %CPU %MEM COMMAND
  TOP  |  30413  100  0.1 apt-check
  TOP  |  23481 24.7  2.8 chrome
  TOP  |   4444  6.2  4.1 gnome-shell

  TOP M|    PID %MEM COMMAND
  TOP M|   6041  4.7 dropbox
  TOP M|   4444  4.1 gnome-shell
```

## About The Project
`systeminfo` was created to solve a simple problem: getting a quick yet thorough overview of a server's health right after logging in via SSH. Instead of running multiple commands (`df -h`, `free -m`, `uptime`, etc.), you get everything in one clean, readable report.

### Key features include:
* Comprehensive Overview: From CPU load to Docker containers, get all the vital stats in one place.
* High Performance: Uses parallel execution for slow operations (like network requests) to ensure the script runs in a fraction of a second.
* Highly Configurable: You can choose exactly which information blocks to display by passing simple arguments.
* Cross-Distro Compatibility: Designed to work on a wide range of Linux distributions (Debian, Ubuntu, CentOS, Fedora, etc.) by gracefully handling missing commands and tools.
* Easy Installation: A simple one-liner gets you up and running in seconds.

## Features
`systeminfo` is divided into modular blocks, each providing specific information. You can choose which blocks to display when running the script.

### Default Blocks (Fast & Always On)

These blocks are displayed by default as they are fast and rely on core system utilities.

| Key   | Block Name          | Description                                                              |
|-------|---------------------|--------------------------------------------------------------------------|
| date  | Date & Time         | Current system date and time.                                            |
| cpu   | CPU Info            | CPU model, core count, load average, and current utilization percentage. |
| gpu   | GPU Info            | GPU model, temperature, and utilization (requires `nvidia-smi`).         |
| mem   | Memory Usage        | RAM and Swap usage (total, used, available/free).                        |
| sys   | System Info         | OS, kernel, architecture, and hostname.                                  |
| up    | Uptime              | How long the system has been running.                                    |
| df    | Disk Usage          | Filesystem usage, excluding temporary and loop devices.                  |
| net   | Network IPs         | Local IP addresses for all network interfaces.                           |
| pubip | Public IP           | The server's public-facing IP address.                                   |
| traf  | Network Traffic     | Total received (Rx) and transmitted (Tx) data for network interfaces.    |
| users | User Info           | Currently logged-in users (`who`) and recent logins (`last`).            |
| top   | Top Processes (CPU) | The top 3 processes consuming the most CPU.                              |
| dckr  | Docker Info         | A list of currently running Docker containers.                           |

### Optional Blocks (Slower or More Specific)
These blocks are not shown by default as they can be slower or require special permissions/tools.

| Key   | Block Name          | Description                                                                       |
|-------|---------------------|-----------------------------------------------------------------------------------|
| io    | I/O Activity        | Disk read/write activity (requires `iostat` from `sysstat` package).              |
| ports | Listening Ports     | A list of open TCP/UDP ports and the processes listening on them (requires `ss`). |
| updt  | Package Updates     | The number of available system updates (for APT or DNF).                          |
| fails | Failed Logins       | The last few failed login attempts (requires `lastb` and often `sudo`).           |
| topm  | Top Processes (Mem) | The top 3 processes consuming the most Memory.                                    |

## Installation
### Quick Install (Recommended)
This command will download the script, place it in /usr/local/bin, and make it executable.
```bash
sudo wget -O /usr/local/bin/systeminfo https://raw.githubusercontent.com/romantelychko/systeminfo/main/systeminfo \
    && sudo chmod +x /usr/local/bin/systeminfo
```

### Auto-run on Login
To have systeminfo run every time you log in via SSH, add it to your shell's profile file.
```bash
echo -e "\nsysteminfo\n" >> ~/.profile
```

Note: Depending on your system, you might need to use `~/.profile` or `~/.bash_login` instead of `~/.bash_profile`.

## Usage
Running the script is simple and flexible.

### Default View
To get the standard set of fast information blocks, just run the command:
```bash
systeminfo
```

### Custom View
To display only specific blocks, provide their keys as a comma-separated list with a - prefix.

#### Example 1: Show only CPU and Memory usage
```bash
systeminfo -cpu,mem
```

#### Example 2: Show all default blocks plus disk I/O activity and available updates
(The script is smart enough to show the default blocks in addition to the ones you request).
```bash
systeminfo -io,updt
```

#### Example 3: Show all available blocks
To see a complete report with every possible information block, use:
```bash
systeminfo -date,cpu,gpu,mem,sys,up,df,net,pubip,traf,users,top,dckr,io,ports,updt,fails,topm
```

## Dependencies
For full functionality, some blocks require external tools. 
The script will safely skip any block if its dependencies are not met.

* `io`: Requires iostat.
  * Debian/Ubuntu: `sudo apt install sysstat`
  * Fedora/CentOS: `sudo dnf install sysstat`
* `gpu`: Requires nvidia-smi for NVIDIA GPUs. 
  * Install the official NVIDIA drivers.
* `fails`: Requires `lastb`, which is usually included in util-linux. May require `sudo` permissions.
* `cpu%`: Requires bc for calculations.
  * Debian/Ubuntu: `sudo apt install bc`
  * Fedora/CentOS: `sudo dnf install bc`
  
## Contributing
Contributions are what make the open-source community such an amazing place to learn, inspire, and create. 
Any contributions you make are greatly appreciated.
If you have a suggestion that would make this better, please fork the repo and create a pull request. 
You can also simply open an issue with the tag "enhancement".

## License
Distributed under the MIT License. See [LICENSE](./LICENSE) file for more information.

