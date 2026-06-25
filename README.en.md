<div align="center">

### 🌐 README Languages

[![한국어](https://img.shields.io/badge/한국어-blue?style=for-the-badge)](README.md)
[![English](https://img.shields.io/badge/English-yellow?style=for-the-badge)](README.en.md)
[![日本語](https://img.shields.io/badge/日本語-red?style=for-the-badge)](README.jp.md)

</div>

# 🌱 Smart Farm IoT System

## < Integrated Smart Farm Management System — Built in Phases >
<br/>
**Phase 1: IoT → Phase 2: Web Development → Phase 3: Mobile App Development**

<img width="1818" height="763" alt="Image" src="https://github.com/user-attachments/assets/74a7ab88-bedf-4768-b77d-3128926f191a" />

---

## 🔗 Repository Links

| Category | Description | Link |
|----------|-------------|------|
| 💾 **Backend (Phase 1)** | Spring Boot-based backend server & API | [🔗 backend_root](https://github.com/launch-kdy/backend_root) |
| 💻 **Web Frontend (Phase 2)** | React-based environment monitoring web dashboard | [🔗 frontend_root](https://github.com/launch-kdy/frontend_root) |
| 📱 **Mobile App (Phase 3)** | React Native-based smart farm mobile app | [🔗 frontend_root_rn](https://github.com/launch-gdy/frontend_root_rn) |

---

## 📘 Project Overview

**root** is an **IoT-based smart farm environment control and monitoring system** for plant cultivation.  
Starting with sensor data collection and automated control via Raspberry Pi,  
this platform integrates **Spring Boot** backend with **React / React-Native** to enable  
**real-time farm monitoring and control** across web and mobile environments.

### Team Name: root

**Meaning:**
- **r** (green sprout) → Symbolizes the beginning of smart farm development and team growth `#72A84F`
- **o** (gear) → Symbolizes the collaborative effort and interlocking work of team members `#505640`
- **o** (circuit) → Symbolizes technical knowledge including IoT, data, smart control, and coding `#72A84F`
- **t** (root) → Symbolizes contribution to agricultural advancement and a stable, rooted foundation for plant health `#B3703E`

---

## 🧩 System Structure
```
📡 IoT Device Raspberry Pi (Phase 1)
 ├─ DHT22 (Temperature & Humidity)
 ├─ FC-28 (Soil Moisture)
 ├─ LDR (Illuminance)
 ├─ PIR + Buzzer (Security)
 ├─ Fan Motor / Water Pump / LED Auto Control
 └─ Python + MariaDB + WebSocket

💻 Web Platform (Phase 2)
 ├─ Spring Boot REST API
 ├─ React Dashboard (Chart.js)
 ├─ IoT Data Visualization / Control Log Management

📱 Mobile App (Phase 3)
 ├─ React Native + Expo
 ├─ Real-time Control (WebSocket)
 ├─ Environmental Data & Notification UI
```

---

## ⚙️ Phase 1: IoT Automated Control System (Raspberry Pi)

### 📍 Core Features

| Feature | Description | Interval / Condition |
|---------|-------------|----------------------|
| 🌡️ **Temperature Management** | Measure temperature with DHT22 sensor → Save to DB | Every 60 min |
| 💧 **Humidity Management** | Measure ambient humidity → Save to DB | Every 60 min |
| 🌱 **Soil Moisture Management** | Measure soil moisture with FC-28 sensor → Auto pump control | Every 60 min |
| ☀️ **Illuminance Management** | Measure light intensity with LDR sensor → Auto LED control | Real-time |
| 🎐 **Fan Motor Control** | Auto-activate when temperature ≥ 28°C | Temperature condition |
| 🚨 **Motion Detection & Buzzer Alarm** | Detect abnormal movement with PIR sensor | Real-time |
| 🌐 **Real-time Remote Control (WebSocket)** | Manual/auto control and data query from web/app | Always connected |

### 🔌 Sensor & Component Configuration

| Sensor / Device | Data Type | MCP3008 Required | Description |
|-----------------|-----------|------------------|-------------|
| **DHT22** (Temp & Humidity) | Digital | ❌ Not required | Read directly via GPIO |
| **FC-28** (Soil Moisture) | Analog | ✅ Required | Converted via MCP3008 |
| **LDR** (Illuminance) | Analog | ✅ Required | Converted via MCP3008 (uses spi.xfer2()) |
| **PIR** Sensor (Motion) | Digital | ❌ Not required | Motion detection → 0/1 |
| **Buzzer** | Digital | ❌ Not required | GPIO output control |
| **Fan Motor** | PWM | ❌ Not required | Uses L9110S driver |
| **MCP3008** | - | - | **Analog-to-Digital Converter (ADC)** |

> 💡 **Key Note:**  
> "Analog sensors can't be read directly by the Raspberry Pi, so data was received through the MCP3008; digital sensors could be connected and read directly."

### ⚡ GPIO Pin Assignment

| Sensor / Device | GPIO Pin | Purpose | Control Method |
|-----------------|----------|---------|----------------|
| DHT22 | 14 | Temp & Humidity measurement | Digital signal |
| FC-28 | MCP3008 CH0 | Soil moisture measurement | ADC conversion |
| LDR | MCP3008 CH1 | Illuminance measurement | ADC conversion |
| LED | 2 | Lighting control | Digital output |
| PIR (HC-SR501) | 17 | Motion detection | Digital input |
| Buzzer | 3 | Alarm output | PWM frequency |
| Fan Motor | 22, 27 | Ventilation control | PWM speed (L9110S) |

### 🔄 Control Logic

| Sensor | Threshold | Action | Control Device |
|--------|-----------|--------|----------------|
| Light sensor | < 250 | LED ON | GPIO 2 |
| Light sensor | ≥ 250 | LED OFF | GPIO 2 |
| Temperature sensor | ≥ 28°C | Fan ON | GPIO 22, 27 |
| Temperature sensor | < 28°C | Fan OFF | GPIO 22, 27 |
| PIR sensor | Detected | Buzzer ON | GPIO 3 |
| PIR sensor | Not detected | Buzzer OFF | - |

### 🧵 Thread Operation Intervals (Integrated System)

| Thread Name | Interval | Sensor | Control Device | DB Save | Main Function |
|-------------|----------|--------|----------------|---------|---------------|
| **Light + LED** | 2 sec | LDR light sensor | LED | ❌ | Real-time lighting control |
| **Environment + DB** | 60 min | DHT22, FC-28 | Fan motor | ✅ | Environment monitoring / data storage |
| **Motion + Buzzer** | 1 sec | PIR | Buzzer | ❌ | Real-time security alert |
| **WebSocket** | Real-time | - | All | ❌ | Remote control communication |

### 📁 Key File Structure

| Filename | Role | Description |
|----------|------|-------------|
| `multi_sensor.py` | Sensor data collection | Collect and convert temp/humidity, illuminance, soil moisture data |
| `multi_control.py` | Automated control | LED, fan, water pump auto-control logic |
| `motion_detector.py` | Security feature | PIR sensor-based motion detection and buzzer alert |
| `fan_motor.py` | Fan control | L9110S driver-based fan motor control |
| `config_manager.py` | Settings management | Crop-specific threshold / auto & manual mode management |
| `websocket_server.py` | Real-time communication | WebSocket communication with clients (web/app) |
| `integrated_system.py` | **Integrated system main** | Manages all threads (sensors, control, alerts, communication) |

---

## 💻 Phase 2: Web-based Monitoring System (Spring + React)

### 📦 Backend (Spring Boot)

#### Project Structure
```
backend_root/
 ├─ src/main/java/com/root/
 │   ├─ common/
 │   │   ├─ config/         # Swagger, MyBatis, CORS
 │   │   ├─ exception/      # GlobalExceptionHandler
 │   │   └─ util/
 │   ├─ domain/
 │   │   ├─ environment/    # Environmental data (temp/humidity/soil/light)
 │   │   │   ├─ controller/
 │   │   │   ├─ service/
 │   │   │   ├─ repository/
 │   │   │   └─ dto/
 │   │   └─ device/         # Device status (fan/pump/LED/motion)
 │   └─ RootApplication.java
 └─ src/main/resources/
     ├─ mapper/             # MyBatis XML
     └─ application.yml
```

#### Key Features
- **REST API**: `/api/growings`, `/api/devices`
- **MyBatis Mapper**-based CRUD
- **Data statistics** / period-based query functionality
- **Swagger/OpenAPI** documentation

### 🖥️ Frontend (React)

- **Chart.js**-based environmental data visualization
- **Dashboard UI**: Real-time query for temperature, humidity, illuminance
- Device status logs / operation counts / control button UI

---

## 📱 Phase 3: Mobile App (React Native)

### Key Features
- **Expo Router**-based native app structure
- **WebSocket** real-time communication for remote control
- **SecureStore** login and user permission persistence
- Daily operation statistics / notifications / control log viewing

### Key Screens

| Screen | Description |
|--------|-------------|
| 🌡️ **Home** | Real-time sensor data display |
| ⚙️ **Control** | Manual/auto control for fan, LED, pump |
| 📊 **Stats** | Daily operation count and DB-connected charts |
| 🔔 **Alert** | Real-time notifications on motion detection |

---

## 👨‍💻 Individual Responsibilities

### 📱 Frontend (React Native)

#### 🏠 **HomeScreen (Main Dashboard) Development**

**Implemented Features:**
- WebSocket-based real-time sensor data reception and display
- Spring Boot API integration for daily device operation count aggregation
- 60-second interval auto-refresh logic
- Temperature / humidity / soil moisture / illuminance sensor data visualization
- LED, fan motor, water pump, motion detection operation statistics
- Connection status check and loading screen handling
- Real-time timestamp display using the dayjs library

**Technical Implementation:**
- Real-time data communication via WebSocket custom hook (`useWebSocket`)
- REST API calls and data filtering logic
- Responsive grid layout design using Flexbox
- Sensor-specific color themes (temperature: red, humidity: purple, soil: blue, light: green)
- Loading state UI using ActivityIndicator
- Auto-refresh and memory leak prevention with useEffect hook

---

#### 🌙 **Dark Mode System Implementation**

**Implemented Features:**
- Light/dark theme toggle functionality
- AsyncStorage-based user theme preference persistence
- Global theme state management using Context API
- Consistent theme application across all app screens

**Technical Implementation:**
- `ThemeContext` creation and Provider pattern implementation
- Persistent theme storage via AsyncStorage
- Dynamic style object generation for per-theme color changes
- Automatic theme restoration from storage on app restart

---

#### 🌐 **Multi-language Support System Implementation**

**Implemented Features:**
- Support for 3 languages: Korean, English, Japanese
- Language switching from the settings screen
- AsyncStorage-based language preference persistence
- Multi-language text handling across the entire app

**Technical Implementation:**
- i18n language resource file structure design and implementation
- Global language state management via `LanguageContext`
- Persistent language storage via AsyncStorage
- Real-time language switching via dynamic text rendering
- Translation resource mapping and application per screen

---

### 💻 Frontend (React Web)

#### 🏢 **CompanyProfile (Company Introduction Page) Development**

**Implemented Features:**
- Scroll-based interactive animation implementation
- Section-by-section layout design for brand storytelling
- Raspberry Pi hardware introduction and system structure visualization
- Responsive image gallery and slider implementation
- Real-time scroll progress calculation and animation triggers

**Technical Implementation:**
- Scroll position tracking using `useRef` and `getBoundingClientRect()`
- Per-section scroll progress state management with `useState`
- Smooth fade-in / slide animations using CSS Transform and Opacity
- Dynamic text animation using Marquee effect
- Differentiated animations per section via conditional rendering
- Scroll performance optimization with passive event listeners

---

#### 🎨 **Advanced CSS Animations & Styling**

**Implemented Features:**
- Glassmorphism-style blur effect card design
- Gradient backgrounds and text clipping effects
- Keyframe animations for per-element entrance effects
- Image slider and 3D rotation effects
- Dynamic background animations based on weather (day/night mode)

**Technical Implementation:**
- Component-scoped styling using CSS Modules
- Complex animation sequences via `@keyframes`
- Glass texture effect using `backdrop-filter` and `blur()`
- Gradient text using `linear-gradient` and `background-clip: text`
- Dynamic style control using CSS variables (`--variable`)
- `transform: scale()` and `box-shadow` transition on mouse hover

---

#### 📊 **UserControl (Sensor Dashboard) Development**

**Implemented Features:**
- Real-time sensor data query via Spring Boot API integration
- Card-style UI for each sensor: temperature / humidity / soil / illuminance
- Motion detection statistics and last detection time display
- Weather widget integration and time-of-day background animations
- Dynamic style changes based on sensor threshold values

**Technical Implementation:**
- Asynchronous REST API communication using Axios
- Data loading on component mount via `useEffect` hook
- Data processing using array methods (`map`, `filter`, `at()`)
- Responsive card layout using CSS Grid
- Visual feedback by sensor state via conditional class binding
- Time-based weather widget background and animation transitions

---

#### ✨ **Special Effects & Interactions**

**Implemented Features:**
- Shooting star animation (night mode)
- Firefly and insect animations (day/night mode)
- Reed swaying effect
- Cloud sync loader animation
- Sunlight sparkle and siren flash effects

**Technical Implementation:**
- Multiple animation elements generated using `Array.from()` and `map()`
- Random positions and delays using `Math.random()`
- Asynchronous animations via CSS `animation-delay` and `animation-duration`
- Complex vector animation using SVG-based cloud loader
- Light effects using `radial-gradient` and `filter`
- Natural movement using `transform-origin` and `rotate()`

---

### 💾 Backend (Spring Boot)

#### 🌱 **Growing Module (Environmental Sensor Data API) Development**

**Implemented Features:**
- Environmental sensor data full query API (`GET /growings`)
- Last 7 days sensor data query API (`GET /growings/weekly`)
- Sensor data reverse-order query API (`GET /growings/rev`)
- ResponseEntity-based exception handling and HTTP status code management

**Technical Implementation:**
- **Controller**: RESTful API endpoint design and implementation
- **Service**: Business logic layer separation
- **Mapper**: DB access via MyBatis interface
- **DTO**: Timestamp handling based on LocalDateTime
- Dependency injection using Lombok `@RequiredArgsConstructor`
- Request/response tracking via SLF4J logging

**API Endpoints:**

| Method | URI | Description | Return Type |
|--------|-----|-------------|-------------|
| GET | `/growings` | Query all sensor data | `List<GrowingDTO>` |
| GET | `/growings/weekly` | Query last 7 days data | `List<GrowingDTO>` |
| GET | `/growings/rev` | Query sensor data in reverse order | `List<GrowingDTO>` |

**DTO Structure:**
```java
@Data
public class GrowingDTO {
  private int temperNum;            // Data unique ID
  private float temper;             // Temperature (℃)
  private float humidity;           // Humidity (%)
  private float soilHumidity;       // Soil moisture (%)
  private int illumination;         // Illuminance
  private LocalDateTime createDate; // Measurement time
}
```

---

#### 🚨 **MotionBuzzer Module (Security & Device Control API) Development**

**Implemented Features:**
- Full motion detection log query API (`GET /motions`)
- Detection statistics and last detection time API (`GET /motions/stats`)
- Daily device operation count API (`GET /motions/today`)
- Exception handling and standardized error messages

**Technical Implementation:**
- **Controller**: Security event and device status query API design
- **Service**: Statistics aggregation logic implementation
- **Mapper**: Complex query handling via MyBatis
- **DTO**: Additional statistics fields (`detectedCount`, `lastDetected`) design
- Stable error handling via try-catch blocks
- HTTP status code-based response handling

**API Endpoints:**

| Method | URI | Description | Return Type |
|--------|-----|-------------|-------------|
| GET | `/motions` | Query all motion detection logs | `List<MotionBuzzerDTO>` |
| GET | `/motions/stats` | Detection count + last detection time stats | `MotionBuzzerDTO` |
| GET | `/motions/today` | Query today's device operation count | `List<MotionBuzzerDTO>` |

**DTO Structure:**
```java
@Data
public class MotionBuzzerDTO {
  // Basic log fields
  private int statusId;               // Status log ID
  private LocalDateTime timestamp;    // Detection time
  private boolean motionDetected;     // Motion detection status (true/false)
  
  // Device control status fields
  private int fanMotor;               // Fan motor operation count
  private int waterPump;              // Water pump operation count
  private int ledLight;               // LED operation count
  
  // Statistics fields
  private Integer detectedCount;      // Total detection count
  private LocalDateTime lastDetected; // Last detection time
}
```

**Key Characteristics:**
- Multi-purpose single DTO handling both log query and statistics query
- Clear expression of motion detection status with boolean type
- Integrated management of per-device operation counts in a single response
- Serves as the daily statistics data API for the mobile app HomeScreen

---

#### 🔧 **Backend Architecture Design Contributions**

**Implementation:**
- **Layered architecture** applied: Controller → Service → Mapper → DB
- **DTO-based data transfer**: Prevents entity exposure and optimizes API responses
- **MyBatis Mapper interface** design for SQL separation and improved maintainability
- **Lombok usage**: Minimizes boilerplate code
- **RESTful API design principles**: Resource-oriented URIs, appropriate HTTP method usage
- **Standardized exception handling**: Consistent error responses via ResponseEntity

**Code Quality:**
- Debugging and monitoring support via SLF4J logger
- Immutability guaranteed through constructor injection
- Stable exception handling via try-catch blocks
- Clear distinction of HTTP status codes (200 OK, 500 Internal Server Error)

---

## 🧠 Tech Stack

| Category | Tech Stack |
|----------|------------|
| **IoT / Embedded** | Python 3.11, RPi.GPIO, spidev, adafruit_dht, asyncio, websockets |
| **Backend** | Spring Boot (Java 17), MyBatis, MariaDB |
| **Frontend (Web)** | React (Vite), Chart.js, Axios, CSS Modules |
| **Mobile App** | React Native, Expo, SecureStore, WebSocket, AsyncStorage |
| **Infra** | Raspberry Pi 3B+, GitHub, VS Code Remote SSH |
| **Design** | Figma, Glassmorphism / Apple-style UI |

---

## 🗂️ Database Design (ERD)

<img width="1959" height="720" alt="Image" src="https://github.com/user-attachments/assets/b3fc07da-fefc-417a-8c74-a7fddd359533" />

### Main Tables

#### 📊 GROWING_ENVIRONMENT (Environmental Data)
| Column | Type | Description |
|--------|------|-------------|
| TEMPER_NUM | INT (PK, AUTO_INCREMENT) | Unique ID |
| TEMPER | FLOAT | Temperature (℃) |
| HUMIDITY | FLOAT | Humidity (%) |
| SOIL_HUMIDITY | FLOAT | Soil moisture (%) |
| ILLUMINATION | INT | Illuminance |
| CREATE_DATE | DATETIME | Data save time |

#### ⚙️ DEVICE_STATUS (Control Device Status)
| Column | Type | Description |
|--------|------|-------------|
| STATUS_ID | INT (PK, AUTO_INCREMENT) | Status log ID |
| TIMESTAMP | DATETIME | Record time |
| MOTION_DETECTED | TINYINT(1) | PIR detection status (0/1) |
| FAN_MOTOR | TINYINT(1) | Fan status (0/1) |
| WATER_PUMP | TINYINT(1) | Pump status (0/1) |
| LED_LIGHT | TINYINT(1) | Lighting status (0/1) |

---

## 🚀 Results & Future Plans

| Phase | Key Achievement | Notes |
|-------|----------------|-------|
| **Phase 1** | Raspberry Pi IoT automated control system built | Sensor & motor control complete |
| **Phase 2** | Spring-React web integration / DB connection | Environmental data visualization |
| **Phase 3** | WebSocket-based real-time remote control app complete | Expo app running successfully |
| **Future Plans** | Cloud DB migration (AWS RDS / Firebase), AI crop condition prediction | Research & expansion planned |

---

## 👣 License & Contact

### 🏁 License

This project is a partially restructured version of the **root team collaboration project**,  
published for **learning and portfolio purposes**.

- Sensitive code (server configuration, DB credentials, API keys, etc.) has been kept private.
- Commercial use and redistribution are prohibited.
- The purpose of this repository is limited to **sharing individual skills and the development process**.

**© 2025 DOYOUNG KIM (金 度榮) (root Team)**  
All Rights Reserved.

---

### 🧠 Credits

**Project Team:** root 🌱

**Development Period:** 2025.09.11 ~ 2025.11.14

**Institution:** K-Digital Training

**Project Goal:** IoT-based crop environment monitoring and automated control system for smart farming

---

### 📬 Contact

| Category | Details |
|----------|---------|
| 👤 **Name** | DOYOUNG KIM (金 度榮) |
| 💻 **GitHub** | [github.com/launch-kdy](https://github.com/launch-kdy) |
| 💌 **Email** | bre.xx.bpm@gmail.com |
| 🏫 **Affiliation** | 桜美林大学 / K-Digital Training |

---

🔒 **This project is a limited public release for portfolio purposes.**  
Sensitive information such as DB credentials and API keys from the actual operating environment has been excluded.  
Only the technical structure, design, and portions of the core code are publicly available.
