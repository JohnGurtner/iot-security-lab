# IoT Security Lab

> Registered an IoT device on AWS IoT Core with X.509 certificates, an IoT policy and MQTT, connected a
> real Raspberry Pi to the cloud, then switched to the attacker's side and cracked the Pi's SSH password
> with a Hydra dictionary attack.

*CNIT 271 (Cybersecurity Fundamentals II), Lab 4. The Raspberry Pi portion was done as a six-person group.*

## Lab setup
- AWS IoT Core: Things, device certificates, IoT policies, MQTT test client
- Raspberry Pi running Raspberry Pi OS with the AWS IoT Device SDK (Python)
- Kali Linux for the attack (`crunch`, `hydra`)

## Procedure

### Part 1: Register a secure device identity
1. In **AWS IoT Core → Manage → Things → Create thing**, created `group37_thing1` and gave it attributes.
2. Chose **Auto-generate a new certificate**. AWS issues an **X.509 device certificate** plus a
   public/private key pair, signed by the Amazon root CA.
3. Created **`group37_policy`** (JSON) to define which MQTT actions the device may perform, and
   **attached it to the certificate**.
4. Downloaded the certificate, keys and root CA, and confirmed the Thing showed the right certificate and policy.

**How the pieces fit:** the certificate handles **authentication** (proving which device this is). The
IoT policy on the certificate handles **authorization** (what the device can publish or subscribe to).
IoT policies are about device messaging actions, while IAM policies control AWS API access for users,
roles and services.

### Part 2: MQTT publish/subscribe
1. In the **MQTT test client**, subscribed to the topic `Home1/Room1/Lightbulb`.
2. Published `"Lightbulb is on"` to the same topic and watched it arrive on the subscription.

MQTT goes through a **broker**. Publishers and subscribers never talk to each other directly, which
suits large numbers of low-power sensors. Security comes from **TLS** on the connection, **X.509
mutual authentication**, and **topic-level authorization** in the policy.

### Part 3: Connect a real device
1. Booted the Raspberry Pi and checked it could reach AWS with `ping`.
2. In IoT Core → *Connect one device*, chose the existing Thing, with **Linux** as the platform and
   **Python** as the SDK.
3. Downloaded the connection kit to the Pi, unzipped it, and ran it:
   ```bash
   unzip <connection-kit>.zip
   ./start.sh
   ```
4. The script installs the SDK and runs a sample that connects over **MQTT/TLS** with the device
   certificate, private key and root CA, then **publishes messages** that showed up in the IoT Core console.

### Part 4: Attack the device
1. Confirmed SSH was running on the Pi (`systemctl status ssh`).
2. On Kali, generated a targeted wordlist with **`crunch`**.
3. Ran a **Hydra** dictionary attack against the Pi's SSH service with that wordlist. It recovered the
   username and password.
4. Logged into the Pi over SSH with the cracked credentials.

**The point:** the certificate setup protected the device-to-cloud channel. The device itself fell to a
guessable SSH password. In IoT, the weakest link is often a default or weak local credential, not the crypto.

## Dictionary vs brute force
| | Dictionary | Brute force |
| --- | --- | --- |
| How | Tries likely passwords from a list | Tries every possible combination |
| Speed | Fast when users pick weak passwords | Slow, but always succeeds eventually |
| Best when | Human-chosen passwords, limited time | Short or numeric secrets (PINs) |

## Mitigations
- **SSH keys only**: `PasswordAuthentication no` in `sshd_config`
- **Lock out or throttle** after failed attempts (fail2ban, `MaxAuthTries`)
- **Strong password policy** and **no default credentials**, changed when the device is provisioned
- **MFA** where possible, and keep management services off untrusted networks

> These same SSH brute-force patterns are what the
> [log-triage-toolkit](https://github.com/JohnGurtner/log-triage-toolkit) and
> [sigma-detections](https://github.com/JohnGurtner/sigma-detections) projects detect.

## Skills demonstrated

AWS IoT Core · X.509 device authentication · IoT policies · MQTT pub/sub · TLS · AWS IoT Device SDK ·
Raspberry Pi · password attacks (crunch, Hydra) · IoT threat mitigation

## Screenshots

See [`screenshots/`](screenshots/) for the Thing, certificate and policy setup, the MQTT test client, the Pi
connection kit run, and the Hydra crack and SSH login.
