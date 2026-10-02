---
name: manage-phones
description: List, register, and assign Voice Logica VoIP phones and numbers to AI agents, including SIP, extensions, inbound DIDs, and PBX edge devices. Use when the user asks about DIDs, SIP phones, extensions, inbound numbers, registration, one-way audio, or why an agent has no phone.
---

# Manage Voice Logica phones

Use Voice Logica MCP phone tools. Do not invent numbers, SIP credentials, or agent IDs. Never print SIP passwords in chat.

## MCP tools

- `get_voip_phones` / `create_voip_phone` / `update_voip_phone` / `delete_voip_phone`
- `activate_voip_phone` / `deactivate_voip_phone`
- `pbx_query` (action `get_edge_installer`) / `pbx_query` (action `get_edge_devices`) / `set_edge_device_forward` / `remove_edge_device_forward`

## Before changing anything

1. List phones and list agents. Match by name, then use real IDs.
2. If the user names a number, find that exact registration first.
3. Confirm the target agent before assigning or moving a number.
4. If inbound calls never appear in Voice Logica at all (zero Call IDs in the window), traffic never reached the platform — treat it as carrier / portability / PBX routing first, not a prompt bug.

## Three phone paths — do not swap them

| Path | When | Transport |
| --- | --- | --- |
| **Direct** SIP | Hosted / cloud PBX (public SIP) | Internet SIP. Register the extension in Phones. |
| **Via edge device** | On-prem PBX on a private LAN (Grandstream, FreePBX, Alcatel, Panasonic, …) | WireGuard **UDP 51820** + control TCP 443. UI: Phones → Edge Devices. |
| **On-prem SSH tunnel** | Private **HTTP ERP / API**, not phones | AI → Tunnels. See `manage-erp`. Wrong product for SIP/RTP. |

Yuboto / cloud numbers usually use **Direct**. A Yuboto PBX **integration** (API key under Integrations) is not the same as a SIP phone and not an Edge Device.

A third-party hosted PBX (Vodafone One Net, Cosmote, …) is configured on **their** portal. Voice Logica cannot set their forwarding rules. They must send the public number to the AI DID / extension themselves.

## Common jobs

**Why does this agent have no phone?**
An agent with no DID usually needs a number from Phones, or a plan that includes DIDs (`did` / `free-did`). Check phones first, then the agent, then `billing_query` (action `get_subscription`). Demo / No Plan / Inactive plans often lock number CTAs.

**Assign a number to an agent.**
List free or existing phones, pick one, attach it to the agent ID, then confirm the agent shows that number. Suggest a test inbound call.

**New Direct SIP / extension registration.**
Ask the user for the values their PBX issued. Typical fields:

- SIP server URL / domain
- Port (usually **5060**)
- Extension / username and Auth ID
- Password (user pastes into the Voice Logica form or MCP write — do not echo it)
- Codec (usually **OPUS** or **G.711** — must match the PBX)

They may need to whitelist the Voice Logica SIP/media IP on the PBX and allow **UDP 5060** plus RTP (typically **10000–20000**). Registration refresh is often **3600** seconds. Do not invent the Voice Logica IP — use the value they were given or ask Voice Logica.

What they provide back: server, port, extension/username, auth ID, password, codec. Enter those in Phones. Then `create_voip_phone` / `activate_voip_phone`. Confirm registration, a test dial, and two-way audio.

**Internal extension as a transfer destination.**
Register that extension as a VoIP phone first, then reference it in the agent transfer prompt (`edit-voice-agents`). Without credentials in Phones, extension transfer fails even if Transfer Connection looks correct.

**Private on-prem PBX (Edge Device), done from the chat.**

1. `pbx_query` (action `get_edge_installer`) with `platform` (`windows` or `linux`, ask the user). Give them the `downloadUrl` (valid 4 hours) or the `command`, plus `steps` and `requirements`. The machine must sit on the PBX LAN; a VM NIC must be **bridged**.
2. Firewall: outbound **UDP 51820** and outbound **TCP 443** to the edge gateway. TCP 443 alone can look **Online** with **no calls / no audio**.
3. Poll `pbx_query` (action `get_edge_devices`) until `ready: true`. Pending devices are approved on read. `online: true` with `tunnelUp: false` = UDP 51820 blocked.
4. Ask for the PBX LAN IP, SIP port (Panasonic often **5060**, Alcatel OXO often **5059**) and the extension username/password. If the user does not know the IP, show `pbxCandidates` and confirm one.
5. `create_sip_trunk` with `host` = PBX LAN IP, `port`, `transport: "udp"`, `inboundAuthType`/`outboundAuthType: "registration"`, `username`, `password`. A private IP is routed through the edge device automatically (the only device, or the one on the PBX subnet; pass `edgeDeviceId` if several match). It waits for the device and registers before returning.
6. Disable **SIP ALG** at the site. Bidirectional UDP to the PBX IP is required (RTP uses dynamic ports; SIP 5060 alone is not enough).
7. Health check is three separate signals: **Online**, **Tunnel: up**, **PBX: reachable** (ping only; can be false where SIP works). Never report "Online" as working.

Do not ask the customer to open inbound SIP ports to the internet when Edge is the design.

If a test call returns busy with a working device, `billing_query` (action `get_subscription`) first. A plan with `seconds: 0` rejects the call as busy (`endCallReason` will say no seconds remaining). That is not an Edge fault.

**Transfers vs phones.**
Phones are the numbers and registrations. Transfer destinations and attended/blind live on the agent Transfer Connection tab (`edit-voice-agents`). Attended often fails on PBXs that decline SIP REFER — switch to blind; file a ticket if they still need attended.

**Portability / failover / "the AI died".**
Ask: which public number, exact window, did **any** Call IDs appear, what rang (desk phone / old PSTN / mobile failover). Call IDs present → AI received some traffic. Zero Call IDs → traffic never reached Voice Logica.

**AI minutes vs provider credits.**
AI minutes = agent talk time. Provider credits = PSTN / failover / outbound at the telephony provider. Both can look like "calls stopped". For Yuboto wallet top-up, send only https://services.yuboto.com/mynumber/ and support@yuboto-telephony.gr — do not invent their portal steps.

**Two landlines on one telephony account.**
Possible, but each DID needs a clean mapping to the correct agent/extension. Collect both numbers and the desired mapping.

**Hold after answer.**
After a human picks up a transfer, hold/resume is on the physical phone or softphone. Ring timeout (how long it rings before fallback) is a transfer setting, not hold.

## Registration vs audio (do not mix)

Work this tree in order. Registration up is not the same as audio working.

1. **Not registered** → credentials, SIP server/port, IP whitelist on the PBX, firewall UDP 5060. Ask them to confirm the extension appears in their PBX.
2. **Registered, calls never connect / agent does not answer** → PBX routing / time-conditions / portability, or the agent is disabled. If the window has **zero Call IDs**, traffic never reached Voice Logica.
3. **Registered, call connects, one-way or no audio** → codec mismatch or RTP blocked (typically UDP 10000–20000). On Edge: UDP 51820 blocked (Online without Tunnel: up).

Do not rewrite the prompt for a registration or RTP failure.

**Each outbound agent needs its own extension** (or port). Do not put two outbound agents on the same SIP extension. See `manage-calls-campaigns`.

## After a change

Say which number is now on which agent (or which edge device / SIP host). Suggest a test inbound call. Do not print secrets.
