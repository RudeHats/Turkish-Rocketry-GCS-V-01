# Ground Control Station (GCS) - Team VAJRA

Welcome to the Team VAJRA Ground Control Station (GCS) software repository, developed for the TEKNOFEST A4 International Category. This software provides a robust and reliable interface to receive real-time telemetry from the rocket, visualize it on a modern dashboard, and securely relay data to the Referee Ground Station (RGS).

## 🚀 Features

- **RX Port (Data Input)**: Reliably receives standard 78-byte telemetry packets from the rocket's radio systems.
- **TX Port (Data Relay)**: Automatically forwards aggregated telemetry to the Referee Ground Station at the competition-required 19200 baud rate.
- **Real-Time Dashboard**: Features live visualization of metrics including altitude, trajectory, IMU (Inertial Measurement Unit) data, and GPS tracking maps.
- **Dynamic Communication Modes**: Supports continuous switching between SIMPLE (36 bytes), FULL (78 bytes), and COMMAND modes based on mission requirements.
- **Low-Latency Interface**: Utilizes WebSockets (Socket.IO) to stream data instantaneously to your local browser environment.

## 📋 Requirements

Ensure your system has Python 3 installed. You can install all necessary dependencies using the standard package manager:

```bash
pip install -r requirements.txt
```
*(Dependencies include: `flask`, `flask-cors`, `flask-socketio`, `pyserial`, and `eventlet`)*

## 🔧 Quick Setup

### 1. Start the Backend Server

Open your terminal and execute the backend application:

```bash
python main/gcs_backend.py
```

The server will initialize locally on `http://127.0.0.1:5000` and will automatically scan and attempt to connect to the active transmission port (defaulting to `/dev/ttyACM0`) at 19200 baud.

### 2. Launch the Dashboard

Navigate to the `main` directory and open `gcs_dashboard.html` in your web browser of choice. You will immediately have access to the Mission Control dashboard.

## 🔌 Port Configuration Details

### Receiving Port (Input from Rocket)
- **Baud Rate**: 19200
- **Packet Structure**: 78-byte fixed binary format
- **Headers**: `0xFF`, `0xFF`
- **Mandatory Data**: Altitude, Dual GPS coordinates, IMU data, and Parachute Deployment Status
- **Footers**: `0x0D`, `0x0A`

### Transmission Port (Output to Referee Station)
- **Target Port**: Typically `/dev/ttyACM0`
- **Baud Rate**: 19200
- **Modes**:
  - **SIMPLE**: 36 bytes (Condensed metrics)
  - **FULL**: 78 bytes (Complete authenticated telemetry packet)
  - **COMMAND**: Dedicated internal command and response protocol

## 🧪 Simulation and Testing

To verify the dashboard interface without an active rocket connection, we provide automated simulation tools within the `Tests` directory.

- **Simulate Incoming Arduino Telemetry**:
  ```bash
  python Tests/real_telemetry_simulator.py
  ```
  *(This generates synthetic flight data, allowing you to confirm that plotting, gauges, and status indicators function correctly.)*

- **Test the Transmission Relay Connection**:
  ```bash
  python Tests/test_tx_port.py
  ```

## 📡 API Reference

The backend software exposes several standard APIs to manage connections dynamically:

**Data Ingestion (RX)**
- `GET /api/ports` - Lists all available USB serial ports detected by the host
- `POST /api/connect` - Connects to the designated RX serial port
- `POST /api/disconnect` - Safely releases the current RX serial port connection

**Data Relay (TX)**
- `POST /api/tx/connect` - Establishes connection to the chosen TX port
- `POST /api/tx/disconnect` - Safely releases the TX port constraint
- `GET /api/tx/status` - Returns the health indicator of the active TX port
- `POST /api/tx/mode` - Alters the active physical transmission mode 

## 🔄 Data Architecture Flow

```text
Rocket (Sensors) → RX Radio Port → Python Backend Server → TX Relay Port → Referee Station
                                              ↓
                   Frontend HTML Dashboard (WebSockets for Real-Time Charts)
```

## 🛠️ Troubleshooting 

- **Port Not Found?** Use `ls /dev/tty*` (Linux/Mac) to confirm your operating system natively recognizes the hardware connection.
- **Permission Denied?** You may need to grant serial port permissions. Execute `sudo chmod 666 /dev/ttyACM0` or formally add your user to the standard `dialout` group (`sudo usermod -a -G dialout $USER`).
- **Dashboard Displays No Data?** Verify the connected hardware is transmitting at precisely 19200 baud and adheres strictly to the 78-byte structure.

For operators looking for a less technical initialization guide, please reference the `QUICK_START_GUIDE.md` stored in the root directory.
