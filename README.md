# Govee-h66A0
govee wire diagram [incomplete] — see **Findings** below: the camera cable is proprietary, not standard USB-C.

Govee official cable
<img width="1257" height="1671" alt="image" src="https://github.com/user-attachments/assets/34b3737f-c637-478f-ad08-c653299ec372" />
<img width="1257" height="1671" alt="image" src="https://github.com/user-attachments/assets/946d07db-a68a-4490-ac9d-4bed3fde4bdd" />
Breakout boards [back and front so there is no confusion]
<img width="2220" height="1671" alt="image" src="https://github.com/user-attachments/assets/abe2129c-d7ef-4a23-a93e-13410eb1aa53" />
<img width="2220" height="1671" alt="image" src="https://github.com/user-attachments/assets/4ccd2d71-d6bb-4a2a-b342-b92bcb8d0b1a" />
<img width="1257" height="1671" alt="image" src="https://github.com/user-attachments/assets/44987f4f-8b34-4df5-8b70-d7e47b81299c" />
<img width="1257" height="1671" alt="image" src="https://github.com/user-attachments/assets/0fce4301-81ee-4c0c-a744-0f808d228a9d" />

## Findings (Oct 2026)

**TL;DR: the H66A0 camera cable is NOT a standard USB-C cable.** Govee uses the USB-C plug shape with its own wiring. No off-the-shelf USB-C cable (USB 2.0 *or* full-featured USB 3.x) will connect the camera to the control box.

Symptom: the Govee app says *"Installation reminder: we detected that your camera has not been installed or initialized, so the video function is unavailable"*. "Install and initialize" does nothing. It has not worked since July 2026.

### Official cable pinout (from the multimeter sheet `Screenshot 2026-07-12 034124.png`)

Measured with breakout boards on both plugs. Both ends (side 1 and side 2) read the same.

| Wired (~0.4–0.8 Ω) | Not connected |
|---|---|
| VCC: A9, B4, B9 | **CC: A5, B5** |
| GND: A12 | SBU: A8, B8 |
| A6 (D+) | A2, A3, A10 |
| B7 (D−) | B2, B3, B11 |
| **A11**, **B10** | |

The colours and numbers in the sheet are the wire colour and resistance for each pin.

### Why regular USB-C cables fail

1. **USB 2.0 C-C cables** (most charging cables) have no wires on A11 or B10, so the camera signal never reaches the box.
2. **Full-featured USB 3.x cables** do wire those pins, but the USB-C spec crosses them over: A11 → B2 and B10 → A3 at the far end. Govee's cable appears to run A11 → A11 and B10 → B10 straight through, because B2 and A3 are open on both ends. The signal lands on a pin the other unit doesn't use.
3. **D+ on A6 and D− on B7** are on opposite rows of the plug, which is not how a standard cable/plug carries USB 2.0.
4. **CC is not wired**, so a PC or charger USB-C port never detects the device and never turns on VBUS. Plugging the camera into a PC shows nothing in Windows. That is expected, and it says nothing about whether the camera works.

The tag on the official cable warns: *"Attention: Do not insert the camera or control box into…"* other adapters/ports.

### Bluetooth (read-only probing from a PC)

| BLE name | Role byte in advert | Notes |
|---|---|---|
| `Govee_H66A0_FF51` | `ec00 02 0101` | Has extra characteristic `…2b12` = `0102 00…` |
| `Govee_H66A0_2808` | `ec00 01 0101` | Same services, no `…2b12` |

- Both units power on, advertise (manufacturer ID `0x8843`), and accept BLE connections. Their main controllers are alive.
- Services: Govee control `00010203-0405-0607-0809-0a0b0c0d1910` (write `…2b11`, notify `…2b10`), plus a vendor service `02f00000-…-00000000fe00` (`ff00`–`ff03`, `ff02` reads `ntf_enable`). That looks like a firmware-update/debug channel. Do **not** write to it.
- Classic Govee status queries (`0xAA 0x01`, `0x04`, `0x06`, `0x07`, …) get **no reply**. Newer firmware only accepts commands over the Govee app's encrypted session.
- The camera ↔ control box link is wired (through the cable above), not Bluetooth. There is nothing to sniff over the air.
- The MAC prefixes (`C0:17:55`, `C0:65:E5`) are not in the public OUI registry, so the chip vendor is still unknown.
- Gotcha: connecting from Windows can leave the unit **paired** to the PC, which may block the phone app. Remove it under Settings → Bluetooth & devices.

### Next steps / TODO

- [ ] Short test: on one plug of the official cable (other end unplugged), check every pair of used pins reads **open**.
- [ ] Flex test: meter on one conductor, bend the cable along its length and watch for dropouts.
- [ ] Control box only (powered, no camera), on a breakout: measure DC volts from GND to VCC, A6, B7, A11, B10. VCC should be ~5 V.
- [ ] Same measurements with camera ⇄ breakout ⇄ cable ⇄ breakout ⇄ box all connected.
- [ ] Photos of the camera and control box PCBs (chip markings) to identify the chipset and the signal type on A11/B10.
- [ ] DIY cable: wire VCC, GND, A6, B7, A11, B10 straight through between two breakouts. Keep it short, twist A6/B7 together.
- If the cable passes and 5 V reaches the camera, the camera is the dead part. That's a warranty claim to Govee.
