# Intro to RetroPie

`EmulationStation` runs inside RetroPie and provides the UI.

Press `F4` on the keyboard to get into the terminal and to get back, type `emulationstation` or `exit`.

## About controllers

The configuration of gamepads are saved in the following path:

> /opt/retropie/configs/all/emulationstation/es_input.cfg

### 8bitdo SN30 Pro

The controller has different modes for different consoles. When connecting to a Raspberry Pi via bluetooth, just press start to get the 4 lights flashing in a rotating pattern. If they aren't flashing like that it's probably because it is in the wrong mode. Try pressing `Y` + `Start`.

## Bluetooth Blues

I had massive problems connecting my 8BitDo controller via bluetooth. After a lot of digging it turns out that there's a timing issue beteween `bluetoothd` and `bthelper`. At some point `bluetoothd` tries to set the privacy mode for the bluetooth chip but that must be done before the `hci0` interface has started. Since `bthelper` runs `hciconfig hci0 up` it becomes a race condition - will `bluetoothd` set privacy before `bthelper` turns on the interface? If it happens in the wrong order `service bluetooth status` will have this line:

> bluetoothd[562]: Failed to set privacy: Rejected (0x0b)

And it seems like the 8BitDo controller requires that the bluetooth chip advertises with whatever privacy `bluetoothd` is trying to set it to.

My temporary solution is to:

1. Turn off `hci0` by running `sudo hciconfig hci0 down`
2. Then restart `bluetoothd` by running `bluetoothctl power off` and then `bluetoothctl power on`

No need to run `sudo hciconfig hci0 up`, that seems to happen when powering on `bluetoothctl`.

### Debug Noise: "Failed to load SAP interface"

The definition of the bluetooth service is located at:

> /lib/systemd/system/bluetooth.service

That files points out the `bluetoothd` daemon that is the actual BlueZ implementation of the bluetooth stack. Adding `--noplugin=sap` after `ExecStart=/usr/lib/bluetooth/bluetoothd` removes the "Failed to load SAP interface" message. The SAP interface is used to read/write to SIM cards.

Example of the removing the SAP plugin by editing `bluetooth.service`:

> ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap