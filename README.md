
# 🧰 tcpman

**An opinionated TCP client for `best.js` servers**, designed to send JSON requests and reliably read complete responses in pure Bash.

It handles partial packets, detects when the last byte arrives, and automatically injects URL paths into the JSON (`/path → "path":"/path"`) per `best.js` conventions.

---

## 🚀 Usage

Send an empty JSON request:

```bash
tcpman tcp://localhost:6001
```

Send a custom JSON payload:

```bash
tcpman tcp://localhost:6001 -d '{"command":"ping"}'
```

Path auto-injection:

```bash
tcpman tcp://localhost:6001/test1 -d '{"user":"alice"}'
# Sends: {"user":"alice","path":"/test1"}
```

---

## ⚙️ How it works

* Sends JSON to the TCP server (with optional path injection).
* Reads responses efficiently in chunks.
* Uses a short socket timeout (default 0.1s) to detect idle periods.
* Closes once no new data arrives for the configured idle period (default 1s).

**Result:** every byte is captured, and the client exits immediately after the last byte.

---

## 🧩 Requirements

* Bash ≥ 5.0 (`/dev/tcp`, `$EPOCHREALTIME`)
* `jq` (JSON validation and merging)

No other dependencies.

---

## 🔍 Why it’s reliable for `best.js`

Unlike generic TCP clients, `tcpman`:

* avoids arbitrary delays,
* handles partial reads correctly,
* and automatically formats requests per the `best.js` path convention.

This makes it a deterministic, opinionated client for automation and testing.


---

### 📂 Install and add to PATH

```bash
git clone https://github.com/empowerd-cms/tcpman
cd tcpman
echo 'export PATH="$PWD:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Now you can run `tcpman` from anywhere.

Perfect — here’s a short, clear way to add that advice:

---

### ⚠️ Note when using it for unknown paths/servers

`tcpman` waits for the **last byte after the first byte is received**.
If the server never sends any data (e.g., for an unknown path), the client will hang indefinitely.
To protect against this, you can wrap the command with a shell timeout, for example:

```bash
timeout 9s tcpman tcp://localhost:6001/unknown # automatically quits after 9s
```

This ensures the client exits even if no bytes are ever sent.

---

