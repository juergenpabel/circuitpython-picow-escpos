## Announcement ##
Because this application is not in any way confined to the Pi Pico W (but rather only to a circuitpython device with usb-host support), I re-published this software in a [new repository](https://github.com/juergenpabel/circuitpython-escpos-server) and will continue its development there. This repository is now a public archive.

---

# Abstract

This CircuitPython application runs on a Rasperry Pico W that connects via USB to a thermal printer; the application waits via MQTT for ESCPOS-formatted printjobs and sends them via USB to the thermal printer (without any parsing; but with an additional paper-cut command at the end).

# Software

**Warning 1**: This code requires CircuitPython >= 9 (as of 2023-09-05 just in alpha status), as it requires USB-Host support for the Pi PicoW.

**Warning 2**: The current CircuitPython implementation status (as of 2023-09-05 on master) has a non-working USB-Host support for the Pi PicoW due to insufficient PIO space. The following diff adapts the PIO configuration into a working state (by simply swapping assignments for PIO #0 and #1):
```
diff --git a/ports/raspberrypi/common-hal/usb_host/Port.c b/ports/raspberrypi/common-hal/usb_host/Port.c
index 93d19acd6..6b9aeae55 100644
--- a/ports/raspberrypi/common-hal/usb_host/Port.c
+++ b/ports/raspberrypi/common-hal/usb_host/Port.c
@@ -122,8 +122,8 @@ usb_host_port_obj_t *common_hal_usb_host_port_construct(const mcu_pin_obj_t *dp,
     pio_usb_configuration_t pio_cfg = PIO_USB_DEFAULT_CONFIG;
     pio_cfg.skip_alarm_pool = true;
     pio_cfg.pin_dp = dp->number;
-    pio_cfg.pio_tx_num = 0;
-    pio_cfg.pio_rx_num = 1;
+    pio_cfg.pio_rx_num = 0;
+    pio_cfg.pio_tx_num = 1;
     // PIO with room for 22 instructions
     // PIO with room for 31 instructions and two free SM.
     if (!_has_program_room(pio_cfg.pio_tx_num, 22) || _sm_free_count(pio_cfg.pio_tx_num) < 1 ||
```
This circuitpython application relies on only the [adafruit_minimqtt](https://github.com/adafruit/Adafruit_CircuitPython_MiniMQTT/) library as an external dependency, it is provided within the "lib" folder. The configuration is contained in settings.toml with the following keys:
- WIFI_SSID: The SSID of the AP to connect to (I didn't want to use CIRCUITPY_WIFI_SSID due to the included web interface; not needed/wanted)
- WIFI_PSK: The password for the network
- PRINTER_USB_VID: The USB vendor ID of the attached thermal printer (in hex notation, like "04b8" for Epson)
- PRINTER_USB_PID: The USB product ID of the attached thermal printer (in hex notation, like "0e15" for TM-T20II)
- MQTT_BROKER_IPV4: MQTT server address (DNS or IPv4)
- MQTT_BROKER_USER: MQTT username (or unset for anonymous)
- MQTT_BROKER_PASS: MQTT password (or unset for anonymous)
- MQTT_BROKER_TOPIC: MQTT topic to listen on for ESCPOS-formatted print data
- DEBUG: If set (to any value that yields *True* when passed to *bool()*) the application will wait on startup for a serial connection to the board (this is for debugging purposes so that nothing interesting happens before the serial connection has been established)

# Hardware

This section describes my own hardware setup, as shown in this picture:

![Picture of setup with more details](https://raw.githubusercontent.com/juergenpabel/circuitpython-picow-escpos/master/resources/images/setup_detail.jpg)

The PicoW is powered using 24VDC provided by the printer (an Epson TM-T20II) via the DK-/Drawer-Kick-Connector; V+ (Pin 4) and GND (Pin 6) are sent into a step-down converter that provides 5V per USB. The PicoW plugs into the step-down converter and exposes a USB host port via GPIO (a new feature in CircuitPython 9) and connects to the USB interface on the printer (for sending ESC/POS prinbt jobs). This setup allows for printer installations using only power (and WiFi).

# FAQ

- But why not simply use a WiFi-enabled thermal printer (Epson supports even some WiFi USB-Dongles)? That's a three-part answer:
  1. Because I don't have a WiFi-enabled model and don't want to pay for getting one
  2. I would like to have MQTT (with ESCPOS as the payload) as the interface to my printer (because MQTT is how most of my stuff at home is "linked" together)
  3. I think that the Pi PicoW (with the Infineon CYW43xxx for WiFi/BT) is probably way more securely implemented at the WiFi-layer than some add-on WiFi-board/-dongle for some thermal printer; even if that's not effectively so, there's probably a much better chance for firmware updates.

- Can't I just use a nightly build of circuitpython? Once the PIO issue (see diff from above) has been resolved: Probably (that diff is the only diff I applied for my builds).

- Shouldn't there be an issue or even pull-request for circuitpython? Check out [issue #8359](https://github.com/adafruit/circuitpython/issues/8359)
