# IoT Security Lab

> Stood up an authenticated IoT device on AWS IoT Core with X.509 certs and MQTT, then switched to
> the attacker's side and cracked SSH on a Raspberry Pi with a Hydra dictionary attack.

*CNIT 271 — Cybersecurity Fundamentals II, Lab 4. Hardware portion done as a group.*

## Summary

Two halves: first the **secure design** — registering an IoT "Thing," authenticating it with X.509
certificates and scoping it with an IoT policy, then exercising the MQTT publish/subscribe model.
Then the **offensive** half — demonstrating why weak device credentials are the soft spot of IoT by
brute-forcing SSH on a real Raspberry Pi.

## Lab setup

- AWS IoT Core (Thing, device certificate, IoT policy, MQTT test client)
- Raspberry Pi connected to AWS IoT via the Python device SDK
- Kali Linux for the attack (`crunch`, `hydra`)

## What I did &amp; found

| Area | Work | Security point |
| --- | --- | --- |
| Device identity | Auto-generated an X.509 cert + keys for the Thing | Certs uniquely identify devices and enable mutual TLS |
| Authorization | Attached a JSON IoT policy to the cert | IoT policies gate device **actions** (unlike broad IAM policies) |
| Messaging | Subscribed/published on `Home1/Room1/Lightbulb` | Pub/sub via a broker decouples publishers from subscribers |
| Transport | Pi → AWS IoT over MQTT + TLS using cert/key/CA | Confidentiality + authenticity on the wire |
| Attack | `crunch` wordlist → `hydra` SSH dictionary attack → login | Weak/guessable creds defeat all the crypto above |

**Mitigations I documented:** strong password policy, key-based SSH (disable password auth),
account lockout after failed attempts, and MFA — the same controls that would have stopped the Hydra run.

## Screenshots

See [`screenshots/`](screenshots/) for the Thing + policy creation, the MQTT test client, the Pi
connecting to AWS IoT, and the Hydra crack + SSH login.

## Skills demonstrated

AWS IoT Core · X.509 device authentication · MQTT pub/sub · TLS · password cracking (Hydra) ·
dictionary vs brute-force analysis · IoT threat mitigation
