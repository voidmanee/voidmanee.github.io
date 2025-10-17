# voidmanee's Technical Blog

Tinkerer, Security researcher and developer documenting technical deep-dives, reverse engineering projects, and real-world development challenges.

**[About](about.md)** | 20+ years professional experience | From bare metal to blockchain

---

## Posts

### 🔌 [Low-Level WebSocket Packet Monitoring in C](websocket_monitoring_c.md)
*Intercepting network traffic at wire speed*

Building a raw socket-based WebSocket packet monitor in C to analyze network traffic at the packet level with microsecond latency - bypassing the OS network stack entirely.

**Topics:** C Programming, Network Programming, Raw Sockets, WebSocket Protocol, Performance Optimization

**Key highlights:**
- Raw socket implementation for packet interception
- Manual parsing of IP, TCP, and WebSocket protocol layers
- Achieving ~30 microsecond processing latency
- Real-time packet size analysis before application layer

---

### 🤖 [Building Your Own DeFi Brain: A Deep Dive into a NodeJS Crypto Trading Bot](defi_trading_bot.md)
*Algorithmic trading meets blockchain automation*

Dissecting a real-world Node.js trading bot that executes automated trades on PancakeSwap DEX using real-time price monitoring and smart contract interactions.

**Topics:** Web3, Blockchain, DeFi, Node.js, Smart Contracts, Algorithmic Trading

**Key highlights:**
- Real-time WebSocket blockchain monitoring
- On-chain price oracle implementation
- PancakeSwap Router smart contract integration
- Automated transaction execution with gas optimization

---

### 📡 [Reverse Engineering LED Sign Control Protocols](blog_post_led_signs.md)
*A case study in protocol analysis and practical problem-solving*

When automation requires understanding proprietary systems - reverse engineering LED sign communications to build custom control software without vendor API access.

**Topics:** Protocol Analysis, Wireshark, Python Socket Programming, Network Security, Reverse Engineering

**Key highlights:**
- Analyzing proprietary binary protocols
- Decoding wrapped JSON communications
- Building webhook-based automation
- TCP socket programming in Python

---

### 💻 [Building a Device Management System: A Development Journey](device_management_system_blog.md)
*From spreadsheets to Django - creating custom inventory software*

Full-stack development case study covering architecture decisions, debugging complex data integrity bugs, and building production software for real users.

**Topics:** Django, PostgreSQL, Full-Stack Development, Data Integrity, Debugging, Testing

**Key highlights:**
- Custom Django application managing 5000+ devices
- Debugging dual-assignment data integrity bug
- Performance optimization with large datasets
- Testing strategies and validation patterns

---

## About

This blog focuses on:
- 🔍 Reverse engineering proprietary systems
- 🛠️ Building practical solutions to real problems
- 🐛 Deep-dive debugging and root cause analysis
- 🔐 Security research and protocol analysis
- 📚 Sharing lessons learned from production systems

## Skills Demonstrated

**Languages & Frameworks:**
- Python (Django, Socket Programming)
- SQL (PostgreSQL)
- JavaScript
- Bash scripting

**Security & Analysis:**
- Network protocol analysis (Wireshark)
- Reverse engineering
- Bug hunting and vulnerability research
- Security testing

**Development:**
- Full-stack web development
- Database design and optimization
- Testing and validation
- Production deployment

**Tools:**
- Django ORM
- Git version control
- Linux system administration
- Network debugging tools

---

## Connect

- GitHub: [@voidmanee](https://github.com/voidmanee)

*Interested in collaboration, security research opportunities, or technical discussions? Feel free to open an issue or reach out.*

---

<sub>All content is provided for educational purposes. Specific implementation details and identifying information have been generalized to protect privacy and organizational security.</sub>
