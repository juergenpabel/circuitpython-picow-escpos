This CircuitPython application runs on a Rasperry Pico W that connects via USB to a thermal printer; the application waits via MQTT for ESCPOS-formatted printjobs and sends them via USB to the thermal printer.

# Warning 1: This code requires CircuitPython >= 9 (as of 2023-09-05 not released, it is presumed as HEAD on master), as it requires USB-Host support for the Pi PicoW.
# Warning 2: The current CircuitPython implementation status (commit 000d22f250750bf0853c08a6ebba63de39ed3c18) has a non-working USB-Host support for the Pi PicoW due to a lack of PIO space. The following diff adapts the PIO configuration into a working state:
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
