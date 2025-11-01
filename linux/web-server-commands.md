# Temporary Web Server Command Line Options

A comprehensive guide to spawning temporary web servers from the command line.

---

## Table of Contents
- [Python http.server](#python-httpserver)
- [Node.js http-server](#nodejs-http-server)
- [PHP Built-in Server](#php-built-in-server)
- [Ruby WEBrick](#ruby-webrick)
- [Other Options](#other-options)

---

## Python http.server

Python 3 includes a simple HTTP server module that's perfect for quick file serving.

### Basic Usage
```bash
python3 -m http.server [port]
```

### Command Line Options

| Option | Long Form | Description | Default |
|--------|-----------|-------------|---------|
| `port` | - | Port to bind to | 8000 |
| `-h` | `--help` | Show help message and exit | - |
| `-b ADDRESS` | `--bind ADDRESS` | Bind to specific address | All interfaces |
| `-d DIRECTORY` | `--directory DIRECTORY` | Serve this directory | Current directory |
| `-p VERSION` | `--protocol VERSION` | HTTP version to use | HTTP/1.0 |
| `--cgi` | - | Run as CGI server | Disabled |

### Examples
```bash
# Serve current directory on port 8000
python3 -m http.server

# Serve on specific port
python3 -m http.server 3000

# Serve specific directory
python3 -m http.server -d /path/to/files

# Bind to localhost only
python3 -m http.server -b 127.0.0.1

# Use HTTP/1.1
python3 -m http.server -p HTTP/1.1

# Combine options
python3 -m http.server 9000 -b localhost -d ./public
```

---

## Node.js http-server

A powerful, feature-rich static file server via npm. Requires Node.js.

### Installation
```bash
# Global installation
npm install -g http-server

# Or use with npx (no installation needed)
npx http-server
```

### Basic Usage
```bash
http-server [path] [options]
```

### Command Line Options

#### Basic Options
| Option | Long Form | Description | Default |
|--------|-----------|-------------|---------|
| `-p` | `--port` | Port to use (0 = find open port) | 8080 |
| `-a` | - | Address to bind to | 0.0.0.0 |
| `-d` | - | Show directory listings | true |
| `-i` | - | Display autoIndex | true |
| `-e` | `--ext` | Default file extension if none supplied | none |
| `-s` | `--silent` | Suppress log messages | false |
| `-h` | `--help` | Print help and exit | - |
| `-v` | `--version` | Print version and exit | - |

#### Compression Options
| Option | Long Form | Description | Default |
|--------|-----------|-------------|---------|
| `-g` | `--gzip` | Serve gzip files when possible | false |
| `-b` | `--brotli` | Serve brotli files when possible | false |

#### CORS & Browser Options
| Option | Description | Default |
|--------|-------------|---------|
| `--cors[=headers]` | Enable CORS, optionally with custom headers | Disabled |
| `-o [path]` | Open browser after starting, optionally to specific path | - |

#### Cache & Timeout Options
| Option | Description | Default |
|--------|-------------|---------|
| `-c` | Cache time in seconds (max-age), use -c-1 to disable | 3600 |
| `-t` | Connection timeout in seconds, use -t0 to disable | 120 |

#### Logging Options
| Option | Long Form | Description | Default |
|--------|-----------|-------------|---------|
| `-U` | `--utc` | Use UTC time format in logs | false |
| `--log-ip` | - | Enable logging of client IP address | false |

#### Proxy Options
| Option | Long Form | Description | Default |
|--------|-----------|-------------|---------|
| `-P` | `--proxy` | Fallback proxy URL if request cannot be resolved | none |
| `--proxy-options` | - | Pass options to proxy (e.g., --proxy-options.secure false) | none |

#### Authentication Options
| Option | Description | Environment Variable |
|--------|-------------|---------------------|
| `--username` | Username for basic authentication | NODE_HTTP_SERVER_USERNAME |
| `--password` | Password for basic authentication | NODE_HTTP_SERVER_PASSWORD |

#### TLS/SSL Options
| Option | Long Form | Description | Default |
|--------|-----------|-------------|---------|
| `-S` | `--tls`, `--ssl` | Enable HTTPS | Disabled |
| `-C` | `--cert` | Path to TLS certificate file | cert.pem |
| `-K` | `--key` | Path to TLS key file | key.pem |

#### Other Options
| Option | Description | Default |
|--------|-------------|---------|
| `-r`, `--robots` | Respond to /robots.txt | User-agent: *\nDisallow: / |
| `--no-dotfiles` | Do not show dotfiles | false |
| `--mimetypes` | Path to custom .types file for mimetype definitions | - |

### Examples
```bash
# Basic usage on default port 8080
http-server

# Serve on specific port
http-server -p 3000

# Serve specific directory
http-server ./public

# Enable CORS
http-server --cors

# Enable CORS with custom headers
http-server --cors="X-Custom-Header, Authorization"

# Open browser automatically
http-server -o

# Open browser to specific path
http-server -o /index.html

# Disable caching
http-server -c-1

# Enable gzip compression
http-server -g

# Bind to localhost only
http-server -a 127.0.0.1

# Enable HTTPS
http-server -S -C cert.pem -K key.pem

# With basic authentication
http-server --username admin --password secret

# Silent mode (no logs)
http-server -s

# Combine multiple options
http-server ./dist -p 8000 --cors -o -g -c-1
```

---

## PHP Built-in Server

PHP includes a built-in web server (PHP 5.4.0+) suitable for development.

### Basic Usage
```bash
php -S <host>:<port> [-t docroot] [router]
```

### Command Line Options

| Option | Description | Example |
|--------|-------------|---------|
| `-S <host>:<port>` | Run server on specified host and port | `-S localhost:8000` |
| `-t <docroot>` | Document root (default: current directory) | `-t public/` |
| `router` | Path to router script (optional) | `router.php` |

### Examples
```bash
# Basic server on localhost:8000
php -S localhost:8000

# Serve from specific directory
php -S localhost:8000 -t public/

# With router script
php -S localhost:8000 router.php

# Accessible from network
php -S 0.0.0.0:8080

# Combine options
php -S 192.168.1.100:9000 -t ./www index.php
```

---

## Ruby WEBrick

Ruby includes WEBrick, a simple HTTP server library.

### Basic Usage
```bash
ruby -run -ehttpd . -p<port>
```

### Examples
```bash
# Serve current directory on port 8000
ruby -run -ehttpd . -p8000

# Serve specific directory
ruby -run -ehttpd /path/to/files -p8080

# Bind to localhost
ruby -run -ehttpd . -p8000 -b127.0.0.1
```

### Alternative: One-liner WEBrick Server
```bash
ruby -e "require 'webrick'; WEBrick::HTTPServer.new(:Port => 8000, :DocumentRoot => Dir.pwd).start"
```

---

## Other Options

### Busybox httpd
```bash
busybox httpd -f -p 8000
```

### Twisted (Python)
```bash
# Requires: pip install twisted
twistd -n web --path=. --port=8000
```

### Live Server (Node.js)
```bash
# Install: npm install -g live-server
live-server --port=8000 --open
```

### Serve (Node.js)
```bash
# Use with npx
npx serve -l 3000
```

### Caddy
```bash
# Install from https://caddyserver.com
caddy file-server --listen :8000
```

### BrowserSync (Node.js)
```bash
# Install: npm install -g browser-sync
browser-sync start --server --port 3000
```

---

## Quick Reference Table

| Tool | Command | Default Port | Auto-reload | HTTPS | Auth |
|------|---------|--------------|-------------|-------|------|
| Python http.server | `python3 -m http.server` | 8000 | ❌ | ❌ | ❌ |
| http-server | `npx http-server` | 8080 | ❌ | ✅ | ✅ |
| PHP | `php -S localhost:8000` | - | ❌ | ❌ | ❌ |
| Ruby WEBrick | `ruby -run -ehttpd . -p8000` | - | ❌ | ❌ | ❌ |
| live-server | `npx live-server` | 8080 | ✅ | ✅ | ❌ |
| serve | `npx serve` | 3000 | ❌ | ✅ | ❌ |

---

## Tips

1. **Port Already in Use?** Most tools allow you to specify a different port with `-p` or similar options.

2. **Security Warning:** These servers are meant for development/testing only, not production use.

3. **Stop Server:** Press `Ctrl+C` to stop any running server.

4. **Check Open Port:** Use `lsof -i :<port>` or `netstat -tuln | grep <port>` to see if a port is in use.

5. **Firewall:** If you can't access from another device, check your firewall settings.

6. **Network Access:** Use `0.0.0.0` as the bind address to allow access from other devices on your network.

---

*Generated: 2025-11-01*
