# Wifi_Phantom
# Abstract 
 Wi-Fi Phantom is a proof-of-concept research platform demonstrating how attackers can exploit vulnerable wireless networks to facilitate credential theft and persistent compromise, and critically how defenders can measure and mitigate such threats in a controlled environment. The modelled attack chain begins with a d-authentication event to induce client disconnects and capture anonymized traffic characteristics across multiple Wi-Fi networks, which informs follow-on social-engineering techniques. The platform specifically demonstrates how common weaknesses such as unprotected Wi-Fi profiles with automatic-join behavior and weak browser security materially increase victim susceptibility, enabling an attacker to steer devices toward malicious captive portals or exploit insecure client behaviors in a lab setting. A captive-portal style landing page, hosted on cloud object storage (e.g., S3) for controlled deployment, is used to simulate an “update” workflow and to measure user interaction and susceptibility. In the laboratory, payload delivery is represented by benign, instrumented artifacts that are triggered only with explicit consent and within an isolated testbed, enabling study of endpoint behaviour, persistence concepts, and detection telemetry without executing unauthorised code. The platform illustrates persistence concepts similar to known Trojans (registry modification, covert embedding) and evaluates polymorphic variability against signature-based detection, while simulated, encrypted exfiltration channels are used solely to assess monitoring gaps. Built-in ethical safeguards documented consent, scope enforcement, anonymization, and a kill-switch ensure responsible research. Results underscore combined risks from wireless exploitation and social engineering and reinforce the importance of WPA2, 802.11w/PMF, anomaly detection, and proactive endpoint hardening.



Algorithm
Step 1: Start

Step 2: Setup the fake AP

•	By using ESP32 to create a fake AP with the same SSID as the target network.

Step 3: De-authenticate clients from the legitimate AP (optional but common)

•	Use a tool like aireplay-ng to send deauthentication packets to clients connected to the legitimate AP.

Step 4: Intercept traffic and serve a fake EXE

•	Set up a web server on the fake AP to serve a malicious EXE (e.g., a fake login application that steals credentials).

•	Alternatively, use DNS spoofing to redirect users to a malicious website that hosts the EXE.

Step 5: Deliver the EXE and steal credentials

•	The EXE could be a fake software update or a required "login client" for the network.

•	Once the user runs the EXE, it collects credentials (and possibly other information) and sends them to the attacker.

Step 6: Clean up and maintain persistence (if desired)

•	The attacker may want to maintain access to the victim's machine or network. The attacker collects the credentials and can use them for further attacks.

Step 7: End


Setting up the project 

Step 1: Setting up the Rogue Access Point
•	Attacker creates a rogue Wi-Fi hotspot with a legitimate-sounding SSID (e.g., Free_Airport_WiFi, Cafe_Guest).
•	The hotspot is set up using a device like a Raspberry Pi, Wi-Fi Pineapple, or laptop with dual Wi-Fi cards.
•	Attacker enables Internet sharing to make the Wi-Fi seem real, but all traffic routes through their machine.

Step 2: Captive Portal & Forced Redirect
•	Attacker uses tools like hostapd (for AP), dnsmasq (for DHCP/DNS), and iptables (for traffic routing).
•	When a user connects, the attacker intercepts their HTTP requests and redirects them to a fake “Windows Update” page (served locally).
•	This is done by:
o	Redirecting all port 80/443 traffic to the attacker’s local web server.
o	Using a captive portal technique to force a splash page before "Internet access."

Stage 3: Social Engineering Payload Delivery
•	The fake “Windows Update” page mimics the official Microsoft style.
•	It includes a message like: “Your system is missing critical updates. Please install this security patch to continue.”
•	A downloadable .exe file is offered (named WindowsUpdateAssistant.exe or similar).
•	The .exe is actually a custom Remote Access Trojan (RAT) or a commodity malware (e.g., Quasar RAT, Covenant, Metasploit payload) disguised as an update.

Stage 4: Malware Execution & Persistence
•	Once the victim runs the .exe:
o	Dropper phase: The installer extracts the real malware into %AppData% or %ProgramData%.
o	Persistence: Achieved via Registry Run keys, scheduled tasks, or service installation.
o	Privilege Escalation: May exploit vulnerabilities (e.g., via getsystem in Metasploit) to gain admin rights.
o	Evasion: Disables antivirus, uses process hollowing, or encrypts C2 communication.

Stage 5: Information Stealing & Exfiltration
The malware performs data harvesting:
•	Keystroke logging: Logs all keyboard input (passwords, messages).
•	Screen capture: Takes periodic screenshots or live video via ffmpeg/DirectX capture.
•	Audio recording: Uses Windows audio APIs to record from microphone.
•	File system enumeration: Searches for documents, images, archives.
•	Credential theft:
o	Extracts saved passwords from browsers (using tools like Mimikatz or LaZagne-style modules).
o	Dumps Windows credential manager and stored Wi-Fi passwords.
•	Network sniffing: Captures nearby network traffic if possible.

Stage 6: Data Exfiltration
•	The malware calls back to the attacker’s C2 server (could be a VPS, Tor hidden service, or dynamic DNS).
•	Exfiltration methods:
o	Encrypted HTTP/HTTPS POST requests.
o	DNS tunneling.
o	Steganography inside innocent-looking image uploads.
•	Attacker can send remote commands: start/stop recording, upload files, deploy further malware.
