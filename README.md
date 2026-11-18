# network-notes
Notes and Wireshark captures from learning how the network works.

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/cfbb2e14-32af-40f5-ad9a-f0a49e8d0ce8" />
<img width="1918" height="1027" alt="image" src="https://github.com/user-attachments/assets/091daed5-3c19-4981-b2ee-430f3c7d62b6" />
A difference between an https and http

# How a web request works (my Wireshark notes)

Notes and captures from learning how the network works, weeks 1-2 of my networking plan. I tested everything on my own computer and my own network.

## The 5 steps of opening a website<img width="707" height="343" alt="site_checker" src="https://github.com/user-attachments/assets/58c4fe8c-cd13-4dc4-aac5-f08a17958891" />


| Step | What happens | Protocol / port |
| --- | --- | --- |
| 1. DNS | Ask "what is the IP address of this name?" | UDP, port 53 |
| 2. TCP handshake | SYN, SYN-ACK, ACK | TCP |
| 3. TLS (HTTPS only) | Both sides agree on a secret key, the server shows its certificate | port 443 |
| 4. HTTP | `GET` request, then `200 OK` and the page | port 80 (HTTP) or 443 (HTTPS) |
| 5. Close | FIN, ACK | TCP |

## Commands I used

| Command | Question it answers |
| --- | --- |
| `ping name` | Is it reachable, and how fast? (ICMP, no ports) |
| `nslookup name` | What IP does this name have? |
| `curl -v url -o NUL` | Can I get a page, and what did we say to each other? |
| `tracert name` | Which routers are on the road? |
| `ipconfig /all` | My IP, gateway and DNS servers |

## Wireshark filters I used

`icmp`, `dns`, `dns.qry.name contains "word"`, `tcp.port == 80`, `tcp.port == 443`, `tls`, and Ctrl + F to search for a string.

## Key facts

- An **IP address** gets a packet to a computer. A **port** gets it to the right program.
- `10.x.x.x` and `172.20.x.x` are **private** addresses. They only work inside a local network.
- The **gateway** is the first router my packets go to.
- **TTL** goes down by 1 at each router, and at 0 the packet is dropped. `tracert` uses this on purpose.
- A DNS answer can be an address, "No such name", or "the name exists but there is no record of that type" (for example `AAAA` with no IPv6 address).
- `ping 1.2.3.4` needs no DNS, because there is no name to look up.

## What I saw myself

### Ping
The reply came back in about 26 ms, with `ttl=52`.

<!-- add your screenshot: ![ping](ping.png) -->

### DNS
For a fake website, the DNS server answered "No such name", so ping never sent a packet to any web server.

<!-- add your screenshot: ![dns](dns.png) -->

### TCP handshake and HTTP
A `curl` to Baidu showed SYN, SYN-ACK, ACK, then `GET / HTTP/1.1`, then `200 OK`, then FIN.

<!-- add your screenshot: ![handshake](handshake.png) -->

### HTTP vs HTTPS
I sent a fake password to the same site over both. Over HTTP, Wireshark found `GET /?user=ali&password=secret123` in plain text. Over HTTPS, the search found nothing.

<!-- add your screenshot: ![http](http.png) -->

### Tracert
`tracert baidu.com` took 14 hops. Hops 4 to 13 showed `* * *`, but the destination answered, so the road was not broken.

<!-- add your screenshot: ![tracert](tracert.png) -->

### Other things I noticed
- `baidu.com` and `www.baidu.com` gave **different IP addresses**. `www.baidu.com` is an alias (CNAME) of another name.
- Baidu's certificate was signed by GlobalSign.

## What HTTPS hides, and what it doesn't

- **Hides:** the page, passwords, cookies.
- **Does not hide:** the IP addresses, the site name in the Client Hello (SNI), when I connected, and how much data was sent.

## Mistakes and surprises

- Red rows in Wireshark were not errors. I started capturing in the middle of a connection.
- `curl` tried IPv6 first, so my IPv4 filter did not show it. Filtering by port is easier.
- I expected Baidu to redirect HTTP to HTTPS, but it answered `200 OK`.
- A search that finds nothing only means something if the packet was really in the capture. I checked by finding the Client Hello first.

## Self-check

1. Why did ping to a fake name send no packet to a website?
2. What is the difference between ping and curl?
3. What does the TCP handshake look like?
4. What could someone on the same Wi-Fi read over HTTP, and over HTTPS?
5. Why are `* * *` in tracert not proof of a broken road?


<img width="707" height="343" alt="site_checker" src="https://github.com/user-attachments/assets/f3f6a5af-e5c1-4483-a1bc-c755801b05d0" />
<img width="1387" height="432" alt="preview" src="https://github.com/user-attachments/assets/55d8873e-5416-43fd-8554-0fb85d3c49b1" />

# python-chat

Small Python programs I wrote while learning how networking works.
I wrote and ran everything myself on Windows, and checked it with Wireshark.

## 1. Chat (server.py and client.py)

A chat where many people can talk at the same time.

- **Server:** waits for people to connect. Each person gets their own thread, so nobody has to wait for someone else to leave.
  My first server had no threads. Client B had to wait until client A left, because the server could only do one thing at a time.
- **Lock:** makes sure two threads don't write to the shared list of clients at the same moment, so the messages don't get mixed up.
- **Client:** has a separate listener thread. The thread is not for typing. It frees the client to receive messages while I am also typing.
- **while loop and break:** the while loop keeps the conversation going. Without it, there would be one message and then it ends. "quit" uses break to leave.

How to run: open 2 or 3 terminals. Run `python server.py` in the first, then `python client.py` in the others.

## 2. Site checker (site_checker.py)

Visits some websites 5 times, measures how long they take, and saves everything to `results.csv`.

It tells me which step failed:

| What happened | Message | Step |
|---|---|---|
| Fake site `notarealwebsite123456.com` | name not found | DNS |
| `127.0.0.1:5999` (nothing there) | connection refused | handshake |
| Server answers with an error code | HTTP error | HTTP |
| No answer in 5 seconds | too slow | timeout |

### My results
- http://www.baidu.com: 200 in about 97 to 176 ms. Median: ___ ms
- https://www.baidu.com: 200 in about 150 to 334 ms. Median: ___ ms
- HTTPS was faster than HTTP, because of  the extra TLS handshake

### Something I learned
Sometimes the same site gave a different result in different tools (a 502 error, or "getaddrinfo failed"). The cause was my VPN/proxy. Python's urllib uses the system proxy settings, but curl normally doesn't.
So a failing check doesn't always mean the site is down. It can be my own setup.

## What I want to do next
Router and switch lab, then cloud networking.

# AWS VPC Lab: A Public Web Server and a Hidden Server

I built a small cloud network on AWS with two servers: a **web server** that anyone on the internet can reach, and a **hidden server** that only the web server can reach. I controlled who can talk to whom, tested it, and fixed what broke along the way.

> Everything here was built in my own AWS account (region: Sydney, `ap-southeast-2`), with free-tier servers, and deleted at the end.

## 1. Goal

I wanted to understand cloud networking by building it, not only reading about it: private addresses, subnets, routes, the internet gateway, and security groups. I had already practised the same ideas on my own computer (Packet Tracer and a Linux network lab), so this lab was the "real cloud" version.

## 2. Design
<img width="954" height="272" alt="vpc_R_map" src="https://github.com/user-attachments/assets/0ec0a2a0-ef1b-452b-a02d-90395d767e98" />


| Part | What I made | Why |
|---|---|---|
| VPC | `10.1.0.0/16` | My own private network inside AWS |
| Public subnet | `10.1.1.0/24` | Connected to the internet through the internet gateway |
| Second subnet | `10.1.2.0/24` | Planned as the private subnet (see section 5: it was created but not used) |
| Internet gateway + route table | `0.0.0.0/0` points to the gateway | This route is what makes a subnet "public" |
| Web server (EC2) | public IP, serves a page on port 80 | The part the internet can see |
| Hidden server (EC2) | no public IP, serves a file on port 9000 | The part that must stay hidden |

Both servers run a tiny Python web server (`python3 -m http.server`), started by a user-data script at first boot.


<img width="776" height="398" alt="image" src="https://github.com/user-attachments/assets/672928d8-a74d-46f0-9553-d5f915dab376" />


<img width="765" height="442" alt="image" src="https://github.com/user-attachments/assets/60fda1b7-4c2a-4886-b419-7cbc4f5a5d51" />


<img width="752" height="437" alt="web_server" src="https://github.com/user-attachments/assets/52b4487c-2570-440a-87f2-fd5ac0426923" />
<img width="743" height="404" alt="hidden_server" src="https://github.com/user-attachments/assets/114e15b0-e6f8-4d95-8c9d-711a83502872" />

## 3. The guards (security groups)

A security group is a guard attached to a server. **Inbound** says who may get in, **outbound** says where the server may go.


<img width="764" height="368" alt="seurity_web" src="https://github.com/user-attachments/assets/70de53bb-4450-4cbe-8ed0-36e6be7fb21d" />


<img width="772" height="376" alt="security_hidden" src="https://github.com/user-attachments/assets/544cb948-7dba-4a10-ba4a-afa5364e3113" />


| Guard | Inbound rule | Source | Why |
|---|---|---|---|
| web-sg | HTTP, port 80 | anywhere (`0.0.0.0/0`) | So anyone can open the website |
| web-sg | SSH, port 22 | my IP | So only I can log in |
| hidden-sg | TCP, port 9000 | `web-sg` (the group) | Only the web server may reach the hidden service |
| hidden-sg | SSH, port 22 | `web-sg` (the group) | So I can reach hidden by jumping through the web server |
| hidden-sg | ICMP | `web-sg` (the group) | So I can test with ping from the web server |

**Why a group as the source and not an IP?** The web server's IP can change (it did, after a stop/start). The group stays the same, so the rule keeps working and means "anything that belongs to the web server group".

Security groups are also **stateful**: if a connection is allowed in, the reply is allowed back automatically.

## 4. How I tested it

| # | Test | Result | Screenshot |
|---|---|---|---|
| 1 | Open `http://<web public IP>` in the browser | The page loads | <img width="710" alt="web_server_access" src="https://github.com/user-attachments/assets/f76ace63-56cd-46e7-a702-a849bfe85464" /> |
| 2 | `ss -ltn` on the web server | Listening on ports 22 and 80 | <img width="822" alt="ltn_web" src="https://github.com/user-attachments/assets/30b72dfa-ad76-4979-a467-aee7957da1ab" /> |
| 3 | `ss -ltn` on the hidden server | Listening on ports 22 and 9000 | <img width="859" alt="ltn_hidden" src="https://github.com/user-attachments/assets/18dba57b-1bbb-4791-ba36-576d3d38371d" /> |
| 4 | From the web server: `curl http://10.1.1.27:9000/secret.txt` | Returns the hidden file's text | <img width="552" alt="curl_hidden" src="https://github.com/user-attachments/assets/b82916a6-af20-403e-b79d-da35bef1c04c" /> |
| 5 | SSH to hidden through the web server (jump host) | I land on the hidden server's prompt | <img width="863" alt="ssh_jump" src="https://github.com/user-attachments/assets/70428a67-f50b-495c-aba5-0fade1bbc389" /> |


The jump command, run from my laptop, keeps my key file on my laptop and never copies it to a server:

```
ssh -i mylab_key.pem -o ProxyCommand="ssh -i mylab_key.pem -W %h:%p ec2-user@<web public IP>" ec2-user@<hidden private IP>
```

I also confirmed the reverse: the hidden server's address (`10.1.1.27`) is **private**, so my laptop has no route to it and a direct SSH from my laptop cannot work.

## 5. What went wrong and what I learned

**1. Timeout vs refused.**
- *Symptom:* the hidden server's port 9000 did not answer one morning, and later the web page timed out.
- *Cause:* a **timeout** means silence (something blocks or there is no road), while **refused** means the connection reached the server but no program was listening. The hidden server was refused because its program was not running.
- *Lesson:* the error type tells you which half is broken. `ss -ltn` on the server shows whether a program is really listening.

**2. Programs died after stop/start.**
- *Symptom:* after stopping the servers at night, the web page and port 9000 stopped working.
- *Cause:* my user-data script runs **only at first boot**, and the programs were started in the background, so they did not come back. The public IP also changed after the restart.
- *Fix:* I started the programs again by hand over SSH (`sudo nohup python3 -m http.server 80 ... &` for port 80, and the same without `sudo` for port 9000).
- *Lesson:* a program started by hand does not survive a reboot. A proper fix is a systemd service or a cron `@reboot` entry (see next steps).

**3. "My IP" and the VPN.**
- *Symptom:* SSH sometimes timed out even though the rule looked right.
- *Cause:* the "My IP" rule stores the address AWS sees. With a VPN, that is the VPN's exit address, and it can differ between my browser and my terminal.
- *Lesson:* create the rule and connect under the same network conditions, and re-select "My IP" when the address changes.

**4. SSH jump host and the key.**
- *Symptom:* `ssh -J` returned "Permission denied (publickey)".
- *Cause:* `-i` only applies to the final machine, so the first hop (the web server) got no key.
- *Fix:* use `ProxyCommand` so the key is used on both hops.

**5. Direct SSH to a private address.**
- *Symptom:* I ran `ssh` straight to the hidden server's private address (`10.1.1.27`) from my laptop. I expected a timeout, but SSH printed "REMOTE HOST IDENTIFICATION HAS CHANGED".
- *Cause:* a private address only exists inside its own network. My local network has a different machine using the same address, and its host key did not match the one I had saved from the real hidden server.
- *What I did:* I did not connect or override the warning, because it was someone else's device.
- *Lesson:* a `10.x.x.x` address means nothing outside its network, so the hidden server is only reachable through the web server. SSH's fingerprint check also protected me from connecting to the wrong machine.

**6. My subnet mistake.**
- *What happened:* I planned a public subnet (`10.1.1.0`) and a private subnet (`10.1.2.0`), but both servers ended up in `10.1.1.0`, because the subnet field defaulted to the first one when I launched the hidden server. I noticed it while writing up the lab.
- *Why it still held:* the hidden server is protected by having **no public IP** and by a security group that only accepts the web server's group. A subnet does not decide who can talk to a server, the security group does.
- *What I would do next time:* put hidden in a real private subnet with no route to the internet gateway. That adds a second layer, so even a mistake in a security group rule would not expose it.

## 6. Security ideas I practised

- **Least privilege:** open only the ports that are needed, and only to who needs them.
- **Hidden server:** no public IP, and a guard that accepts only the web server's group.
- **SSH limited to my IP**, not open to the whole internet.
- **Never copy the private key to a server.** Use a jump through the web server instead.
- **Defense in depth:** guards and addresses protect together, and a private subnet would add one more layer.

## 7. Cost and cleanup

- Free-tier-eligible instance size, and a budget alert in the billing settings.
- Cleanup: terminated both instances, deleted the security groups, deleted the VPC, then checked for leftover Elastic IPs and Volumes.


## 8. Next steps

- Make the web programs restart by themselves after a reboot (systemd or cron `@reboot`).
- Move the hidden server to a real private subnet.
- Try Session Manager instead of open SSH.
- Add HTTPS (certificate and a domain name).

