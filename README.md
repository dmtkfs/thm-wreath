# [Wreath Room - Concise Walkthrough](https://tryhackme.com/room/wreath)

> *NOTE:  This write‑up only records *facts, commands, and question answers*. Whenever a step was literally “follow the box instructions”, that fact is stated and no extra detail is invented.*

---

## 0. Prelude - Network Facts from Thomas

* Three hosts → one CentOS web server, one *internal* Git server, one Windows workstation (repurposed server).
* Git server is private; public‑facing site is a clone.
* Workstation has AV protections and no direct exposure.

---

## 1. External Enumeration & Foothold (xx.xxx.xx.xxx)

| Item                   | Result                                                   |
| ---------------------- | -------------------------------------------------------- |
| Open ports (<15 000)   | **x**                                                    |
| Nmap OS guess          | **xxxxxx**                                               |
| Web redirect           | `https://xxxxxxxxxxx.xxx`                                |
| Manual hosts entry     | `xx.xxx.xx.xxx x.xxx`                                    |
| Phone number on page   | **+xxxxxxxxxxxx**                                        |
| Service version (Nmap) | **xxxxxxxxxxxxxxxxxxxxxxx**                              |
| Vulnerability          | **CVE‑xxxx‑xxxxx**                                       |

*Exploit performed exactly as described in the room using Metasploit.*

### Loot

* Root SSH key: **`/xxxx/.xxx/xxxxxxx`** (chmod 600 locally).
* Root hash: `$x$x$xxx…` (not crackable).

---

## 2. Pivot Scanning from .xxx

1. Upload static nmap (renamed **`nmap-<USER>`**) → `/tmp`.
2. Scan `xx.xxx.xx.xxx/24 ‑sn`.

| Finding                                     | Value              |
| ------------------------------------------- | ------------------ |
| Alive hosts (excluding out‑of‑scope & .200) | **x**              |
| Last octets                                 | **x, x**           |
| Host with non‑filtered ports                | **x**              |
| Open TCP (<15000)                           | **x, x, x**        |
| Most promising service                      | **xxxx**           |

Pivot path established with **`sshuttle`** (per room instructions) from attacker → xx.xxx.xx.xxx via xx.xxx.xx.xxx.

---

## 3. GitStack RCE on xx.xxx.xx.xxx

| Detail                 | Answer          |
| ---------------------- | --------------- |
| Service banner         | **xxxxxxxx**    |
| Exploit (searchsploit) | **xxx xxxxx**   |
| Exploit date           | **xx‑xx‑xxxx**  |
| Cookie name on line 74 | **`xxxxxxxxx`** |

Running the modified `xxxxxx.py` yielded an **NT AUTHORITY\SYSTEM** shell.

### Enumerated Facts

| Prompt                              | Answer                  |
| ----------------------------------- | ----------------------- |
| Hostname                            | **xxx-xxxx**            |
| OS                                  | **xxxxxxx**             |
| Running user                        | **xx xxxxxxxxx\xxxxxx** |
| ICMP test packets reaching listener | **x**                   |

### Hashes via Mimikatz

| Account         | Hash                                 |
| --------------- | ------------------------------------ |
| `Administrator` | **xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx** |
| `Thomas` NTLM   | **xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx** |
| Thomas password | **xxxxxxx**                          |

Subsequent access used **Evil‑WinRM** with the Administrator hash tunneled through SSH port‑forwarding.

---

## 4. Second Pivot → xx.xxx.xx.xxx (actually lateral move via .xxx)

Proxy chain built with:

1. SSH `-L 5985` & `-L 47099` through xx.xxx.xx.xxx.
2. Chisel **server** on xx.xxx.xx.xxx (`--socks5`) and **client** on Attacking Machine.
3. Browser/Burp through local SOCKS5 (127.0.0.1:1080).

### Quick Findings

| Prompt                                  | Answer                                     |
| --------------------------------------- | ------------------------------------------ |
| Open web ports (Invoke‑Portscan top 50) | **xx, xxxx**                               |
| Server‑side language                    | **xxx x.x.xx**                             |
| Absolute path to `Website.git`          | **`x:\xxxxxxxx\xxxxxxxxxxxx\xxxxxxx.xxx`** |
| Thomas to phone                         | **xxxxxxxxxxxxx xxxxx xxxxxxxx**           |
| Upload filter accepted extensions       | **xxx,xxxx,xxx,xxx**                       |

### Auth & Web Shell

* Login: **xxxxxx : xxxxxxxx**.
* Obfuscated PHP shell embedded into image; accessed as instructed.

| Prompt                | Answer                                            |
| --------------------- | ------------------------------------------------- |
| Hostname              | **xxxxxx‑xx**                                     |
| Current user          | **xxxxxx-xx\xxxxxx**                              |
| `certutil.exe` output | `CertUtil: -dump command completed successfully.` |

Reverse shell executed with `nc-xxxx.exe` → attacker port 4444.

---

## 5. Privilege Escalation on xxxxxx‑xx

Privilege `xxxxxxxxxxxxxxxxxxxxx` confirmed (PrintSpoofer/Potato class).

Unquoted‑service‑path identified:

* Service **xxxxxxxxxxxxxxxxxxxx**
* Runs as **xxxxxxxxxxxx**
* Writable dir: `x:\xxxxxxx xxxxxxx (xxx)\xxxxxx xxxxxxxx\`

Steps:

```cmd
copy %TEMP%\wrapper-xxxx.exe "x:\xxxxxxxxxx xxxxx (xxx)\xxxxxxxx xxxxxxxxx\System.exe"
sc stop xxxxxxxxxxxxxxxxxxxxxxxxxx
sc start xxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Netcat listener caught **xx xxxxxxxxxxxx\xxxxxxxxxx**.

### SAM Hive Dump (Impacket `xxxxxxxxxxxx`)

| Account       | NT Hash                              |
| ------------- | ------------------------------------ |
| Administrator | **xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx** |

---

## 6. Clean‑Up Notes

* Deleted `System.exe`, restored service, closed firewall holes as per box guidance.
* All temporary binaries (nmap, nc, wrapper, chisel) removed from targets.

---

### END

Full command transcripts retained privately. No extra content has been invented beyond the room instructions and captured output.
