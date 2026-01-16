# Tremulous Master Server

A Python implementation of the Tremulous Master Server, designed to track game servers and provide server lists to clients.

## Overview

The Tremulous Master Server is a UDP-based server that maintains a list of active Tremulous game servers. It handles server heartbeats, verifies servers through challenge-response authentication, and provides server lists to clients upon request. The server supports both IPv4 and IPv6, includes logging capabilities, and offers various security features.

## Features

### Core Functionality

- **Server Tracking**: Maintains a list of active game servers through heartbeat messages
- **Challenge-Response Verification**: Servers are verified before being added to the server list
- **Server List Distribution**: Responds to client requests for server lists
- **Server Caching**: Persists server list to disk (`serverlist.txt`) for quick recovery
- **Timeout Management**: Automatically removes inactive servers

### Network Support

- **Dual-Stack Operation**: Supports both IPv4 and IPv6 simultaneously
- **Multiple Listening Ports**: Can listen on multiple ports for incoming requests
- **Separate Challenge Port**: Uses a dedicated port for outgoing challenges to avoid NAT issues
- **Address Filtering**: Supports IP blacklisting via CIDR notation

### Protocol Support

The master server implements the following protocol messages:

| Protocol | Description |
|----------|-------------|
| `heartbeat <game>\n` | Server registration request |
| `getservers <protocol> [empty] [full]` | Standard server list request |
| `getserversExt Tremulous <protocol> [ipv4\|ipv6] [empty] [full]` | Extended server list request |
| `getmotd` | Message of the day request |
| `gamestat` | Game statistics logging |
| `infoResponse` | Server challenge response |

### Featured Servers

- **Labeled Server Groups**: Featured servers can be organized into labeled groups
- **Special Display**: Featured servers are sent in separate response packets with labels
- **Configuration File**: Featured servers defined in `featured.txt`

### Security Features

- **IP Blacklisting**: Block specific IP addresses or CIDR ranges via `ignore.txt`
- **Challenge-Response Authentication**: Servers must respond to a challenge before being listed
- **Chroot Support**: Can run in a chroot jail for enhanced security
- **Privilege Dropping**: Supports switching to a non-root user after initialization
- **Packet Filtering**: Rejects invalid packets and blacklisted sources

### Logging and Statistics

- **Multiple Verbosity Levels**: Configurable logging (ERROR, PRINT, VERBOSE, DEBUG)
- **Database Backends**: Supports SQLite, TDB, or no database
- **Client Statistics**: Logs client version and renderer information
- **Game Statistics**: Records game statistics from servers
- **Timestamped Logging**: All log entries include timestamps

### Configuration

- **Command-Line Options**: Extensive command-line interface for configuration
- **Configuration Files**: Uses text files for featured servers, blacklist, and MOTD
- **Configurable Timeouts**: Adjust challenge and server timeouts
- **Max Server Limit**: Optional limit on the number of tracked servers

## Requirements

### Python Version

- Python 2.6 or higher

### Standard Library Dependencies

The following Python standard library modules are required:

- `socket` - Network socket operations
- `select` - I/O multiplexing
- `signal` - Signal handling
- `time` - Time functions
- `os` - Operating system interface
- `sys` - System-specific parameters
- `itertools` - Iterator functions
- `random` - Random number generation
- `errno` - Standard errno system symbols
- `hashlib` - Secure hash and message digest algorithms
- `functools` - Higher-order functions and operations

### Optional Dependencies

- `sqlite3` - For SQLite database backend (standard library in Python 2.5+)
- `tdb` - For TDB (Trivial Database) backend
- `pwd` - For user switching (Unix only)
- `grp` - For group switching (Unix only)

## Installation

1. Clone or download the repository
2. Ensure Python 2.6+ is installed
3. (Optional) Install the `tdb` module if you want to use the TDB database backend
4. (Optional) Create configuration files (see Configuration section)

## Usage

### Basic Usage

Run the master server with default settings:

```bash
python master.py
```

### Command-Line Options

| Option | Description |
|--------|-------------|
| `-h, --help` | Display help and exit |
| `-4, --ipv4` | Only use IPv4 |
| `-6, --ipv6` | Only use IPv6 |
| `-d, --db <none\|tdb\|sqlite\|auto>` | Database backend (default: auto) |
| `-j, --jail <DIR>` | Path to chroot into at startup |
| `-l, --listen-addr <ADDR>` | IPv4 address to listen to |
| `-L, --listen6-addr <ADDR>` | IPv6 address to listen to |
| `-n, --max-servers <NUM>` | Maximum number of servers to track |
| `-p, --port <NUM>` | Port for incoming requests (can be specified multiple times) |
| `-P, --challengeport <NUM>` | Port for outgoing challenges |
| `-q` | Decrease verbose level (can be used multiple times) |
| `-u, --user <USER>` | User to switch to at startup |
| `-v` | Increase verbose level (can be used multiple times) |
| `--verbose <LEVEL>` | Set verbose level directly (0-4) |
| `-V, --version` | Show version information |

### Examples

Run with IPv4 only on port 30710:

```bash
python master.py -4 -p 30710
```

Run with verbose logging and SQLite database:

```bash
python master.py -v -d sqlite
```

Run with chroot and privilege dropping:

```bash
sudo python master.py -j /var/chroot/tremulous-master -u tremulous
```

## Configuration Files

### featured.txt

Defines featured server groups with labels. Format:

```
Label Name
server1.example.com:30720
server2.example.com:30720

Another Label
server3.example.com:30720
```

- Lines starting with a label (no leading whitespace) define a new group
- Indented lines define server addresses for that group
- Blank lines and comments (lines starting with `#`) are ignored

### ignore.txt

Defines IP addresses and CIDR ranges to blacklist. Format:

```
# Comments start with #
192.168.1.100
10.0.0.0/8
2001:db8::/32
```

### motd.txt

Contains the message of the day sent to clients. Simple text file with one message.

### serverlist.txt

Automatically generated file that caches the server list. Created and managed by the master server.

## Database Setup

### SQLite

To create a new SQLite database:

```bash
python logsqlite.py stats.db
```

This creates two tables:
- `clients` - Stores client statistics (addr, version, renderer)
- `gamestats` - Stores game statistics (addr, time, data)

### TDB

The TDB backend automatically creates database files (`clientStats.tdb` and `gameStats.tdb`) when needed.

### No Database

To run without database logging:

```bash
python master.py -d none
```

## Protocol Details

### Heartbeat

Servers send heartbeat messages to register:

```
heartbeat Tremulous\n
```

The master responds with a challenge packet to verify the server.

### getservers

Clients request server lists:

```
getservers 71 empty full
```

- `71` - Protocol version
- `empty` - Include empty servers
- `full` - Include full servers

### getserversExt

Extended server list request:

```
getserversExt Tremulous 71 ipv4 ipv6 empty full
```

Supports IPv4/IPv6 filtering and returns multiple packets if needed.

### getmotd

Clients request the message of the day:

```
\xff\xff\xff\xffgetmotd\challenge\version\renderer\...
```

### infoResponse

Servers respond to challenges:

```
\xff\xff\xff\xffinfoResponse\challenge\...\hostname\...\protocol\...\clients\...\sv_maxclients\...
```

## Logging Levels

The master server supports five logging levels:

| Level | Value | Description |
|-------|-------|-------------|
| ALWAYS | 0 | Always displayed |
| ERROR | 1 | Error messages only |
| PRINT | 2 | Default level |
| VERBOSE | 3 | Detailed operational information |
| DEBUG | 4 | Full debugging output |

Use `-v` to increase verbosity or `-q` to decrease it. Use `--verbose <LEVEL>` to set directly.

## Default Values

| Setting | Default Value |
|---------|---------------|
| Listen Port | 30710 |
| Challenge Port | 30711 (or next available) |
| Challenge Length | 12 characters |
| Challenge Timeout | 5 seconds |
| Server Timeout | 660 seconds (11 minutes) |
| Max Servers per Packet | 256 |
| Database Backend | auto (tries SQLite, then TDB) |

## File Structure

```
tremulous-master-web/
├── master.py          # Main server implementation
├── config.py          # Configuration management
├── db.py              # Database connection abstraction
├── logsqlite.py       # SQLite logging backend
├── logtdb.py          # TDB logging backend
├── utils.py           # Utility functions
├── .gitignore         # Git ignore patterns
├── featured.txt       # Featured servers configuration (optional)
├── ignore.txt         # IP blacklist (optional)
├── motd.txt           # Message of the day (optional)
├── serverlist.txt     # Server cache (auto-generated)
└── stats.db           # SQLite database (auto-generated, if using SQLite)
```

## License

This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 2 of the License, or (at your option) any later version.

## Credits

- Original C implementation: Mathieu Olivier
- Python implementation: Ben Millwood (2009-2011)
- Updates: Jeff Kent (2015), Darren Salt (2012)

## Troubleshooting

### Port Already in Use

If you get a "port already in use" error, either:
- Stop the existing process using the port
- Use a different port with `-p <PORT>`

### Database Import Errors

If you see "database not available" warnings:
- Install the required database module (`sqlite3` is included in Python 2.5+)
- Use `-d none` to run without database logging

### Permission Denied

If you get permission errors:
- Ensure you have permission to bind to the specified ports (ports < 1024 require root)
- Check file permissions for configuration files and databases

### IPv6 Not Working

If IPv6 doesn't work:
- Ensure your system supports IPv6
- Check that the IPv6 address is valid and not already in use
- Use `-4` to run with IPv4 only
