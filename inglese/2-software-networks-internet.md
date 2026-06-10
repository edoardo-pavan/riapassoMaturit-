# DAY 2 — MICROLINGUA: SOFTWARE, NETWORKS, INTERNET

> Focus: technical vocabulary for the oral exam. The examiner will ask you to describe IT concepts in English.

---

## 1. UNIT 12 — COMPUTER SOFTWARE AND PROGRAMMING

### System Software vs Application Software

| | System Software | Application Software |
|---|---|---|
| **Purpose** | Manages hardware and provides a platform to run apps | Performs specific tasks for the user |
| **Examples** | Windows, macOS, Linux, Android, iOS, device drivers | Word, Chrome, Photoshop, Spotify |
| **Runs** | In the background | Directly used by the user |

### Programming Languages

| Language | Type | Typical Use |
|---|---|---|
| **Python** | High-level, interpreted | AI, data science, automation, web |
| **JavaScript** | High-level, interpreted | Web development (frontend AND backend) |
| **C#** | High-level, compiled (to IL) | Enterprise apps, games (Unity), ASP.NET Core |
| **Java** | High-level, compiled (to bytecode) | Android apps, enterprise, banking |
| **C/C++** | Mid-level, compiled | Operating systems, embedded, games, high-perf |
| **TypeScript** | Superset of JavaScript | Large-scale web apps |
| **SQL** | Query language | Database management |

### Programming Languages Most in Demand (2024-2025)
1. Python — AI, data science, automation
2. JavaScript / TypeScript — web development
3. Java — enterprise, Android
4. C# — .NET ecosystem, game dev
5. SQL — data management (always relevant)

### Key Vocabulary

| Term | Definition |
|---|---|
| **source code** | The human-readable code written by a programmer |
| **compiler** | A program that translates source code into machine code |
| **interpreter** | A program that executes code line by line without compiling first |
| **debugging** | Finding and fixing errors in code |
| **IDE** (Integrated Development Environment) | Software with tools for coding (VS Code, Visual Studio, IntelliJ) |
| **framework** | A reusable set of libraries for building apps (ASP.NET Core, React) |
| **open source** | Software whose source code is freely available to modify and share |
| **API** (Application Programming Interface) | A set of rules allowing different software to communicate |
| **version control** | System for tracking changes to code (Git) |
| **repository** | A central location where code is stored and managed (GitHub) |

### Software Safety

- **Testing**: Unit tests, integration tests, end-to-end tests
- **Code review**: Other developers check your code before merging
- **Static analysis**: Tools that analyze code without executing it
- **Continuous Integration (CI)**: Automatically build and test code on every commit
- **Security patches**: Updates to fix known vulnerabilities

### The Hidden Hero That Died in Disgrace

**Alan Turing** (1912-1954):
- British mathematician, logician, computer scientist
- Broke the German **Enigma code** during WWII — saved millions of lives
- Laid the foundations of computer science (**Turing Machine**, **Turing Test**)
- After the war, prosecuted for homosexuality (illegal in UK at the time)
- Chose chemical castration as an alternative to prison
- Died in 1954, officially a suicide
- Received a posthumous royal pardon in 2013

### Cloud Computing vs Edge Computing

| | Cloud Computing | Edge Computing |
|---|---|---|
| **Where data is processed** | Centralized data centers | Close to the data source (local devices/gateways) |
| **Latency** | Higher (tens/hundreds of ms) | Very low (< 10 ms) |
| **Use cases** | Web apps, storage, ML training | IoT, autonomous vehicles, smart factories |
| **Examples** | AWS, Azure, Google Cloud | Edge gateways, 5G edge servers |

### Women Pioneers in Computing

| Name | Contribution |
|---|---|
| **Ada Lovelace** | First computer programmer (wrote algorithms for Babbage's Analytical Engine, 1840s) |
| **Grace Hopper** | Developed the first compiler, created COBOL, coined the term "bug" |
| **Margaret Hamilton** | Led NASA's Apollo flight software team |
| **Hedy Lamarr** | Hollywood actress who co-invented frequency hopping (basis for Wi-Fi, Bluetooth) |

---

## 2. UNIT 14 — COMPUTER NETWORKS AND THE INTERNET

### How Computers Are Linked

- **Wired**: Ethernet (UTP/STP cable), fiber optic
- **Wireless**: Wi-Fi (802.11), Bluetooth, mobile networks (4G/5G)
- **Topologies**: star, mesh, bus, ring

### Brief History of the Internet

| Year | Event |
|---|---|
| 1969 | **ARPANET** — first network, 4 US universities connected (US Dept. of Defense) |
| 1970s | TCP/IP protocol developed |
| 1983 | TCP/IP becomes the standard for ARPANET |
| 1989 | **Tim Berners-Lee** invents the **World Wide Web** (HTTP, HTML, URLs) |
| 1990s | Internet opens to the public; first browsers (Mosaic, Netscape) |
| 2000s | Web 2.0 (social media, interactive web), broadband, Wi-Fi everywhere |
| 2010s | Mobile internet, cloud computing, IoT |
| 2020s | Web 3.0/4.0, AI, 5G |

### Internet Services

| Service | Description |
|---|---|
| **WWW** | World Wide Web — collection of hyperlinked documents |
| **Email** | Electronic mail (SMTP, POP3, IMAP) |
| **FTP** | File Transfer Protocol — transferring files |
| **VoIP** | Voice over IP (Skype, Zoom, WhatsApp calls) |
| **Streaming** | Real-time transmission of audio/video (Netflix, Spotify) |
| **Cloud storage** | Remote file storage (Google Drive, Dropbox) |
| **Social media** | Platforms for social interaction |

### How the Internet Works

1. You type a URL in your browser
2. **DNS** translates the URL to an IP address
3. Your request travels through **routers** across the Internet
4. **Packets** of data are split, routed, and reassembled (**TCP/IP**)
5. The server processes your request and sends data back
6. Your browser renders the webpage

### Web Addresses

```
https://www.example.com/path/page?query=value#fragment
└┬┘   └──┬───┘└──────┬──────┘└─path─┘└─query───┘└fragment┘
protocol  subdomain  domain   path    query string  anchor
```

### LAN — Local Area Network

A network covering a small area (home, office, school). Devices share resources (printers, files, internet connection) through a router/switch. Components: **NIC** (network card), **switch**, **router**, **cables/Wi-Fi**.

### Social and Ethical Problems of IT

| Problem | Description |
|---|---|
| **Privacy** | Companies collect vast amounts of personal data |
| **Digital divide** | Unequal access to technology (rich vs. poor countries, urban vs. rural, young vs. elderly) |
| **Cyberbullying** | Harassment through digital platforms |
| **Misinformation** | Fake news spreads faster than truth |
| **Addiction** | Social media and gaming designed to be addictive |
| **E-waste** | Disposal of electronic devices creates toxic waste |
| **Job displacement** | Automation replacing workers |
| **Surveillance** | Governments and corporations monitoring citizens |

### Online Dangers

- **Phishing**: fake emails trying to steal passwords
- **Malware**: malicious software (viruses, ransomware, spyware)
- **Identity theft**: someone steals personal info to commit fraud
- **Hacking**: unauthorized access to computer systems
- **Cyberstalking**: using the internet to harass someone

---

## SELF-CHECK QUESTIONS

1. Difference between system software and application software. Give 2 examples of each.
2. What is open source software? Name an example.
3. What is an API? Why is it important?
4. Who was Alan Turing? What did he do during WWII? Why did he "die in disgrace"?
5. Cloud vs Edge computing: key difference and a use case for each.
6. Name 3 women pioneers in computing and their contributions.
7. How did the Internet begin? Key dates: ARPANET, TCP/IP, World Wide Web.
8. What is a LAN? What devices connect to it?
9. Explain how the Internet works (DNS → routing → TCP/IP → response).
10. List 3 social problems of IT and 3 online dangers.
