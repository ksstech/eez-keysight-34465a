# eez-keysight-34465a

EEZ Studio extension for the **Keysight (Agilent) 34465A** 6.5-digit Truevolt
digital multimeter. Also covers the 34461A (6.5-digit base) and 34470A
(7.5-digit) — same command family.

Unlike [eez-ea-ps2k](https://github.com/ksstech/eez-ea-ps2k), this instrument
speaks **native SCPI** directly — there is no custom bridge process here.
EEZ Studio (or anything else) connects straight to the instrument.

## Connection

- **Ethernet (recommended):** raw SCPI on port `5025`, or HiSLIP. Enter the
  instrument's IP address when adding it in EEZ Studio — default lab address
  `192.168.1.51`.
- **USB-TMC:** `idVendor 0x2a8d`, `idProduct 0x0101`.
- Optional GPIB, not covered by this extension's connection config.

Older firmware may identify as `Agilent Technologies` rather than `Keysight
Technologies` in `*IDN?` — update the IDN field in EEZ Studio's connection
settings if auto-detection doesn't match.

## Structure

| Path | Purpose |
|---|---|
| `package.json` | Extension metadata + all 29 shortcuts (measurement mode buttons, acquisition scripts, diagnostics) |
| `keysight_34465a.idf` | EEZ Studio instrument definition (SCPI error query, links to `.sdl`) |
| `keysight_34465a.sdl` | SCPI command/response definitions |
| `image.png` | Extension icon |

Built as a zip and published via [GitHub Releases](https://github.com/ksstech/eez-keysight-34465a/releases) — not committed to the repo.

## Functionality — shortcuts

**Measurement mode** (toolbar): `DCV` `ACV` `DCI` `ACI` `2W Res` `4W Res`
`Freq` `Cap` `Cont` `Diode` — one-click mode switch via `scpi-commands`
shortcuts (no JavaScript needed for these).

**Acquisition scripts** (JavaScript, use the shared
[`qts()`](https://github.com/ksstech/eez/blob/main/docs/qts-helper.md) helper):
`LR v46`, and a family of `Acq <mode> <count>×<interval>` shortcuts (e.g.
`Acq DCV 100×0.5s`) that switch mode, take N readings at a fixed interval,
and report results.

**Statistics:** `Stats On` / `Stats Read` / `Stats Off` — toggle the meter's
built-in min/max/average statistics tracking and read it back.

**Utility:** `Status`, `Clr Errors`, `Reset`, `Diag` (full diagnostic dump —
same purpose as the `Diag` shortcut in eez-ea-ps2k, adapted to this
instrument's command set).

## Using it without EEZ Studio

Since this instrument speaks native SCPI, it already works with any SCPI
client — nothing in this repo is required to use it from other tools, this
repo only adds convenience shortcuts on top.

**Raw socket (LAN SCPI, port 5025):**
```bash
echo -e "*IDN?\n" | nc 192.168.1.51 5025
# Keysight Technologies,34465A,MY12345678,A.02.14-02.40-02.14-00.49-01-01
```

**Python via PyVISA** (works over LAN, USB-TMC, or GPIB — same code):
```python
import pyvisa
rm = pyvisa.ResourceManager()
dmm = rm.open_resource("TCPIP0::192.168.1.51::5025::SOCKET")
dmm.read_termination = "\n"
dmm.write_termination = "\n"

print(dmm.query("*IDN?"))
dmm.write("CONF:VOLT:DC")
print(dmm.query("READ?"))
```

## License

No LICENSE file is currently set — add one if this is meant to be reused
under specific terms; until then, standard GitHub default (all rights
reserved) applies to original content here.
