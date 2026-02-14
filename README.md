# Nelko P21 Label Printer

A Python command-line tool for printing text and images to a **Nelko P21 Bluetooth thermal label printer** over bluetooth.


## Pairing

### Linux

#### Ensure the current user is in the `dialout` group

This allows access to `/dev/rfcomm0` without root:

```bash
sudo usermod -aG dialout "$USER"
```

Log out and back in for the group change to take effect.

#### Pair the powered-on P21 device

```bash
bluetoothctl
# In the bluetoothctl shell:
scan on
pair <DEVICE_MAC>
```

<details>
<summary>Output:</summary>

```bash
❯ bluetoothctl
Waiting to connect to bluetoothd...
[bluetooth]# scan on
[NEW] Device 15:B5:EF:46:08:B6 P21
[P21]# pair 15:B5:EF:46:08:B6
Attempting to pair with 15:B5:EF:46:08:B6
[CHG] Device 15:B5:EF:46:08:B6 Connected: yes
[P21]# [CHG] Device 15:B5:EF:46:08:B6 Bonded: yes
[P21]# [CHG] Device 15:B5:EF:46:08:B6 Paired: yes
[P21]# Pairing successful
```

</details>

#### Bind the device as an RFCOMM serial port

```bash
sudo rfcomm bind /dev/rfcomm0 <DEVICE_MAC> 1
```

You should now see `/dev/rfcomm0`.

### macOS

macOS does not require `usermod`, `bluetoothctl`, or `rfcomm` commands. The Bluetooth serial port is created automatically when the device is paired.

#### Install blueutil

```bash
brew install blueutil
```

#### Scan for the P21 device

Power on the P21 printer, then scan for nearby Bluetooth devices:

```bash
blueutil --inquiry 15
```

Look for a device named "P21" and note its MAC address (e.g., `aa-bb-cc-dd-ee-ff`).

#### Pair the device

Pair the P21 through **System Settings > Bluetooth**. The printer should appear as a discoverable device. Click **Connect** to pair it.

This creates the serial port at `/dev/tty.P21` and `/dev/cu.P21`.

> **Note:** The P21's Bluetooth stack locks up after a single serial session. The `--bt-mac` and `--blueutil` flags handle this automatically by performing an unpair/re-pair cycle before each print. See [macOS usage](#macos-1) below.


## Example Usage

### Linux

```bash
./p21.py --text "100Ω" # Single line
./p21.py --text $'100\n(Ω)' # Multi line
./p21.py --image test-template.png
```

### macOS

On macOS, pass `--bt-mac` and `--blueutil` to enable the automatic unpair/re-pair cycle required by the P21:

```bash
./p21.py --image test-template.png --device /dev/cu.P21 \
    --bt-mac aa-bb-cc-dd-ee-ff --blueutil "$(which blueutil)"

./p21.py --text "100Ω" --device /dev/cu.P21 \
    --bt-mac aa-bb-cc-dd-ee-ff --blueutil "$(which blueutil)"
```

### Preview only (no printing)

Display the generated label instead of printing it so you can sanity-check layout and waste less tape:

```bash
./p21.py --text "100Ω" --preview-only
./p21.py --image test-template.png --preview-only
```

## Acknowledgements

This script was made possible thanks to the excellent protocol reverse-engineering work achieved in [merlinschumacher/nelko-p21-capture](https://github.com/merlinschumacher/nelko-p21-capture/).
