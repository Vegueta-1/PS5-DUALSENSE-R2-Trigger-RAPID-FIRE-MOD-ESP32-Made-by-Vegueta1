PS5 DUALSENSE R2 Trigger RAPID-FIRE MOD ESP32— Made by Vegueta1


⚠️ FULLY WORKING STATUS — READ BEFORE INSTALLING.

Introducing the PS5 Dual Sense R2 Trigger Rapid-Fire Mod modular rapid-fire system targeting the R2 trigger on all Dual Sense controller revision (BDM-010)

This project is open-source under the MIT License (see https://opensource.org/licenses/MIT) and designed to be accessible for hobbyists, with a noob-friendly installation guide. A huge shoutout to RDC for His legendary Dual Sense PCB scans and hardware insights, which made this project possible.

Status: Beta – Currently Working to expand support to other Board This Is a Beta to confirm that work on other boards.

⚠️ Important Notes.

Educational Use Only — Online Play at your own risk!!!!!!
This release is provided as is, with no warranty or guarantee of fitness for any purpose. It is designed for learning, experimentation, and offline testing.
Do not use this modification in online environments — doing so may result in bans, account suspension, or other penalties.
You are solely responsible for how you use this project.


Visual Aids for ESP32 Chip is ESP32-D0WD-V3 (revision v3.1) Features: Wi-Fi, BT, Dual Core, 240MHz.

<img width="750" height="538" alt="image" src="https://github.com/user-attachments/assets/1a39beab-5c86-46a6-9eb1-a78d2637c53b" />

FEATURES:
<img width="504" height="1190" alt="image" src="https://github.com/user-attachments/assets/8905d5c2-efb2-4dc1-9b1f-4794a7504245" />
<img width="501" height="1273" alt="image" src="https://github.com/user-attachments/assets/dd3c590d-29ff-4404-abe8-db8b11a53f30" />

PCB PS5 Controller triggers R2 Assembly: You Don't need to remove R2 trigger you can solder the point taking the flex out only, fast and easy.


BWL-010, TRIGGER ASSEMBLY:
<img width="1024" height="719" alt="image" src="https://github.com/user-attachments/assets/88ae1d03-3fb0-4a16-b160-eb52fbf08101" />

BWL-020, TRIGGER ASSEMBLY:
<img width="1024" height="715" alt="image" src="https://github.com/user-attachments/assets/f791726f-7928-43dd-bc14-2f621900b964" />
What You Need

- ESP32‑D0WD‑V3 (rev 3.1)

- Source code paste it on Arduino IDE

- Data‑capable USB cable

- Windows, macOS, or Linux PC
A huge thank you to RDC From AcidMods for His invaluable PCB scans and hardware insights, and to the PSX-Place community for inspiring this project.


STATUS UPDATE: It Work Fully on My Ps5 Dual Sense BDM-010 need you guys to test on other controller versions.


Let's Build This Together!
If you try this mod, I'd love to hear your feedback.
Drop a comment with your results, ideas, or any tweaks you'd like to see — it really helps me improve future releases.
If you try it, drop a comment with your results — especially if you test on other Dual Sense board revisions. Your feedback will help shape the next release.

Collaboration Request — 3D‑Printed Back Case for ESP32‑D0WD‑V3
Goal:

Design a low‑profile, snap‑fit or screw‑on back case for the ESP32‑D0WD‑V3 (rev 3.1) that mounts to the rear of the PS5 Dual Sense controller (model CFI‑ZCT1W), in the central flat area between the grips.

Requirements:

• Exact Fit for Module: ESP32‑D0WD‑V3 (rev 3.1) with Wi‑Fi, Bluetooth, dual‑core 240 MHz.
• Mounting Method: Secure to controller shell without stressing solder joints; screw‑on, clip‑on, or adhesive‑backed options welcome.

• Access Points: Cut‑outs for USB port, status LEDs, and airflow.

• Ergonomics: Low‑profile so it doesn't interfere with grip comfort.

• Serviceability: Easy to remove for firmware updates or repairs.

Why This Matters:

This mount will allow me to expand the mod with new modes and external button support while keeping the install clean, safe, and reversible.

Notes:

• I don't have a 3D printer or prior experience with 3D modeling/printing, so I'm looking for someone who can handle the design and test‑print process. If you're interested in helping, please reach out — your work will be credited in the project release.

[ESP32 PS5 DualSense RapidFire ModV6.5.txt](https://github.com/user-attachments/files/30240729/ESP32.PS5.DualSense.RapidFire.ModV6.5.txt)


// ESP32 PS5 DualSense RapidFire Mod for BDM-010 Board
// Made By Vegueta1 - FIXED & OPTIMIZED v6.5
// Version: 6.5 (Continuous SPS Stabilized)
#include <WiFi.h>
#include <AsyncTCP.h>
#include <ESPAsyncWebServer.h>
#include <Preferences.h>
#include <esp_random.h>
#include <algorithm>

#define R2_PIN 32
#define STATUS_LED_PIN 2

#define MODE_OFF 0
#define MODE_CONTINUOUS 1
#define MODE_BURST 2
#define NUM_MODES 3

#define DEFAULT_PULSE_WIDTH_US 40000UL
#define BURST_DELAY_US 45000UL
#define BURST_COOLDOWN 250

#define INACTIVITY_TIMEOUT 300000UL

#define ADC_SAMPLE_INTERVAL 8
#define SERVER_INTERVAL 50
#define INACTIVITY_CHECK 200

#define CAL_SAMPLES 16
#define DEFAULT_DEBOUNCE_MS 45
#define BURST_DEBOUNCE_MS 5
#define ADC_AVG_SAMPLES 16
#define INPUT_SETTLE_US 3000

#define DEFAULT_ADC_IDLE 790
#define DEFAULT_ADC_PRESSED 252
#define DEFAULT_HYSTERESIS_PERCENT 10
#define DEFAULT_ADC_POLARITY 0
#define DEFAULT_PRESS_PERCENT 95

#define VREF 3.3f
#define FIXED_RESISTOR 10000.0f

#define NVS_NAMESPACE "rapidfire"
#define NVS_MAGIC 0xA5

// Continuous-mode release sensing is now done inside the scheduled press phase.
// This prevents random GPIO input switching during the release gap.
#define CONT_INITIAL_RELEASE_US 5000UL
#define CONT_PRESS_CHECK_START_US 1200UL
#define CONT_PRESS_SAMPLE_INTERVAL_US 2000UL
#define CONT_RELEASE_SAMPLES 6
#define CONT_RELEASE_PERCENT 20

#define FIRING_RESTART_LOCKOUT_MIN_MS 60
#define FIRING_RESTART_LOCKOUT_MAX_MS 220

#define DEFAULT_CUSTOM_SPS 10
#define DEFAULT_CUSTOM_BURST 3

const char* modeNames[] PROGMEM = {"OFF", "Continuous", "Burst"};

AsyncWebServer server(80);
AsyncWebSocket ws("/ws");
Preferences prefs;

unsigned long current_time = 0;
unsigned long last_adc_time = 0;
unsigned long last_server_time = 0;
unsigned long next_pulse_time_us = 0;
unsigned long last_inactivity_time = 0;
unsigned long debounce_start = 0;
unsigned long last_burst_end = 0;
unsigned long mode_change_time = 0;

// Stable continuous SPS scheduler.
unsigned long cycle_start_time_us = 0;

// Continuous press-phase release sensing.
unsigned long press_phase_start_us = 0;
unsigned long last_press_phase_sample_us = 0;
unsigned long last_pulse_end_time_us = 0;

// Restart protection after a release stop.
unsigned long last_firing_stop_time = 0;
unsigned long firing_restart_unlock_time = 0;
bool pending_fire_after_unlock = false;

int current_mode = MODE_OFF;
int last_active_mode = MODE_CONTINUOUS;

int adc_idle = DEFAULT_ADC_IDLE;
int adc_pressed = DEFAULT_ADC_PRESSED;
int adc_threshold = DEFAULT_ADC_PRESSED;
int adc_hysteresis = 0;
int hysteresis_percentage = DEFAULT_HYSTERESIS_PERCENT;
int adc_polarity = DEFAULT_ADC_POLARITY;
int jitter_percentage = 0;
int debounce_ms = DEFAULT_DEBOUNCE_MS;
int press_percent = DEFAULT_PRESS_PERCENT;

unsigned long pulse_width_us = DEFAULT_PULSE_WIDTH_US;

int custom_sps = DEFAULT_CUSTOM_SPS;
int custom_burst_count = DEFAULT_CUSTOM_BURST;
unsigned long continuous_interval_us = 1000000UL / DEFAULT_CUSTOM_SPS;

int press_level = LOW;
int release_level = HIGH;

bool firing = false;
bool pulse_active = false;
bool r2_pressed = false;
bool debounce_pending = false;
bool debounce_target = false;
bool pulse_ready = false;
bool gpio_is_output = false;

int burst_counter = 0;
int burst_target = 0;

int adc_sample = DEFAULT_ADC_IDLE;
unsigned long inactivity_remaining = INACTIVITY_TIMEOUT;

float trigger_press_ratio = 0.0f;
int firing_release_count = 0;

const char index_html[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
<title>RapidFire Mod v6.5 Beta</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; text-align: center; margin: 0; padding: 20px; background: linear-gradient(to bottom, #1e1e1e, #333); color: #fff; }
.container { max-width: 460px; margin: auto; padding: 20px; background: rgba(255,255,255,0.1); border-radius: 15px; box-shadow: 0 4px 30px rgba(0,0,0,0.5); backdrop-filter: blur(5px); }
h1 { color: #00ffcc; text-shadow: 0 0 10px #00ffcc; }
h3 { color: #fff; margin-bottom: 5px; }
button { padding: 12px 24px; margin: 8px; font-size: 16px; border: none; border-radius: 8px; cursor: pointer; transition: all 0.3s; box-shadow: 0 2px 10px rgba(0,0,0,0.3); }
button:hover { transform: scale(1.05); box-shadow: 0 4px 15px rgba(0,255,204,0.5); }
.cal-btn { background: linear-gradient(#ffc107, #ff8c00); color: #000; }
.apply-btn { background: linear-gradient(#28a745, #1e7e34); color: white; }
.mode-btn { background: linear-gradient(#007bff, #0056b3); color: white; }
.off-btn { background: linear-gradient(#dc3545, #bd2130); color: white; }
.set-btn { background: linear-gradient(#17a2b8, #138496); color: white; }
.reset-btn { background: linear-gradient(#6c757d, #5a6268); color: white; }
.danger-btn { background: linear-gradient(#dc3545, #bd2130); color: white; }
.debug-btn { background: linear-gradient(#ff9800, #f57c00); color: white; }
.status { font-size: 14px; margin-top: 20px; padding: 15px; background: rgba(0,0,0,0.3); border-radius: 10px; text-align: left; box-shadow: inset 0 0 10px rgba(0,0,0,0.5); }
.status p { margin: 8px 0; color: #ccc; }
.value { color: #00bfff; font-weight: bold; }
.led { width:16px; height:16px; border-radius:50%; display:inline-block; border:1px solid #fff; vertical-align: middle; margin-left: 5px; box-shadow: 0 0 5px #fff; }
.value-control { display: flex; flex-direction: row; align-items: center; justify-content: center; margin: 10px 0; gap: 10px; }
input[type=number] { width: 80px; padding: 8px; font-size: 18px; text-align: center; border: 1px solid #00ffcc; border-radius: 5px; background: #222; color: #fff; }
hr { border: 0; height: 1px; background: linear-gradient(to right, transparent, #00ffcc, transparent); margin: 20px 0; }
.debug-note { background: rgba(76,175,80,0.25); padding: 12px; border-radius: 6px; margin: 12px 0; font-size: 13px; color: #4caf50; border-left: 4px solid #4caf50; }
.polarity-btn { padding: 10px 16px; font-size: 15px; }
.section-box { background: rgba(0,0,0,0.2); padding: 15px; border-radius: 10px; margin: 15px 0; border: 1px solid rgba(255,255,255,0.1); }
</style>
</head>
<body>
<div class="container">
<h1>RapidFire Mod v6.5</h1>
<div class="debug-note">
FIXED: Continuous SPS no longer jumps randomly<br>
Release sensing is now done inside the scheduled press phase<br>
<strong>Includes:</strong> Custom SPS, Custom Burst, Jitter, Hysteresis, Debounce, Press%, Pulse Width, Polarity
</div>
<div class="section-box">
<h3>Continuous Mode (Custom SPS)</h3>
<div class="value-control">
<input type="number" id="spsValue" min="1" max="50" step="1" value="10">
<span>SPS</span>
</div>
<button class="set-btn" onclick="setCustomSPS()">Set SPS</button>
<button class="mode-btn" onclick="sendCommand('/activate_continuous', 'Activate Continuous')">Activate Continuous</button>
</div>
<div class="section-box">
<h3>Burst Mode (Custom Shots)</h3>
<div class="value-control">
<input type="number" id="burstValue" min="1" max="50" step="1" value="3">
<span>Shots</span>
</div>
<button class="set-btn" onclick="setCustomBurst()">Set Shots</button>
<button class="mode-btn" onclick="sendCommand('/activate_burst', 'Activate Burst')">Activate Burst</button>
</div>
<hr>
<div>
<h3>Jitter (0-100%)</h3>
<div class="value-control">
<input type="number" id="jitterValue" min="0" max="100" step="1" value="0">
</div>
<button class="set-btn" onclick="setJitter()">Set Jitter</button>
<button class="reset-btn" onclick="resetJitter()">Reset</button>
</div>
<hr>
<div>
<h3>Hysteresis (0-100%)</h3>
<div class="value-control">
<input type="number" id="hystValue" min="0" max="100" step="1" value="10">
</div>
<button class="set-btn" onclick="setHyst()">Set Hysteresis</button>
<button class="reset-btn" onclick="resetHyst()">Reset</button>
</div>
<hr>
<div>
<h3>Debounce (0-500ms)</h3>
<div class="value-control">
<input type="number" id="debounceValue" min="0" max="500" step="1" value="45">
</div>
<button class="set-btn" onclick="setDebounce()">Set Debounce</button>
<button class="reset-btn" onclick="resetDebounce()">Reset</button>
</div>
<hr>
<div>
<h3>Press Percent (0-100%)</h3>
<div class="value-control">
<input type="number" id="pressValue" min="0" max="100" step="1" value="95">
</div>
<button class="set-btn" onclick="setPressPercent()">Set Press %</button>
<button class="reset-btn" onclick="resetPressPercent()">Reset</button>
</div>
<hr>
<div>
<h3>Pulse Width (5-200ms)</h3>
<div class="value-control">
<input type="number" id="pulseValue" min="5" max="200" step="1" value="40">
</div>
<button class="set-btn" onclick="setPulseWidth()">Set Pulse Width</button>
<button class="reset-btn" onclick="resetPulseWidth()">Reset to 40ms</button>
</div>
<hr>
<div>
<h3>Polarity (Manual)</h3>
<button class="polarity-btn set-btn" onclick="setPolarity(0)">Decreasing (Default)</button>
<button class="polarity-btn set-btn" onclick="setPolarity(1)">Increasing</button>
<button class="polarity-btn reset-btn" onclick="setPolarity(2)">Auto (from Cal)</button>
</div>
<hr>
<div>
<h3>Global Controls</h3>
<button class="off-btn" onclick="sendCommand('/off', 'Turn OFF')">Turn OFF</button>
<button class="danger-btn" onclick="sendCommand('/reset_state', 'State Reset')">Reset Trigger State</button>
<button class="debug-btn" onclick="sendCommand('/auto_fix', 'Auto Fix')">Auto Fix State</button>
</div>
<div class="status">
<p>Active Mode: <span id="activeMode" class="value">OFF</span></p>
<p>Target SPS: <span id="targetSPS" class="value">10</span> | Burst Shots: <span id="burstShots" class="value">3</span></p>
<p>R2: <span id="voltage" class="value">0.00V</span> | ADC: <span id="adc" class="value">0</span></p>
<p>State: <span id="r2State" class="value">Released</span> <span id="r2Led" class="led"></span></p>
<p>Trigger Press: <span id="triggerPress" class="value">0%</span></p>
<p>Polarity: <span id="polarity" class="value">Unknown</span></p>
<p>Idle ADC: <span id="idleAdc" class="value">0</span> | Pressed ADC: <span id="pressedAdc" class="value">0</span> | Threshold: <span id="threshold" class="value">0</span></p>
<p>Press %: <span id="pressDisplay" class="value">95</span> | Pulse Width: <span id="pulseDisplay" class="value">40</span>ms</p>
<p>Status: <span id="status" class="value">Idle</span></p>
</div>
<hr>
<h3>Calibration</h3>
<button class="cal-btn" onclick="sendCommand('/auto_cal', 'Auto Calibrate')">Smart Auto-Calibrate</button>
<button class="cal-btn" onclick="sendCommand('/cal_release', 'Calibration')">Manual Cal Released</button>
<button class="cal-btn" onclick="sendCommand('/cal_press', 'Calibration')">Manual Cal Pressed</button>
<button class="apply-btn" onclick="sendCommand('/apply_cal', 'Calibration')">Apply Manual Cal</button>
<button class="danger-btn" onclick="sendCommand('/reset_cal', 'Reset Cal')">Reset Calibration to Defaults</button>
</div>
<script>
let ws = null;
function connectWebSocket() {
ws = new WebSocket('ws://' + window.location.hostname + '/ws');
ws.onopen = () => console.log('WS Connected');
ws.onmessage = (event) => {
try {
const data = JSON.parse(event.data);
document.getElementById('activeMode').innerText = data.activeMode;
document.getElementById('targetSPS').innerText = data.targetSPS;
document.getElementById('burstShots').innerText = data.burstShots;
document.getElementById('voltage').innerText = data.voltage.toFixed(2) + 'V';
document.getElementById('adc').innerText = data.adc;
document.getElementById('r2State').innerText = data.r2State;
document.getElementById('r2Led').style.backgroundColor = data.r2State === 'Pressed' ? '#ff4d4d' : '#4dff4d';
document.getElementById('triggerPress').innerText = data.triggerPress.toFixed(0) + '%';
document.getElementById('polarity').innerText = data.polarity;
document.getElementById('idleAdc').innerText = data.idleAdc;
document.getElementById('pressedAdc').innerText = data.pressedAdc;
document.getElementById('threshold').innerText = data.threshold;
document.getElementById('pressDisplay').innerText = data.pressPercent;
document.getElementById('pulseDisplay').innerText = data.pulseWidthMs || 40;
document.getElementById('status').innerText = data.status;
} catch(e) {}
};
ws.onclose = () => setTimeout(connectWebSocket, 800);
}
async function sendCommand(url, action) {
try {
const r = await fetch(url, {method: 'GET'});
const data = await r.json();
alert(data.message || 'OK - ' + action);
} catch(e) { alert('Error: ' + e.message); }
}
async function setCustomSPS() { const v=document.getElementById('spsValue').value; await sendCommand(`/set_continuous_sps?value=${v}`); }
async function setCustomBurst() { const v=document.getElementById('burstValue').value; await sendCommand(`/set_burst_count?value=${v}`); }
async function setJitter() { const v=document.getElementById('jitterValue').value; await sendCommand(`/set_jitter?value=${v}`); }
async function resetJitter() { document.getElementById('jitterValue').value=0; setJitter(); }
async function setHyst() { const v=document.getElementById('hystValue').value; await sendCommand(`/set_hyst?value=${v}`); }
async function resetHyst() { document.getElementById('hystValue').value=10; setHyst(); }
async function setDebounce() { const v=document.getElementById('debounceValue').value; await sendCommand(`/set_debounce?value=${v}`); }
async function resetDebounce() { document.getElementById('debounceValue').value=45; setDebounce(); }
async function setPressPercent() { const v=document.getElementById('pressValue').value; await sendCommand(`/set_press_percent?value=${v}`); }
async function resetPressPercent() { document.getElementById('pressValue').value=95; setPressPercent(); }
async function setPulseWidth() { const v=document.getElementById('pulseValue').value; await sendCommand(`/set_pulse_width?value=${v}`); }
async function resetPulseWidth() { document.getElementById('pulseValue').value=40; setPulseWidth(); }
async function setPolarity(pol) { await sendCommand(`/set_polarity?value=${pol}`); }
connectWebSocket();
</script>
</body>
</html>
)rawliteral";

void loadFromNVS() {
  prefs.begin(NVS_NAMESPACE, true);

  if (prefs.getUChar("magic", 0) == NVS_MAGIC) {
    current_mode = prefs.getInt("current_mode", MODE_OFF);
    last_active_mode = prefs.getInt("last_active_mode", MODE_CONTINUOUS);
    adc_idle = prefs.getInt("adc_idle", DEFAULT_ADC_IDLE);
    adc_pressed = prefs.getInt("adc_pressed", DEFAULT_ADC_PRESSED);
    adc_threshold = prefs.getInt("adc_threshold", DEFAULT_ADC_PRESSED);
    jitter_percentage = prefs.getInt("jitter_percentage", 0);
    hysteresis_percentage = prefs.getInt("hyst_percentage", DEFAULT_HYSTERESIS_PERCENT);
    adc_polarity = prefs.getInt("adc_polarity", DEFAULT_ADC_POLARITY);
    debounce_ms = prefs.getInt("debounce_ms", DEFAULT_DEBOUNCE_MS);
    press_percent = prefs.getInt("press_percent", DEFAULT_PRESS_PERCENT);
    pulse_width_us = prefs.getULong("pulse_width_us", DEFAULT_PULSE_WIDTH_US);
    custom_sps = prefs.getInt("custom_sps", DEFAULT_CUSTOM_SPS);
    custom_burst_count = prefs.getInt("custom_burst", DEFAULT_CUSTOM_BURST);
  }

  prefs.end();

  int travel = abs(adc_idle - adc_pressed);
  if (travel < 150 || travel > 1500) {
    adc_idle = DEFAULT_ADC_IDLE;
    adc_pressed = DEFAULT_ADC_PRESSED;
    adc_polarity = DEFAULT_ADC_POLARITY;
  }

  custom_sps = constrain(custom_sps, 1, 50);
  custom_burst_count = constrain(custom_burst_count, 1, 50);
  continuous_interval_us = 1000000UL / custom_sps;

  recalculateThreshold();
}

void recalculateThreshold() {
  if (adc_idle != adc_pressed) {
    int travel = abs(adc_idle - adc_pressed);
    adc_hysteresis = travel * hysteresis_percentage / 100;

    if (adc_polarity == 0) {
      adc_threshold = adc_idle - (travel * press_percent / 100);
    } else {
      adc_threshold = adc_idle + (travel * press_percent / 100);
    }

    press_level = (adc_polarity == 0) ? LOW : HIGH;
    release_level = (adc_polarity == 0) ? HIGH : LOW;
  } else {
    adc_threshold = adc_idle;
  }
}

void saveToNVS() {
  prefs.begin(NVS_NAMESPACE, false);

  prefs.putUChar("magic", NVS_MAGIC);
  prefs.putInt("current_mode", current_mode);
  prefs.putInt("last_active_mode", last_active_mode);
  prefs.putInt("adc_idle", adc_idle);
  prefs.putInt("adc_pressed", adc_pressed);
  prefs.putInt("adc_threshold", adc_threshold);
  prefs.putInt("jitter_percentage", jitter_percentage);
  prefs.putInt("hyst_percentage", hysteresis_percentage);
  prefs.putInt("adc_polarity", adc_polarity);
  prefs.putInt("debounce_ms", debounce_ms);
  prefs.putInt("press_percent", press_percent);
  prefs.putULong("pulse_width_us", pulse_width_us);
  prefs.putInt("custom_sps", custom_sps);
  prefs.putInt("custom_burst", custom_burst_count);

  prefs.end();
}

void handleAutoCal(AsyncWebServerRequest *request) {
  pinMode(R2_PIN, INPUT);
  gpio_is_output = false;
  delay(50);

  long total = 0;
  for (int i = 0; i < 32; i++) {
    total += analogRead(R2_PIN);
    delay(2);
  }

  adc_idle = total / 32;

  if (adc_idle > 2048) {
    adc_polarity = 1;
    adc_pressed = adc_idle + (4095 - adc_idle) * 0.80;
  } else {
    adc_polarity = 0;
    adc_pressed = adc_idle - (adc_idle * 0.80);
  }

  if (abs(adc_idle - adc_pressed) < 200) {
    adc_pressed = (adc_idle > 2048) ? adc_idle + 400 : adc_idle - 400;
  }

  recalculateThreshold();
  saveToNVS();

  char json[150];
  snprintf(json, sizeof(json), "{\"message\":\"Auto Cal Done. Idle:%d, Pressed:%d, Pol:%d\"}", adc_idle, adc_pressed, adc_polarity);
  request->send(200, "application/json", json);
}

void handleResetCal(AsyncWebServerRequest *request) {
  adc_idle = DEFAULT_ADC_IDLE;
  adc_pressed = DEFAULT_ADC_PRESSED;
  adc_polarity = DEFAULT_ADC_POLARITY;

  recalculateThreshold();
  saveToNVS();

  request->send(200, "application/json", "{\"message\":\"Calibration reset to defaults\"}");
}

void handleCalRelease(AsyncWebServerRequest *request) {
  pinMode(R2_PIN, INPUT);
  gpio_is_output = false;
  delay(50);

  long total = 0;
  for (int i = 0; i < CAL_SAMPLES; i++) {
    total += analogRead(R2_PIN);
    delay(4);
  }

  adc_idle = total / CAL_SAMPLES;

  char json[100];
  snprintf(json, sizeof(json), "{\"message\":\"Released ADC: %d\"}", adc_idle);
  request->send(200, "application/json", json);
}

void handleCalPress(AsyncWebServerRequest *request) {
  pinMode(R2_PIN, INPUT);
  gpio_is_output = false;
  delay(50);

  long total = 0;
  for (int i = 0; i < CAL_SAMPLES; i++) {
    total += analogRead(R2_PIN);
    delay(4);
  }

  adc_pressed = total / CAL_SAMPLES;

  char json[100];
  snprintf(json, sizeof(json), "{\"message\":\"Pressed ADC: %d\"}", adc_pressed);
  request->send(200, "application/json", json);
}

void handleApplyCal(AsyncWebServerRequest *request) {
  if (abs(adc_idle - adc_pressed) < 150 || abs(adc_idle - adc_pressed) > 1500) {
    request->send(400, "application/json", "{\"message\":\"Cal rejected: Travel invalid (150-1500 required).\"}");
    return;
  }

  adc_polarity = (adc_idle > adc_pressed) ? 0 : 1;
  recalculateThreshold();
  saveToNVS();

  char json[180];
  snprintf(json, sizeof(json), "{\"message\":\"Cal applied, polarity=%d, threshold=%d\"}", adc_polarity, adc_threshold);
  request->send(200, "application/json", json);
}

void handleResetState(AsyncWebServerRequest *request) {
  firing = false;
  pulse_active = false;
  pulse_ready = false;
  r2_pressed = false;
  debounce_pending = false;
  debounce_target = false;
  firing_release_count = 0;

  pending_fire_after_unlock = false;
  firing_restart_unlock_time = 0;
  last_firing_stop_time = 0;

  cycle_start_time_us = 0;
  press_phase_start_us = 0;
  last_press_phase_sample_us = 0;
  last_pulse_end_time_us = 0;

  pinMode(R2_PIN, INPUT);
  gpio_is_output = false;

  char json[100];
  snprintf(json, sizeof(json), "{\"message\":\"Trigger state has been reset\"}");
  request->send(200, "application/json", json);
}

void handleSetCustomSPS(AsyncWebServerRequest *request) {
  if (request->hasParam("value")) {
    int v = request->getParam("value")->value().toInt();
    if (v >= 1 && v <= 50) {
      custom_sps = v;
      continuous_interval_us = 1000000UL / custom_sps;
      saveToNVS();
    }
  }

  char json[80];
  snprintf(json, sizeof(json), "{\"message\":\"Continuous SPS set to %d\"}", custom_sps);
  request->send(200, "application/json", json);
}

void handleSetCustomBurst(AsyncWebServerRequest *request) {
  if (request->hasParam("value")) {
    int v = request->getParam("value")->value().toInt();
    if (v >= 1 && v <= 50) {
      custom_burst_count = v;
      saveToNVS();
    }
  }

  char json[80];
  snprintf(json, sizeof(json), "{\"message\":\"Burst count set to %d\"}", custom_burst_count);
  request->send(200, "application/json", json);
}

void handleSetJitter(AsyncWebServerRequest *request) {
  if (request->hasParam("value")) {
    int v = request->getParam("value")->value().toInt();
    if (v >= 0 && v <= 100) {
      jitter_percentage = v;
      saveToNVS();
    }
  }

  char json[50];
  snprintf(json, sizeof(json), "{\"message\":\"Jitter set\",\"jitter\":%d}", jitter_percentage);
  request->send(200, "application/json", json);
}

void handleSetHyst(AsyncWebServerRequest *request) {
  if (request->hasParam("value")) {
    int v = request->getParam("value")->value().toInt();
    if (v >= 0 && v <= 100) {
      hysteresis_percentage = v;
      saveToNVS();
      recalculateThreshold();
    }
  }

  char json[50];
  snprintf(json, sizeof(json), "{\"message\":\"Hyst set\"}");
  request->send(200, "application/json", json);
}

void handleSetDebounce(AsyncWebServerRequest *request) {
  if (request->hasParam("value")) {
    int v = request->getParam("value")->value().toInt();
    if (v >= 0 && v <= 500) {
      debounce_ms = v;
      saveToNVS();
    }
  }

  char json[50];
  snprintf(json, sizeof(json), "{\"message\":\"Debounce set\"}");
  request->send(200, "application/json", json);
}

void handleSetPressPercent(AsyncWebServerRequest *request) {
  if (request->hasParam("value")) {
    int v = request->getParam("value")->value().toInt();
    if (v >= 0 && v <= 100) {
      press_percent = v;
      recalculateThreshold();
      saveToNVS();
    }
  }

  char json[80];
  snprintf(json, sizeof(json), "{\"message\":\"Press %% set\",\"pressPercent\":%d}", press_percent);
  request->send(200, "application/json", json);
}

void handleSetPulseWidth(AsyncWebServerRequest *request) {
  if (request->hasParam("value")) {
    int v = request->getParam("value")->value().toInt();
    if (v >= 5 && v <= 200) {
      pulse_width_us = (unsigned long)v * 1000UL;
      saveToNVS();
    }
  }

  char json[80];
  snprintf(json, sizeof(json), "{\"message\":\"Pulse width set\",\"pulseWidthMs\":%lu}", pulse_width_us / 1000);
  request->send(200, "application/json", json);
}

void handleSetPolarity(AsyncWebServerRequest *request) {
  if (request->hasParam("value")) {
    int v = request->getParam("value")->value().toInt();

    if (v == 0 || v == 1) {
      adc_polarity = v;
      recalculateThreshold();
      saveToNVS();
    } else if (v == 2) {
      if (adc_idle != adc_pressed) {
        adc_polarity = (adc_idle > adc_pressed) ? 0 : 1;
        recalculateThreshold();
        saveToNVS();
      }
    }
  }

  char json[80];
  snprintf(json, sizeof(json), "{\"message\":\"Polarity set to %d\", \"polarity\":%d}", adc_polarity, adc_polarity);
  request->send(200, "application/json", json);
}

void handleActivateContinuous(AsyncWebServerRequest *request) {
  setMode(MODE_CONTINUOUS);

  char json[100];
  snprintf(json, sizeof(json), "{\"message\":\"Continuous Mode Activated (%d SPS)\"}", custom_sps);
  request->send(200, "application/json", json);
}

void handleActivateBurst(AsyncWebServerRequest *request) {
  setMode(MODE_BURST);

  char json[100];
  snprintf(json, sizeof(json), "{\"message\":\"Burst Mode Activated (%d Shots)\"}", custom_burst_count);
  request->send(200, "application/json", json);
}

void handleOff(AsyncWebServerRequest *request) {
  setMode(MODE_OFF);
  request->send(200, "application/json", "{\"message\":\"OFF\"}");
}

void handleAutoFix(AsyncWebServerRequest *request) {
  debounce_ms = 80;
  hysteresis_percentage = 15;

  recalculateThreshold();
  saveToNVS();

  char json[120];
  snprintf(json, sizeof(json), "{\"message\":\"Auto Fix applied: higher debounce & hysteresis.\"}");
  request->send(200, "application/json", json);
}

void sendStatusUpdate() {
  float voltage = (adc_sample / 4095.0f) * VREF;
  const char* r2StateStr = r2_pressed ? "Pressed" : "Released";
  const char* statusStr = firing ? "Firing" : r2StateStr;
  const char* activeModeStr = modeNames[current_mode];
  const char* polStr = (adc_polarity == 0) ? "Decreasing" : "Increasing";

  char json[600];
  snprintf(json, sizeof(json),
    "{\"activeMode\":\"%s\",\"targetSPS\":%d,\"burstShots\":%d,\"voltage\":%.2f,\"adc\":%d,"
    "\"r2State\":\"%s\",\"triggerPress\":%.0f,\"polarity\":\"%s\","
    "\"idleAdc\":%d,\"pressedAdc\":%d,\"threshold\":%d,\"pressPercent\":%d,\"pulseWidthMs\":%lu,"
    "\"status\":\"%s\"}",
    activeModeStr, custom_sps, custom_burst_count, voltage, adc_sample,
    r2StateStr, trigger_press_ratio * 100, polStr,
    adc_idle, adc_pressed, adc_threshold, press_percent, pulse_width_us / 1000,
    statusStr);

  ws.textAll(json);
}

void setMode(int mode) {
  if (mode < 0 || mode >= NUM_MODES) mode = MODE_OFF;

  if (mode != MODE_OFF) last_active_mode = mode;

  current_mode = mode;

  firing = false;
  pulse_active = false;
  pulse_ready = false;
  burst_counter = 0;
  burst_target = 0;
  r2_pressed = false;
  debounce_pending = false;
  debounce_target = false;
  firing_release_count = 0;

  cycle_start_time_us = 0;
  press_phase_start_us = 0;
  last_press_phase_sample_us = 0;
  last_pulse_end_time_us = 0;

  pending_fire_after_unlock = false;
  firing_restart_unlock_time = 0;
  last_firing_stop_time = 0;

  pinMode(R2_PIN, INPUT);
  gpio_is_output = false;

  mode_change_time = millis();
  saveToNVS();
}

unsigned long getFiringRestartLockoutMs() {
  unsigned long interval_ms = continuous_interval_us / 1000UL;
  unsigned long lock = (interval_ms * 3UL) / 4UL;

  if (lock < FIRING_RESTART_LOCKOUT_MIN_MS) lock = FIRING_RESTART_LOCKOUT_MIN_MS;
  if (lock > FIRING_RESTART_LOCKOUT_MAX_MS) lock = FIRING_RESTART_LOCKOUT_MAX_MS;

  return lock;
}

bool firingRestartUnlocked() {
  if (firing_restart_unlock_time == 0) return true;
  return ((long)(current_time - firing_restart_unlock_time) >= 0);
}

void beginFiring() {
  if (current_mode == MODE_OFF) return;

  firing = true;
  pulse_active = false;
  pulse_ready = false;

  firing_release_count = 0;

  cycle_start_time_us = 0;
  press_phase_start_us = 0;
  last_press_phase_sample_us = 0;
  last_pulse_end_time_us = 0;

  pending_fire_after_unlock = false;
  firing_restart_unlock_time = 0;

  if (current_mode == MODE_CONTINUOUS) {
    // Start with a short clean release override, then the first press phase.
    pinMode(R2_PIN, OUTPUT);
    gpio_is_output = true;
    digitalWrite(R2_PIN, release_level);

    next_pulse_time_us = micros() + CONT_INITIAL_RELEASE_US;
  } else {
    next_pulse_time_us = micros();

    burst_target = custom_burst_count;
    burst_counter = burst_target;
  }
}

void stopFiringDuringRelease() {
  firing = false;
  pulse_active = false;
  pulse_ready = false;

  r2_pressed = false;
  debounce_pending = false;
  debounce_target = false;
  firing_release_count = 0;

  pending_fire_after_unlock = false;
  last_firing_stop_time = current_time;
  firing_restart_unlock_time = current_time + getFiringRestartLockoutMs();

  // Hold release state during lockout.
  // This prevents a false release stop from becoming a constant press.
  pinMode(R2_PIN, OUTPUT);
  gpio_is_output = true;
  digitalWrite(R2_PIN, release_level);
}

unsigned long applyJitterMicros(unsigned long base) {
  if (jitter_percentage == 0) return base;

  long range = base * jitter_percentage / 100;
  long jitter = (esp_random() % (2 * range + 1)) - range;

  return max(8000UL, (unsigned long)((long)base + jitter));
}

bool continuousReleaseSample(bool finalSample) {
  if (!firing) return false;
  if (current_mode != MODE_CONTINUOUS) return false;
  if (!pulse_active) return false;
  if (gpio_is_output) return false;

  int reading = 0;

  // For very short pulse widths, use a fast single sample.
  // For normal pulse widths, use a small median filter.
  if (pulse_width_us < 10000UL) {
    reading = analogRead(R2_PIN);
  } else {
    const int N = 4;
    int samples[N];

    for (int i = 0; i < N; i++) {
      samples[i] = analogRead(R2_PIN);
      delayMicroseconds(20);
    }

    std::sort(samples, samples + N);
    reading = samples[N / 2];
  }

  adc_sample = (adc_sample * 3 + reading * 5) / 8;

  int travel = abs(adc_idle - adc_pressed);

  if (travel > 0) {
    if (adc_polarity == 0) {
      trigger_press_ratio = constrain((float)(adc_idle - adc_sample) / travel, 0.0f, 1.0f);
    } else {
      trigger_press_ratio = constrain((float)(adc_sample - adc_idle) / travel, 0.0f, 1.0f);
    }
  } else {
    trigger_press_ratio = (adc_sample < 500) ? 1.0f : 0.0f;
  }

  float release_ratio = CONT_RELEASE_PERCENT / 100.0f;
  bool raw_released = (trigger_press_ratio < release_ratio);

  if (raw_released) {
    firing_release_count++;

    if (firing_release_count >= CONT_RELEASE_SAMPLES) {
      stopFiringDuringRelease();
      return true;
    }
  } else {
    firing_release_count = 0;
  }

  return false;
}

void sampleContinuousPressPhase() {
  if (!firing) return;
  if (current_mode != MODE_CONTINUOUS) return;
  if (!pulse_active) return;
  if (gpio_is_output) return;

  unsigned long now_us = micros();

  if ((unsigned long)(now_us - press_phase_start_us) < CONT_PRESS_CHECK_START_US) return;
  if ((unsigned long)(now_us - last_press_phase_sample_us) < CONT_PRESS_SAMPLE_INTERVAL_US) return;

  last_press_phase_sample_us = now_us;
  continuousReleaseSample(false);
}

void firingLogic() {
  if (!firing || !pulse_ready) return;

  unsigned long now_us = micros();

  if (pulse_active) {
    // End press phase.
    if (current_mode == MODE_CONTINUOUS) {
      // Final release check while still in the scheduled press phase.
      if (continuousReleaseSample(true)) {
        pulse_ready = false;
        return;
      }

      // Start release override phase.
      pinMode(R2_PIN, OUTPUT);
      gpio_is_output = true;
      digitalWrite(R2_PIN, release_level);

      pulse_active = false;
      last_pulse_end_time_us = now_us;

      // Stable SPS timing:
      // Next press is scheduled from the actual press start + interval.
      // This avoids random catch-up pulses and SPS jumps.
      if (cycle_start_time_us == 0) cycle_start_time_us = now_us;

      cycle_start_time_us += continuous_interval_us;

      if (cycle_start_time_us <= now_us) {
        next_pulse_time_us = now_us;
      } else {
        next_pulse_time_us = cycle_start_time_us;
      }

      pulse_ready = false;
    } else if (current_mode == MODE_BURST) {
      if (!gpio_is_output) {
        pinMode(R2_PIN, OUTPUT);
        gpio_is_output = true;
      }

      digitalWrite(R2_PIN, release_level);
      pulse_active = false;

      burst_counter--;

      if (burst_counter > 0) {
        next_pulse_time_us = now_us + applyJitterMicros(BURST_DELAY_US);
      } else {
        firing = false;
        pulse_ready = false;
        last_burst_end = current_time;
        return;
      }

      pulse_ready = false;
    }
  } else {
    // Start press phase.
    if (current_mode == MODE_CONTINUOUS) {
      // Continuous press phase uses the real trigger pass-through.
      // This is the important fix:
      // The GPIO is INPUT during the scheduled press pulse, not during random release gaps.
      pinMode(R2_PIN, INPUT);
      gpio_is_output = false;

      pulse_active = true;
      press_phase_start_us = now_us;
      last_press_phase_sample_us = now_us;

      cycle_start_time_us = now_us;
      next_pulse_time_us = now_us + pulse_width_us;

      pulse_ready = false;
    } else if (current_mode == MODE_BURST) {
      if (!gpio_is_output) {
        pinMode(R2_PIN, OUTPUT);
        gpio_is_output = true;
      }

      digitalWrite(R2_PIN, press_level);
      pulse_active = true;

      next_pulse_time_us = now_us + pulse_width_us;
      pulse_ready = false;
    }
  }
}

void checkR2Trigger() {
  if (firing) return;

  // During release-stop lockout, keep release state and do not read the trigger yet.
  // This prevents extra edges immediately after a release stop.
  if (!firingRestartUnlocked()) {
    if (!gpio_is_output) {
      pinMode(R2_PIN, OUTPUT);
      gpio_is_output = true;
      digitalWrite(R2_PIN, release_level);
    }
    return;
  }

  if (gpio_is_output) {
    pinMode(R2_PIN, INPUT);
    gpio_is_output = false;
    delayMicroseconds(INPUT_SETTLE_US);
  }

  int samples[ADC_AVG_SAMPLES];

  for (int i = 0; i < ADC_AVG_SAMPLES; i++) {
    samples[i] = analogRead(R2_PIN);
    delayMicroseconds(50);
  }

  std::sort(samples, samples + ADC_AVG_SAMPLES);
  int new_sample = samples[ADC_AVG_SAMPLES / 2];

  adc_sample = (adc_sample * 4 + new_sample * 4) / 8;

  int travel = abs(adc_idle - adc_pressed);

  if (travel > 0) {
    if (adc_polarity == 0) {
      trigger_press_ratio = constrain((float)(adc_idle - adc_sample) / travel, 0.0f, 1.0f);
    } else {
      trigger_press_ratio = constrain((float)(adc_sample - adc_idle) / travel, 0.0f, 1.0f);
    }
  } else {
    trigger_press_ratio = (adc_sample < 500) ? 1.0f : 0.0f;
  }

  int press_th = adc_threshold;
  int rel_th = adc_polarity ? (adc_threshold - adc_hysteresis) : (adc_threshold + adc_hysteresis);

  bool raw_pressed = adc_polarity
    ? (adc_sample > (r2_pressed ? rel_th : press_th))
    : (adc_sample < (r2_pressed ? rel_th : press_th));

  if (current_time - mode_change_time < 100) {
    r2_pressed = raw_pressed;
    debounce_pending = false;
    firing = false;
    pending_fire_after_unlock = false;
    return;
  }

  if (raw_pressed && current_mode == MODE_BURST && (current_time - last_burst_end < BURST_COOLDOWN)) {
    raw_pressed = false;
  }

  int eff_deb = (current_mode == MODE_BURST) ? BURST_DEBOUNCE_MS : debounce_ms;

  if (raw_pressed != r2_pressed) {
    if (!debounce_pending || raw_pressed != debounce_target) {
      debounce_pending = true;
      debounce_target = raw_pressed;
      debounce_start = current_time;
    } else if (current_time - debounce_start >= (unsigned long)eff_deb) {
      bool was_pressed = r2_pressed;
      r2_pressed = debounce_target;
      debounce_pending = false;

      if (r2_pressed && !was_pressed) {
        if (current_mode != MODE_OFF) {
          if (!firingRestartUnlocked()) {
            pending_fire_after_unlock = true;
          } else {
            beginFiring();
          }
        }
      } else if (!r2_pressed && was_pressed) {
        firing = false;
        firing_release_count = 0;
        pending_fire_after_unlock = false;
      }
    }
  } else {
    debounce_pending = false;
  }

  if (!firing &&
      pending_fire_after_unlock &&
      r2_pressed &&
      current_mode != MODE_OFF &&
      firingRestartUnlocked()) {
    if (current_mode != MODE_BURST || (current_time - last_burst_end >= BURST_COOLDOWN)) {
      beginFiring();
    }
  }
}

void checkInactivity() {
  if (current_time - last_inactivity_time < INACTIVITY_CHECK) return;

  if (r2_pressed) {
    inactivity_remaining = INACTIVITY_TIMEOUT;
  } else if (inactivity_remaining > 0) {
    inactivity_remaining = max(0UL, inactivity_remaining - INACTIVITY_CHECK);
    if (inactivity_remaining == 0 && current_mode != MODE_OFF) setMode(MODE_OFF);
  }

  last_inactivity_time = current_time;
}

void updateLED() {
  digitalWrite(STATUS_LED_PIN, (current_mode == MODE_OFF) ? HIGH : ((current_time % 1000) < 120 ? LOW : HIGH));
}

void onWsEvent(AsyncWebSocket *server, AsyncWebSocketClient *client, AwsEventType type, void *arg, uint8_t *data, size_t len) {}

void setupWiFi() {
  WiFi.mode(WIFI_AP);
  WiFi.softAP("RapidFireMod_v6.5", "");
  WiFi.softAPConfig(IPAddress(192, 168, 4, 1), IPAddress(192, 168, 4, 1), IPAddress(255, 255, 255, 0));
}

void handleRoot(AsyncWebServerRequest *request) {
  request->send(200, "text/html", index_html);
}

void setup() {
  Serial.begin(115200);

  pinMode(STATUS_LED_PIN, OUTPUT);
  digitalWrite(STATUS_LED_PIN, HIGH);

  pinMode(R2_PIN, INPUT);
  gpio_is_output = false;

  loadFromNVS();
  setupWiFi();

  ws.onEvent(onWsEvent);
  server.addHandler(&ws);

  server.on("/", HTTP_GET, handleRoot);
  server.on("/cal_release", HTTP_GET, handleCalRelease);
  server.on("/cal_press", HTTP_GET, handleCalPress);
  server.on("/apply_cal", HTTP_GET, handleApplyCal);
  server.on("/auto_cal", HTTP_GET, handleAutoCal);
  server.on("/reset_cal", HTTP_GET, handleResetCal);
  server.on("/activate_continuous", HTTP_GET, handleActivateContinuous);
  server.on("/activate_burst", HTTP_GET, handleActivateBurst);
  server.on("/set_continuous_sps", HTTP_GET, handleSetCustomSPS);
  server.on("/set_burst_count", HTTP_GET, handleSetCustomBurst);
  server.on("/set_jitter", HTTP_GET, handleSetJitter);
  server.on("/set_hyst", HTTP_GET, handleSetHyst);
  server.on("/set_debounce", HTTP_GET, handleSetDebounce);
  server.on("/set_press_percent", HTTP_GET, handleSetPressPercent);
  server.on("/set_pulse_width", HTTP_GET, handleSetPulseWidth);
  server.on("/set_polarity", HTTP_GET, handleSetPolarity);
  server.on("/off", HTTP_GET, handleOff);
  server.on("/reset_state", HTTP_GET, handleResetState);
  server.on("/auto_fix", HTTP_GET, handleAutoFix);

  server.begin();

  setMode(current_mode);

  Serial.println("RapidFire v6.5 Beta - http://192.168.4.1");
  Serial.println("FIXED: Continuous SPS stabilized");
}

void loop() {
  current_time = millis();
  unsigned long current_micros = micros();

  pulse_ready = (current_micros >= next_pulse_time_us);
  firingLogic();

  if (current_time - last_adc_time >= ADC_SAMPLE_INTERVAL) {
    if (!firing) {
      checkR2Trigger();
    }
    last_adc_time = current_time;
  }

  // Release sensing happens only during the scheduled continuous press phase.
  sampleContinuousPressPhase();

  if (current_time - last_server_time >= SERVER_INTERVAL) {
    ws.cleanupClients();
    sendStatusUpdate();
    last_server_time = current_time;
  }

  checkInactivity();
  updateLED();
  yield();
}



