# Reverse Engineering LED Sign Control Protocols: A Technical Deep-Dive

## When Automation Requires Understanding Proprietary Systems

**A case study in protocol analysis and practical problem-solving**

---

## The Challenge: Integrating Separate Alert Systems

An organization had large outdoor LED displays at multiple locations that displayed announcements and information. They also had an existing emergency notification system that would send alerts during critical situations - lockdowns, severe weather, etc.

The issue? These systems didn't communicate with each other.

During an emergency, the workflow required:
1. Triggering the alert via the notification system
2. Manually logging into the LED sign management software
3. Manually selecting and displaying the appropriate emergency message
4. Manually reverting to normal programming afterward

This created a potential delay during time-sensitive situations and introduced the possibility of human error.

## Initial Research: Exploring Options

After contacting the LED sign vendor about automation capabilities, it became clear that their software only supported manual control through their desktop application. There was no API, webhook support, or any programmatic control method available - even as a paid option.

This presented an interesting technical challenge: Could the signs be controlled programmatically despite the lack of vendor support?

The answer would require understanding how the sign control protocol actually worked at the network level.

## The Approach: Protocol Analysis and Reverse Engineering

Network protocol analysis seemed like a promising approach. Using Wireshark, I captured the traffic between the vendor's management software and the LED signs to understand how they communicated.

### Phase 1: Reverse Engineering the Protocol

The signs communicate over TCP on port **16603**. The vendor's software sends binary packets that look like absolute gibberish at first glance:

```python
loginSIGNAHexstring=b'\x41\x56\x4f\x4e\x01\x00\x00\x00\x51\x52\
\x00\x00\x00\x00\x00\x00\x52\x00\x00\x00\x00\x00\x2a\x02\x7b\x22\
\x73\x6e\x22\x3a\x22\x42\x5a\x52\x41\x34\x38\x32\x36\x34\x57\x30\
\x45\x36\x30\x30\x30\x36\x35\x31\x33\x22...'
```

But decoding those hex bytes revealed something interesting - **JSON!** The proprietary protocol was essentially JSON wrapped in a binary envelope.

```json
{
  "sn": "BZRA48264W0E6000XXXX",
  "username": "admin",
  "password": "[redacted]",
  "loginType": 0
}
```

Understanding this structure made it possible to craft custom control commands.

### Phase 2: Building the Automation System

With the protocol understood, I developed two Python scripts to handle the automation:

**Script #1: Webhook Listener (`wsl_prod.py`)**
- Listens for incoming HTTP requests from the alert system
- Parses JSON payloads containing sign ID, message type, and duration
- Spawns the sign control script with appropriate parameters
- Uses threading to handle multiple concurrent requests

**Script #2: Sign Controller (`signctrl_prod.py`)**
- Establishes TCP socket connections to LED signs (port 16603)
- Handles authentication using the reverse-engineered protocol
- Retrieves available message programs dynamically
- Switches to specified message, waits for duration, then restores normal programming
- Includes error handling and connection validation

### Phase 3: Deployment

The system runs on a dedicated Linux server that automatically starts the webhook listener on boot:

```bash
#!/bin/bash
openvt -f -c 7 /usr/bin/python3 /signctrl/wsl_prod.py
```

The complete workflow:
1. Alert system sends webhook to listener
2. Sign switches to emergency message (typically < 2 seconds)
3. Message displays for specified duration
4. System automatically restores normal programming

This eliminated manual intervention and reduced the response time significantly.

## The Technical Details (For Fellow Nerds)

### Protocol Structure

The sign protocol has a simple structure:
- **Header:** `AVON` (literally just ASCII "AVON")
- **Command bytes:** Various control codes
- **Payload length:** Variable
- **Payload:** JSON wrapped in binary

### Authentication Flow

1. Send login command with serial number + credentials
2. Receive auth token
3. Send "list programs" command
4. Parse response to find message identifiers
5. Send "switch program" command with identifier

### Error Handling

The production version includes:
- Connection timeout handling
- Response validation (minimum bytes received)
- JSON parsing error recovery
- Multi-sign support (TMS and LES)
- Automatic rollback if something fails

### Dynamic Message Discovery

An important design decision was to query the sign for available programs rather than hardcoding message identifiers. This makes the system resilient to changes in the sign's configuration:

```python
for entry in parsed['programInfos']:
    if entry['name'] == 'SIGNAmsg1':
        SIGNAmsg1_id = entry['identifier']
```

If administrators modify messages through the vendor software, the automation system adapts automatically without requiring code changes.

## Reflections and Lessons Learned

This project highlighted several interesting aspects of working with proprietary systems and automation:

### Technical Insights

**1. Protocol Simplicity**
Many proprietary protocols are simpler than they appear. This particular system used standard JSON wrapped in a custom binary header - once identified, the rest became straightforward.

**2. The Value of Network Analysis**
Wireshark and similar tools are invaluable for understanding how systems communicate. Being able to observe actual traffic patterns makes reverse engineering much more approachable.

**3. Defensive Programming**
When working with hardware you don't fully control, robust error handling is essential. The production code includes connection timeouts, response validation, and graceful failure modes.

### Practical Considerations

**4. When Vendor Solutions Don't Exist**
Sometimes the functionality you need simply isn't available - at any price. In these cases, reverse engineering may be the only path forward if you have the technical capability and proper authorization.

**5. Cost-Benefit Analysis**
Custom solutions become especially attractive when:
- No vendor solution exists for your use case
- The organization has technical capability in-house
- The requirements are well-defined and stable
- The risk can be managed appropriately

**6. Documentation Matters**
Documenting both the protocol analysis and the implementation proved crucial for maintenance and troubleshooting. Future administrators need to understand how the system works.

## Key Takeaways

**For Technical Professionals:**
- Network protocol analysis is an accessible skill with the right tools
- Start with traffic capture, look for patterns and recognizable data structures
- Many "proprietary" protocols use standard formats (JSON, XML, etc.) with custom wrappers
- Test thoroughly in non-production environments before deployment
- Document everything - protocol details, decision rationale, and system architecture

**For System Administrators:**
- Consider total cost of ownership, including licensing and integration costs
- Evaluate whether in-house solutions are viable for your organization
- Balance the benefits of vendor support against flexibility needs
- Maintain good relationships with technical staff who can build custom solutions

**For Security Researchers:**
- This type of analysis is excellent practice for understanding how systems work
- Reverse engineering for interoperability is generally protected under fair use
- Always operate within the bounds of your authority and applicable laws
- Document your methodology - it's valuable for future projects

## Results

The system has operated reliably in production with:
- Automated emergency message activation (functionality that didn't previously exist)
- Sub-second response times
- Automatic restoration of normal programming
- Support for multiple sign locations
- No manual intervention required during alerts

This project created automation capabilities that weren't available through any vendor-provided solution, demonstrating that reverse engineering can unlock functionality beyond what manufacturers offer.

---

## Technical Appendix

### Key Code Snippets

**Webhook Listener (Simplified):**
```python
def handle(conn_addr):
    conn = conn_addr[0]
    data = conn.recv(1024)
    
    json_data = json.loads(getJSON(data))
    sign = json_data['sign']
    msgtype = json_data['type']
    duration = json_data['duration']
    
    # Spawn the sign controller
    Process = Popen(
        f'python3 /path/to/signctrl.py {sign} {msgtype} {duration}',
        shell=True
    )
    
    # Send HTTP 200 OK
    conn.send(b"HTTP/1.1 200 OK\r\n\r\n")
```

**Sign Control Flow:**
```python
# 1. Connect and authenticate
s.connect((HOST_IP, HOST_PORT))
s.send(loginCMD)

# 2. Switch to emergency message
s.send(playSolutionHexstring)

# 3. Wait for specified duration
time.sleep(WAIT_TIME)

# 4. Reconnect and restore normal programming
s.connect((HOST_IP, HOST_PORT))
s.send(loginCMD)
s.send(origSolution)
```

### Network Security Considerations

The signs operate on an isolated internal network segment with appropriate firewall rules. When implementing custom control systems, proper network segmentation and access controls are essential security measures.

---

## Conclusion

This project demonstrates that reverse engineering proprietary protocols is an accessible skill that can solve real-world integration challenges. With network analysis tools, patience, and systematic investigation, it's possible to understand how systems communicate and build custom solutions where needed.

The key is approaching the problem methodically:
1. Observe and capture actual communications
2. Identify patterns and data structures
3. Test hypotheses incrementally
4. Build robust, well-documented implementations
5. Deploy with appropriate safeguards

Whether you're integrating legacy systems, building automation tools, or just curious about how things work, protocol analysis is a valuable skill that opens up possibilities beyond vendor-provided solutions.

---

*This case study is presented for educational purposes. When reverse engineering systems, always ensure you have proper authorization and operate within applicable laws and regulations.*

*Interested in similar technical deep-dives? Feel free to reach out with questions or share your own protocol analysis experiences.*
