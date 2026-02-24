# WhatsApp Setup for OpenClaw

## Step 1 — Get a physical prepaid SIM

Buy any cheap prepaid SIM at a supermarket or phone shop (Proximus, Orange, Base in Belgium). **Do not use a VoIP or virtual number** — WhatsApp blocks them aggressively. You don't need data or minutes on the SIM, just enough credit to receive one SMS. Put it in any spare phone or a cheap handset.

---

## Step 2 — Register the SIM on WhatsApp

1. Insert the SIM in a phone
2. Install WhatsApp, open it, tap **Agree and Continue**
3. Enter the SIM's phone number → receive verification SMS → enter the code
4. Set up a profile name (e.g. *"IDEA Assistant"*) — this is what people see in the group
5. WhatsApp is now registered on that number

You only need the phone for this step and for the QR scan. After that the phone can sit in a drawer as long as it has occasional power — WhatsApp's multi-device keeps the session alive without the phone being online.

---

## Step 3 — Configure OpenClaw

Add the WhatsApp channel to your `openclaw.json`:

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "pairing",
      "allowFrom": ["+32XXXXXXXXX"]
    }
  }
}
```

`allowFrom` is your own personal WhatsApp number — so only you can DM OpenClaw directly. Field workers and the supporters group work differently (they interact via a group, not DMs).

Edit it the same way as any other openclaw.json config:

```bash
sudo nano /var/lib/docker/volumes/openclaw_openclaw-data/_data/openclaw.json
sudo docker restart openclaw-gateway
```

---

## Step 4 — Scan the QR code

Run the login command inside the OpenClaw container:

```bash
sudo docker exec openclaw-gateway node dist/index.js channels login --channel whatsapp
```

A QR code appears in the terminal. **Scan it quickly** — it expires in about 60 seconds.

On the dedicated WhatsApp phone:
> **WhatsApp → Settings → Linked Devices → Link a Device** → point camera at QR code

WhatsApp will briefly disconnect and reconnect — that's normal. OpenClaw is now linked.

---

## Step 5 — Add OpenClaw to your groups and contacts

- **Supporters group**: open the group on your phone, add the dedicated number as a participant
- **Your own phone**: save the dedicated number as a contact (*"IDEA Assistant"* or *"OpenClaw"*)
- **Field workers**: share the number with them, they save it and can message it directly

---

## What you'll spend

| Item | Cost |
|------|------|
| Prepaid SIM | €5–10 one-time |
| Cheap spare handset (if you don't have one) | €15–30 second-hand |
| Ongoing | Nothing |

---

## Sources

- [OpenClaw WhatsApp docs](https://docs.openclaw.ai/channels/whatsapp)
- [Getting Started with OpenClaw and WhatsApp — MarkTechPost](https://www.marktechpost.com/2026/02/14/getting-started-with-openclaw-and-connecting-it-with-whatsapp/)
- [OpenClaw WhatsApp Setup — Macaron](https://macaron.im/blog/openclaw-whatsapp-setup)
