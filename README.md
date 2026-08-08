# esp8266-tft-draw-photo
Web-based live drawing and photo display on ILI9341 TFT via ESP8266 — draw from any browser or send a photo, mirrors in real-time over WebSocket. Self-hosted, no cloud. By R Track Creation.
# ESP8266 TFT Draw & Photo Display

Draw live from any web browser and see it mirrored in real-time on an ILI9341 TFT display via ESP8266. You can also send a photo from your browser and it will be displayed full-screen on the TFT. No cloud dependency — everything runs locally over WiFi.

By R Track Creation

---

## Features

- Live drawing — draw with mouse/touch on a browser canvas, mirrored instantly on the TFT via WebSocket
- 5-color picker — white, red, green, blue, yellow
- Photo upload — select any image, it gets converted to RGB565 and displayed full-screen on the TFT
- Clear button — clears both the TFT and browser canvas in one click
- Self-hosted — ESP8266 runs its own web server and WebSocket server, no internet/cloud dependency, works on local WiFi only
- Mobile friendly — supports touch events, so you can draw from a phone browser too

---

## Hardware Required

| Component | Notes |
|---|---|
| ESP8266 (NodeMCU / Wemos D1 mini) | Main controller |
| ILI9341 TFT display | 240x320, SPI interface |
| Jumper wires | Male-to-female recommended |
| Micro USB cable | For programming + power |

---

## Wiring / Connection Details

ESP8266 <-> ILI9341 TFT

TFT VCC -> 3.3V
TFT GND -> GND
TFT CS -> D8 (GPIO15)
TFT RESET -> D4 (GPIO2)
TFT DC -> D3 (GPIO0)
TFT MOSI -> D7 (GPIO13)
TFT SCK -> D5 (GPIO14)
TFT LED -> 3.3V
TFT MISO -> D6 (GPIO12) [optional, not used for drawing]


Use 3.3V only — ESP8266 GPIO pins are not 5V tolerant. Connecting the TFT to 5V can permanently damage it.

If you need to move CS/DC/RESET to different GPIOs, update these lines in the code:

#define TFT_CS 15 // D8
#define TFT_DC 0 // D3
#define TFT_RST 2 // D4


---

## Libraries Required

Install via Arduino IDE -> Sketch -> Include Library -> Manage Libraries:

1. Adafruit_ILI9341
2. Adafruit_GFX — must be the latest version (photo feature needs the writePixels() function)
3. ESPAsyncTCP — by me-no-dev
4. ESPAsyncWebServer — by me-no-dev
5. WebSockets — by Markus Sattler (Links2004)

If a library isn't available in Library Manager, download the .zip from GitHub and add it via Sketch -> Include Library -> Add .ZIP Library.

---

## Setup Instructions

1. Open esp8266_tft_draw_photo.ino in Arduino IDE
2. Update your WiFi credentials (near the top of the file):

const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

3. Select Tools -> Board -> NodeMCU 1.0 (ESP-12E Module) (or whichever board you have)
4. Select the correct COM port
5. Click Upload
6. Once uploaded, open Tools -> Serial Monitor (baud rate: 115200)
7. Once WiFi connects, the IP address will be printed (e.g. 192.168.1.5) — note it down
8. On a phone/laptop connected to the same WiFi network, open that IP in a browser (e.g. http://192.168.1.5)
9. The page will load — you can start drawing or upload a photo

---

## How It Works

**Drawing Mode:**
Browser Canvas (JS) -> mousedown/mousemove/touchmove event -> {x0, y0, x1, y1, color} -> WebSocket send (text) -> ESP8266 (WebSocketsServer, port 81) -> parses coordinates + color -> tft.drawLine() -> instantly rendered on the ILI9341 TFT

**Photo Mode:**
Photo selected -> drawn onto a hidden 320x240 canvas (stretched to fit) -> each pixel converted to RGB565 (16-bit color) -> split into 1KB chunks (512 pixels each) -> sent over WebSocket as binary -> ESP8266 calls tft.startWrite() + tft.setAddrWindow() -> each chunk written via tft.writePixels() (no full-image buffering, keeps ESP8266 RAM usage low) -> tft.endWrite() called on IMG_END -> full photo displayed on the TFT

---

## Troubleshooting

| Problem | Possible Fix |
|---|---|
| TFT blank/white screen | Check wiring, especially CS/DC/RESET pins. Confirm VCC is 3.3V. |
| "TFT init failed" in Serial Monitor | Verify TFT_CS, TFT_DC, TFT_RST pin numbers match your wiring in the code |
| Colors wrong/inverted | Try changing tft.setRotation() value (0, 1, 2, or 3) |
| Compile error: writePixels not declared | Update Adafruit_GFX library to the latest version |
| Page won't open in browser | Confirm ESP and your device are on the same WiFi network. Double-check the IP address |
| Status shows "disconnected" | WebSocket port 81 may be blocked (rare) — check router firewall, or refresh the page |
| Photo upload is slow | Normal — a 320x240 image sends ~150 chunks. Weak WiFi will make it slower |
| Drawing lags/delays | Check WiFi signal strength, keep the ESP closer to the router |

---

## Project Structure

esp8266-tft-draw-photo/
├── esp8266_tft_draw_photo.ino   (Main sketch: WiFi + WebSocket + TFT + embedded HTML)
├── README.md
└── .gitignore

---

## Customization Ideas

- Change canvas resolution via CANVAS_W / CANVAS_H (must match your TFT rotation)
- Add more colors by adding .swatch elements in the HTML with matching RGB565 hex values
- Adjust brush thickness by changing the drawLine() offset count in webSocketEvent()
- Add mDNS so you can access it as esp8266.local instead of remembering the IP

---

## License

MIT — free to use, modify, and share.

---

Built by R Track Creation
