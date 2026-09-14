1. *NMAP*:
	1. Scans a network to see the devices and services running
	2. Think of it like knocking on every door in a building to see which ones open and what's behind them.
	3. Fetches: Open ports, operating systems, software versions, network topology.
	4. Used for reconnaissance
2. *Metasploit*:
	1. Massive framework of pre-built exploits.
	2. Once you know the target's vulnerable software, Metasploit likely has a ready-made attack for it.
	3. Whether a vulnerability is actually exploitable, and gives you access if it is.
3. *Burp Suite*
	1. **What it does:** Intercepts and manipulates web traffic between a browser and a web server.
	2. **What it tells you:** How a web application handles inputs, authentication, sessions — and where it can be tricked.
	3. **In pen testing:** Specialized for _web application_ attacks — things like SQL injection, cross-site scripting, broken authentication.
4. *Wireshark
	1. *What it does:* Captures and reads all network traffic passing through an interface in real time — like a wiretap on the network.
	2. *What it tells you:* Exactly what data is being sent where, including passwords sent in plain text, unencrypted protocols, etc.
	3. *In pen testing:* Useful for _passive reconnaissance_ and catching credentials or sensitive data flowing across the network.*