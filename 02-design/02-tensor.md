# 🌡️ Temperature Sensor – Design Documentation

## I- Project Context
This project is developed as part of an IoT system involving:
- A **mobile application** (Flutter) to visualize and send data
- A **web server** to store and distribute commands/data
- A **"thing"**, namely a temperature & humidity sensor simulator

We act as a consulting firm helping clients connect physical devices to the internet. This document focuses on the **temperature sensor** component.

---

## II- Objectives & Requirements

### A- Core Functional Requirements
| Ref | Description |
|-----|-------------|
| NTS1 | The sensor must send telemetry data (temperature) to the server |
| NTS2 | Temperature is sent every 10s (active) or 20s (sleep) |
| NTS3 | Data must be user-editable with 0.1°C precision |
| NTS4 | Temperature is always sent in °C |
| NTS5 | The GUI must display the temperature unit from server attributes |
| NTS6 | Humidity data is also sent |
| NTS7 | Humidity is sent every 20s (active) or 50s (sleep) |
| NTS8 | Humidity must be user-editable with 1% precision |
| NTS9 | Humidity is always in percentage |
| NTS10 | The UI must include a toggle between active/sleep modes |
| NTS11 | The sensor polls the server every 10s (active) or 20s (sleep) |
| NTS+1 | Data transmission over WebSocket as an enhancement |

---

## III- Architecture Overview

```mermaid
graph TD
    A[User Interface<br>(Flutter App)]
    B[AutoModeService / ManualModeService]
    C[ThingClient<br>(WebSocket sender)]
    D[FakeServer<br>(WebSocket Server)]

    A --> B
    B --> C
    C --> D
```

---

## IV- Technologies and Justifications

| Component        | Technology | Reason |
|------------------|------------|--------|
| Mobile App       | Flutter    | Cross-platform, UI flexibility |
| Communication    | WebSocket  | Real-time, efficient, bidirectional |
| Server (local)   | Dart (FakeServer) | Quick testing without needing a hosted backend |

---

## V- Data Flow Explanation

### A- Transmission Logic

1. **Sensor Mode Logic** (handled by `AutoModeService`)
   - Switch between active/sleep mode
   - Different intervals for temp/humidity simulation
   - Data is encapsulated into a `TelemetryPoint`

2. **Sending Data via WebSocket**
   - `ThingClient` encodes the telemetry point into JSON:
```json
{
  "thingId": "flutter_app_tensor_01",
  "temperature": 23.5,
  "humidity": 58,
  "timestamp": "2025-04-06T12:00:00Z"
}
```
   - Sent via WebSocket to the `FakeServer`

3. **Fake Server**
   - Listens on `ws://127.0.0.1:5001`
   - Responds to received data and can echo back values

### B- Receiving Server Attributes
- Polled using HTTP (via `FakeServer.getAttributes()`)
- Example response:
```json
{ "temperatureUnit": "°C" }
```

---

## VI- Simulation Strategy

- During development, the system was isolated from the actual server.
- The `FakeServer` mimics real server behavior:
   - Accepts WebSocket connections
   - Returns mock attributes
   - Logs messages for verification

---

## VII- Testing & Validation

- Verified switching between modes and message frequency
- Used local IP for networking (`127.0.0.1`)
- Verified structured JSON messages
- Checked UI state update and responsiveness

---

## VIII- Future Enhancements

- Replace FakeServer with a real hosted server
- Persist telemetry to a real database
- Add support for multiple sensors/devices
- Secure WebSocket communication (wss://)
- Add configuration screen to adjust intervals and thresholds

---

## IX- Folder Location
This documentation belongs to the `02-design/` folder and complements:
- `01-specification/03-temperature-sensor/README.md`
- `mobile_app-server-api.yaml`
- `things-server-api.yaml`

## X Extended Explanation: Transmission Workflow & Justifications

### 1. WebSocket vs HTTP – Why WebSocket?
The system prioritizes **WebSocket** for telemetry because it allows:
- Real-time communication (low latency),
- Persistent bidirectional connection,
- Reduced overhead compared to repeated HTTP requests.

**HTTP fallback** was considered but not implemented in this version to keep the stack simple. Future updates may introduce POST endpoints as a backup mechanism.

---

### 2. Data Transmission Process – Step by Step

#### a- Authentication
- After establishing the WebSocket, the app sends a JSON payload:
```json
{ "id": "flutter_app_tensor_01", "key": "secret_api_key" }
```
- The server replies with a confirmation if accepted.

#### b- Periodic Transmission (Telemetry)
Depending on the selected mode:
- **Active**: Sends telemetry every 10s (temp) and 20s (hum)
- **Sleep**: Sends every 20s (temp) and 50s (hum)

Each payload contains:
```json
{
  "thingId": "flutter_app_tensor_01",
  "temperature": 22.7,
  "humidity": 44,
  "timestamp": "2025-04-06T13:55:00.123Z"
}
```

#### c- Server-Side Handling
The `FakeServer` (written in Dart) mimics:
- Receiving and logging WebSocket data,
- Replying with acknowledgments and attributes,
- Broadcasting to other clients (if needed).

---

### 3. UI Design & Behavior

The user interface plays a critical role in **usability and flexibility**.

#### Manual Mode
- User can enter values for temperature and humidity.
- Useful for test cases and validation.

#### Auto Mode
- Values are generated randomly at the defined intervals.
- Ideal for simulating live environments.

#### Mode Toggle
Switching between **Active** and **Sleep** directly influences:
- Frequency of data generation
- Frequency of polling attributes from the server

This ensures the UI reflects not just values, but also the **state of operation** of the system.

---

### 4. Reasoning Behind Choices

| Design Choice | Justification |
|---------------|----------------|
| WebSocket | Real-time, responsive communication |
| Flutter | Cross-platform development, rich UI |
| Simulation (Manual + Auto) | Allows testing edge cases and timing |
| FakeServer | Enables development without full backend |
| Telemetry structure | Future-proof JSON format with timestamp & IDs |

---