---
name: manage-phones
description: List, register, and assign Voice Logica VoIP phones and numbers to AI agents, including SIP, extensions, inbound DIDs, and PBX edge devices. Use when the user asks about DIDs, SIP phones, extensions, inbound numbers, registration, one-way audio, or why an agent has no phone.
---

# Manage Voice Logica phones

Use Voice Logica MCP phone tools. Do not invent numbers, SIP credentials, or agent IDs. Never print SIP passwords in chat.

## MCP tools

- `get_voip_phones` / `create_voip_phone` / `update_voip_phone` / `delete_voip_phone`
- `activate_voip_phone` / `deactivate_voip_phone`
- `get_edge_devices` / `set_edge_device_forward` / `remove_edge_device_forward`

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
An agent with no DID usually needs a number from Phones, or a plan that includes DIDs (`did` / `free-did`). Check phones first, then the agent, then `get_subscription`. Demo / No Plan / Inactive plans often lock number CTAs.

**Assign a number to an agent.**
List free or existing phones, pick one, attach it to the agent ID, then confirm the agent shows that number. Suggest a test inbound call.

**New Direct SIP / extension registration.**
Ask the user for the values their PBX issued. Typical fields:

- SIP server URL / domain
- Port (usually **5060**)
- Extension / username and Auth ID
- Password (user pastes into the Voice Logica form or MCP write — do not echo it)
- Codec (usually **OPUS** or **G.711** — must match the PBX)

They may need to whitelist the Voice Logica SIP/media IP on the PBX and allow **UDP 5060** plus RTP (typically **10000–20000**). Registration refresh is often **3600** seconds.

Then `create_voip_phone` / `activate_voip_phone`. Confirm registration, a test dial, and two-way audio.

**Internal extension as a transfer destination.**
Register that extension as a VoIP phone first, then reference it in the agent transfer prompt (`edit-voice-agents`). Without credentials in Phones, extension transfer fails even if Transfer Connection looks correct.

**Private on-prem PBX (Edge Device).**

1. Customer IT installs the connector on the PBX LAN (Windows EXE, Linux/Pi token installer, or OVA). VM NIC must be **bridged** onto the PBX LAN.
2. Firewall: outbound **UDP 51820** to the edge gateway **and** outbound **TCP 443**. TCP 443 alone can look **Online** with **no calls / no audio**.
3. Disable **SIP ALG**. Bidirectional UDP to the PBX IP is required (RTP uses dynamic ports; SIP 5060 alone is not enough).
4. Approve the device. Push Config: PBX LAN IP + SIP port (Panasonic often **5060**, Alcatel OXO often **5059**).
5. Create the VoIP phone as **Via edge device**, pick the device, set the private PBX server IP, transport **UDP**.
6. Health check is three separate signals: **Online**, **Tunnel: up**, **PBX: reachable**. Never report "Online" as working.

Do not ask the customer to open inbound SIP ports to the internet when Edge is the design.

If a test call returns busy with a working device, `get_subscription` first. A plan with `seconds: 0` rejects the call as busy (`endCallReason` will say no seconds remaining). That is not an Edge fault.

**Transfers vs phones.**
Phones are the numbers and registrations. Transfer destinations and attended/blind live on the agent Transfer Connection tab (`edit-voice-agents`).

**Portability / failover / "the AI died".**
Ask: which public number, exact window, did **any** Call IDs appear, what rang (desk phone / old PSTN / mobile failover). Call IDs present → AI received some traffic. Zero Call IDs → traffic never reached Voice Logica.

**AI minutes vs provider credits.**
AI minutes = agent talk time. Provider credits = PSTN / failover / outbound at the telephony provider. Both can look like "calls stopped". For Yuboto wallet top-up, send only https://services.yuboto.com/mynumber/ and support@yuboto-telephony.gr — do not invent their portal steps.

**Two landlines on one telephony account.**
Possible, but each DID needs a clean mapping to the correct agent/extension. Collect both numbers and the desired mapping.

**Hold after answer.**
After a human picks up a transfer, hold/resume is on the physical phone or softphone. Ring timeout (how long it rings before fallback) is a transfer setting, not hold.

**One-way audio / no audio.**
Codec mismatch, RTP blocked, or (on Edge) UDP 51820 blocked. Check codec match and RTP / tunnel health before rewriting the prompt.

**Each outbound agent needs its own extension** (or port). Do not put two outbound agents on the same SIP extension. See `manage-calls-campaigns`.

## After a change

Say which number is now on which agent (or which edge device / SIP host). Suggest a test inbound call. Do not print secrets.
