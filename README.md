
# 🧰 tcpman

**A lightweight, multi-message TCP client for [best.js](https://github.com/empowerd-cms/best.js) servers**.
Designed for Bash, `tcpman` reliably sends first_char + JSON messages and reads complete responses, even when packets arrive in pieces. Supports optional path injection and keeps every byte intact.

---

## ⚡ Quick Start

Send a connect message (`c{...}`) followed by a query (`q{...}`) with optional path auto-injection:

```bash
time tcpman localhost:6001/test123 'c{"apiKey":"changeme"}' 'q{"i":1}'
```

Example output:

```
>>> Sending: c{"apiKey":"changeme"}
{"status":"ok","type":"connect"}
>>> Sending: q{"i":1,"path":"/test123"}
{"route":"/test123","status":"ok","execution_time_seconds":0.014,"execution":[{"node":"1","func":"route_/test123","yaml_output":{"command":["echo","1"],"context":{"i":1,"path":"/test123","ECHO_OUTPUT":"hi!","ROUTE_/TEST123_OUTPUT":"1","NODE_3_OUTPUT":"hi!"},"output":"1","error":"","exitCode":0},"raw_output":"1"},{"node":"3","func":"node_3","yaml_output":{"command":["echo","hi!"],"context":{"i":1,"path":"/test123","ECHO_OUTPUT":"hi!","ROUTE_/TEST123_OUTPUT":"1","NODE_3_OUTPUT":"hi!"},"output":"hi!","error":"","exitCode":0},"raw_output":"hi!"}],"context":{"i":1,"path":"/test123","ECHO_OUTPUT":"hi!","ROUTE_/TEST123_OUTPUT":"1","NODE_3_OUTPUT":"hi!"}}
```

* `c{...}`: connect/auth message (e.g., API key).
* `q{...}`: follow-up query; `path` is automatically injected from the URL if not already present.

---

## 🏗 Usage

```bash
tcpman host:port[/path] 'c{...}' 'q{...}' ...
```

* **host:port** — target TCP server, optional path.
* **'c{...}'** — connect/auth JSON message.
* **'q{...}'** — follow-up JSON messages; path is appended automatically if the JSON object starts with `{`.

**Example:**

```bash
tcpman localhost:4000/test 'c{"apiKey":"changeme"}' 'q{"action":"ping"}'
```

* Injects `"path":"/test"` into the second message automatically.

---

## ⚙️ How it Works

1. Opens a persistent TCP connection (`/dev/tcp`) to the target server.
2. Creates a FIFO to read asynchronously from the socket.
3. Sends messages line-by-line (`first_char + JSON`).
4. Waits for the first byte of response before reading all remaining bytes until idle.
5. Optionally injects the `path` into JSON messages after the first.
6. Cleans up the FIFO and socket automatically.

**Result:** reliable, byte-accurate communication, even if packets are fragmented or delayed.

---

## 🧩 Requirements

* Bash ≥ 5.0 (`/dev/tcp`, `$EPOCHREALTIME`)
* `jq` for optional JSON path injection

No other dependencies.

---

## ⚡ Features

* **Deterministic TCP client** for `best.js` automation/testing.
* Handles partial reads correctly.
* Optional `path` injection per `best.js` conventions.
* Works in pure Bash without external TCP tools like `nc`.
* Configurable idle timeout using environment variable:

```bash
export IDLE_TIME=0.01
```

---

## 📂 Install

```bash
git clone https://github.com/empowerd-cms/tcpman
cd tcpman
echo 'export PATH="$PWD:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Now you can run `tcpman` from anywhere.

---

## ⚠️ Notes

* `tcpman` waits for **the last byte after the first byte is received**.
* If the server never sends any data (unknown path or offline), it will hang.
* Wrap with a shell timeout for safety:

```bash
timeout 5s tcpman localhost:6001/unknown 'c{"apiKey":"changeme"}' 'q{"i":1}'
```


