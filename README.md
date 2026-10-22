Bluetooth Low Energy<br>
Thermal Printer Library<br>
Copyright (c) 2020 BitBank Software, Inc.<br>
Written by Larry Bank<br>
bitbank@pobox.com<br>
<br>
![printer demo](/ble_prt.jpg?raw=true "Thermal_Printer")

<br>
This Arduino library allows you to easily generate text and graphics
and send them to various BLE thermal printers. I can support the printers that I currently own; if your model is not yet supported, the best way to encourage me to support it is to send me funds to buy it.<br>
There are many different
BLE APIs depending on the board manufacturer, I support the three more
popular ones: ESP32, Adafruit and Arduino (e.g. Nano BLE 33). The three main features of
thermal printers are supported by this code - plain text, barcodes (1D+2D) and dot addressable
graphics. The graphics are treated like a display driver - you define a RAM buffer and draw text, dots, lines and bitmaps into it, then send it to the printer. Text output supports the various built-in font size+attribute options of some printers as well as Adafruit_GFX format bitmap fonts (sent as graphics).
See the Wiki: https://github.com/bitbank2/Thermal_Printer/wiki/Public-API for a description of each function.
<br>

<br>
Features<br>
========<br>
- Supports the GOOJPRT PT-210, MTP-3, PeriPage+ and 'cat' printers (so far)<br>
- Supports the CYLO printers distributed as `Cylo Mini Thermal Printer` (PRINTER_CYLO), including this fork's specific device<br>
- Compiles on ESP32, Adafruit nRF52 and Arduino boards with bluetooth<br>
- Supports graphics (dots, lines, text, bitmaps), 1D + 2D barcodes, and plain text output<br>
- Allows printing Adafruit_GFX fonts one line at a time or drawing them into a RAM buffer<br>
- Can scan/connect to printers by BLE name or auto-detect the supported models<br>
- Doesn't depend on any other 3rd party code<br>
<br>

Here is a subjective chart of the printer models I've tested and are supported by this code. Please feel free to send me info about other models that work and additional comments about these printers.<br>

<br>
<p align="center">
  <img width="939" height="203" src="https://github.com/bitbank2/Thermal_Printer/blob/master/printer_chart.jpg?raw=true">
</p>
<br>

If you find this code useful, please consider becoming a Github Sponsor or sending a single donation.

## CYLO printer support (this fork)

- **Printer type:** `PRINTER_CYLO` — registered as its own type, not routed
  through any other printer's protocol selection.
- **Display/registry name:** `Cylo Mini Thermal Printer` (display name only;
  it is not assumed that the device broadcasts this exact string). Matching
  uses the existing exact-name table plus a CYLO-only fallback that accepts
  any advertised name beginning with `Cylo`, so a BLE advertising-packet
  truncation of a longer name still resolves. **Unknown (not observed on
  hardware):** the actual advertised name of the device. If it does not
  begin with `Cylo`, add it to `szPrinterIDs` in `src/Thermal_Printer.cpp`.
  Use `tpScan("Cylo", seconds)` followed by `tpConnect()`, or auto-detect
  with `tpScan()`.
- **Protocol reference:** the printer speaks the protocol documented at
  <https://github.com/joshmcarthur/yhk-mini-printer/blob/main/docs/protocol.md>.
  That reference defines the device family's communication; it is used here
  only as the technical specification — the printer in this fork is supported
  as CYLO.
- **Transport:** Bluetooth LE, ISSC Transparent UART service
  `49535343-fe7d-4ae5-8fa9-9fafd205e455`, write characteristic
  `49535343-8841-43f4-a8d4-ecbe34729bb3` (write-without-response by default;
  `tpSetWriteMode()` can switch to write-with-response if data is lost).
- **Printing:** standard ESC/POS raster (`GS v 0`, `1D 76 30 00`), 384 dots
  (48 bytes) per row, MSB-first pixel bits, little-endian width/height —
  identical to the existing MTP2/MTP3 raster path, and all rendering APIs
  (back buffer, fonts, BMP, lines) work unchanged. 1D barcodes work; QR codes
  (`tpQRCode`) are not enabled for this type yet.
- **Paper feed:** `ESC d n` (`1B 64 n`) advances n lines via `tpFeed()`.
- **Timing:** the printer has only a small internal buffer; data sent faster
  than the print head consumes it is silently dropped (usually truncating the
  bottom of the image). The library sends 20-byte chunks with short delays.
  A 1500 ms wait is applied after the last raster line (a starting value from
  the protocol reference, **not hardware-verified** — increase the delay if
  the bottom of the print is truncated, decrease it if white horizontal bands
  appear).

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=SR4F44J2UR8S4)
