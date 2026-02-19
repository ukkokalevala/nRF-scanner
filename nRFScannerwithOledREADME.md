RF Radar with OLED - Technical Briefing
It will detect:
•	WiFi traffic
•	Bluetooth activity
•	2.4 GHz interference
•	Other nRF devices
(It will NOT decode WiFi — just detect RF energy.)

Overview
A standalone 2.4GHz RF spectrum analyzer that displays real-time signal activity directly on an OLED screen - no PC required!
Hardware Configuration
ESP-C3-32S-Kit Pin Connections
Component	Pin	Function
nRF24L01		
CE	GPIO4	Chip Enable
CSN	GPIO5	SPI Chip Select
SCK	GPIO6	SPI Clock
MOSI	GPIO7	SPI Data Out
MISO	GPIO2	SPI Data In
VCC	3.3V	Power
GND	GND	Ground
OLED (SSD1306)		
SDA	GPIO8	I2C Data
SCL	GPIO9	I2C Clock
VCC	3.3V	Power
GND	GND	Ground
How It Works
1. RF Detection Process
nRF24L01 scans 126 channels (2400-2525MHz)
    ↓
For each channel: enable receiver for 150μs
    ↓
testRPD() checks if signal > -64dBm
    ↓
If detected → mark channel as active
    ↓
Peak tracking stores maximum signals
2. Data Processing
loop() {
  Clear previous readings
  For each channel (0-125):
    - Set frequency
    - Listen for 150μs
    - Check for signal
    - Update peak values
    - Decay old peaks
  Draw display
  Wait 50ms
}
3. Display Layout
┌─────────────────────┐
│ 2.4GHz RF Radar     │  ← Title
│                     │
│    ██   ██  ██      │  ← Signal lines
│    ██   ██  ██      │    (channels with activity)
│    ██   ██  ██      │
│    ██   ██  ██      │
│ ────────────────── │  ← Baseline
│ 0   |   |   |   125 │  ← Channel markers
│    1   6   11       │    WiFi channels
└─────────────────────┘
Key Features
1. Real-Time Spectrum Display
•	X-axis: 126 channels (2400-2525MHz)
•	Y-axis: Signal detection (binary)
•	White lines: Current RF activity
2. Peak Hold Function
•	Dots/Pixels: Show strongest recent signals
•	Gradual decay: Peaks fade over time
•	Memory: Tracks historical maximums
3. WiFi Channel Markers
•	Ch 1 (2412MHz): Position 20
•	Ch 6 (2437MHz): Position 61
•	Ch 11 (2462MHz): Position 102
•	Helps identify WiFi networks
Signal Processing
Detection Logic
if (radio.testRPD()) {
  channelActivity[ch] = 1;        // Mark active
  if (channelPeak[ch] < 10) {
    channelPeak[ch] = 10;         // Set peak
  }
} else {
  if (channelPeak[ch] > 0) {
    channelPeak[ch]--;             // Decay peak
  }
}
Peak Decay
•	Peaks start at height 10
•	Decrease by 1 each sweep
•	Provides ~0.5 second history (at 50ms sweep rate)
Performance
•	Scan time: ~19ms (126 channels × 150μs)
•	Display update: Every 50ms (20 fps)
•	Frequency range: 2400MHz - 2525MHz
•	Resolution: 1MHz per channel
•	Sensitivity: Signals > -64dBm
Visual Elements Explained
Element	Purpose
White lines	Current active channels
White pixels	Historical peak signals
Baseline	Reference line at bottom
Channel markers	WiFi channel positions
Title	System identification
Applications
1.	WiFi Network Detection
o	See active WiFi channels
o	Identify crowded frequencies
2.	RF Interference Hunting
o	Find microwave oven leaks
o	Detect Bluetooth devices
o	Locate cordless phones
3.	Educational Tool
o	Visualize RF spectrum
o	Understand radio frequencies
o	Demonstrate signal detection
Advantages Over PC Version
•	Portable: Battery operation possible
•	Self-contained: No computer needed
•	Instant-on: Display shows immediately
•	Low cost: Only OLED + nRF24L01
Troubleshooting
Issue	Solution
No OLED display	Check I2C address (0x3C or 0x3D)
No RF signals	Set RF24_PA_MAX instead of MIN
Flickering display	Increase delay(50) to delay(100)
Weak detection	Check antenna connection
Random peaks	Add capacitor across VCC/GND
Code Structure Summary
setup()
  ├─ Initialize Serial debug
  ├─ Setup I2C for OLED
  ├─ Initialize OLED display
  ├─ Show splash screen
  ├─ Initialize SPI for nRF24
  ├─ Configure nRF24 for radar mode
  └─ Show "READY" message

loop()
  ├─ Clear channel activity
  ├─ Scan all 126 channels
  │  ├─ Set channel frequency
  │  ├─ Listen for 150μs
  │  ├─ Check testRPD()
  │  └─ Update peaks
  ├─ Clear OLED display
  ├─ Draw title and baseline
  ├─ Draw active channels
  ├─ Draw peak pixels
  ├─ Draw WiFi markers
  └─ Update display
Power Consumption
•	nRF24L01: ~12mA (listening mode)
•	OLED: ~20mA (with display on)
•	ESP32-C3: ~30mA
•	Total: ~62mA continuous

