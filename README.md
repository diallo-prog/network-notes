# network-notes
Notes and Wireshark captures from learning how the network works.

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/cfbb2e14-32af-40f5-ad9a-f0a49e8d0ce8" />
<img width="1918" height="1027" alt="image" src="https://github.com/user-attachments/assets/091daed5-3c19-4981-b2ee-430f3c7d62b6" />
A difference between an https and http

# How a web request works (my Wireshark notes)

Notes and captures from learning how the network works, weeks 1-2 of my networking plan. I tested everything on my own computer and my own network.

## The 5 steps of opening a website

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
