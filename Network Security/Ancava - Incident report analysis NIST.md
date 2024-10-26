Incident report analysis
Instructions
As you continue through this course, you may use this template to record your findings after completing an activity or to take notes on what you've learned about a specific tool or concept. You can also use this chart as a way to practice applying the NIST framework to different situations you encounter.
Summary
Due to a recent distributed denial of services (DDoS) attack, the organization’s internal network services were compromised for two hours until it was resolved. This attack was caused by a flood of incoming ICMP packets, preventing access to any network resources. The incident management team responded by blocking incoming ICMP packets and stopping all non-critical network services offline in order to restore critical network services.
Identify
Malicious actor/s targeted the company’s network through an ICMP flood attack. All the entire internal network was compromised. Critical services needed to be secured and restored. The actors exploited an unconfigured firewall.
Protect
The internal network was protected by implementing a new firewall rule to limit the rate of incoming ICMP packets. The network security team also implemented an IDS/IPS system.
Detect
The network security team installed a new network monitoring software to detect abnormal traffic patterns and configured a source IP address verification on the firewall to check for spoofed IP addresses on incoming ICMP packets.
Respond
After recovering from the incident, the cybersecurity team designed a new action plan for responding to this type of attack. In the first place, the non affected systems would be isolated and contained to prevent the attack from spreading. If possible, critical network systems affected by the attack will be recovered. Once the attack is contained, by blocking news ICMP packets too, the team analyzed network logs to check abnormal or malicious activities. Finally, the incident will be reported to the appropriate superior or legal figure.
Recover
To recover from a DDoS attack by ICMP flooding, the access to the network needs to be restored to regular behavior. Then, the non-critical services will be stopped, to prevent unnecessary traffic. At this moment, the critical network services will be restored.  Finally, once the attack is fully stopped and controlled, all the network systems can begin working correctly. The implementation of a new firewall rule to limit the rate of incoming ICMP packets and an IDS/IPS system should help block incoming ICMP flooding attacks.




Reflections/Notes:



