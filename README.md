# Team VAJRA - GCS v.0.1
# TEKNOFEST MODEL ROCKETRY A4 CATEGORY 

 - Gaurav Shahi, Anshuman Pathak 

This is a fixed version of the `gcs_backend.py` that properly receives telemetry data from the backend, plots it in the frontend, and emits that same data to `/dev/ttyACM0` at 19200 baud rate.

## 🚀 Features

- **RX Port**: Receives 78-byte telemetry packets from Arduino/rocket
- **TX Port**: Automatically forwards received data to `/dev/ttyACM0` at 19200 baud
- **Real-time Plotting**: Live telemetry visualization in the frontend
- **Multiple TX Modes**: SIMPLE (36B), FULL (78B), and COMMAND modes
- **Web Interface**: Socket.IO real-time data transmission to frontend

## 📋 Requirements

```bash
pip install flask flask-cors flask-socketio pyserial eventlet
```

## 🔧 Setup

### 1. Start the Backend Server

```bash
python gcs_backend.py
```

The server will start on `http://127.0.0.1:5000` and automatically attempt to connect to `/dev/ttyACM0` at 19200 baud.

### 2. Open the Frontend

Open `gcs_dashboard.html` in your web browser to view the real-time telemetry data.

## 🔌 Port Configuration

### RX Port (Input from Arduino)
- **Default Baud**: 19200
- **Packet Format**: 78-byte binary packets
- **Header**: `FF FF`
- **Team ID**: 142
- **Data**: Altitude, GPS, IMU, Status
- **Footer**: `0D 0A`

### TX Port (Output to /dev/ttyACM0)
- **Port**: `/dev/ttyACM0`
- **Baud Rate**: 19200
- **Modes**:
  - **SIMPLE**: 36 bytes (Header + 8 floats + Checksum + Footer)
  - **FULL**: 78 bytes (Complete telemetry packet)
  - **COMMAND**: Command/Response protocol

## 🧪 Testing

### Test TX Port Connection

```bash
python test_tx_port.py
```

This will test the connection to `/dev/ttyACM0` and send sample packets.

### Simulate Arduino Data

```bash
python arduino_simulator.py [port] [duration]
```

Example:
```bash
python arduino_simulator.py /dev/ttyUSB0 30
```

This simulates an Arduino sending telemetry data for testing the RX functionality.

## 📡 API Endpoints

### RX Port Management
- `GET /api/ports` - List available USB serial ports
- `POST /api/connect` - Connect to RX serial port
- `POST /api/disconnect` - Disconnect from RX serial port

### TX Port Management
- `POST /api/tx/connect` - Connect to TX port
- `POST /api/tx/disconnect` - Disconnect from TX port
- `GET /api/tx/status` - Get TX port status
- `POST /api/tx/mode` - Set TX mode (SIMPLE/FULL/COMMAND)

### Testing
- `POST /api/test_telemetry` - Send test telemetry data
- `GET /api/data` - Get latest telemetry data
- `GET /api/config` - Get/set configuration

## 🔄 Data Flow

```
Arduino → RX Port → Backend → TX Port → /dev/ttyACM0
                ↓
            Frontend (Real-time plotting)
```

## 📊 Packet Structure

### RX Packet (78 bytes)
```
[FF FF] [TeamID] [Counter] [Altitude] [GPS Data] [IMU Data] [Angle] [Status] [Checksum] [0D 0A]
```

### TX Packet (SIMPLE - 36 bytes)
```
[AB] [8×Float] [Checksum] [0D 0A]
```

### TX Packet (FULL - 78 bytes)
```
[FF FF] [TeamID] [Counter] [Full Telemetry] [Checksum] [0D 0A]
```

## 🛠️ Troubleshooting

### Port Not Found
```bash
ls /dev/tty*
```
Check if your device is connected and identify the correct port.

### Permission Denied
```bash
sudo chmod 666 /dev/ttyACM0
```
Add your user to the dialout group:
```bash
sudo usermod -a -G dialout $USER
```

### No Telemetry Data
1. Check if Arduino is sending data in correct format
2. Verify baud rate matches (19200)
3. Use `arduino_simulator.py` to test RX functionality
4. Check backend logs for packet parsing errors

### TX Port Not Working
1. Verify `/dev/ttyACM0` exists and is accessible
2. Run `test_tx_port.py` to test connection
3. Check if another application is using the port
4. Verify baud rate is 19200

## 📈 Real-time Features

- **Live Charts**: Altitude, IMU, GPS tracking
- **Flight Status**: Phase detection, parachute status
- **Performance Metrics**: Deviation coefficient, reach coefficient
- **Communication Stats**: Packet rate, data rate, packet loss
- **Mission Time**: Real-time mission duration tracking

## 🔧 Configuration

The system can be configured via the `/api/config` endpoint:

```json
{
  "team_id": 142,
  "target_altitude": 8000,
  "category_max_altitude": 10000,
  "static_margin": 2.3
}
```

## 🎯 Usage Example

1. **Start the backend**:
   ```bash
   python gcs_backend.py
   ```

2. **Connect Arduino** to a USB port (e.g., `/dev/ttyUSB0`)

3. **Connect to RX port** via web interface or API:
   ```bash
   curl -X POST http://127.0.0.1:5000/api/connect \
     -H "Content-Type: application/json" \
     -d '{"port": "/dev/ttyUSB0", "baud": 19200}'
   ```

4. **Open frontend** (`gcs_dashboard.html`) to view real-time data

5. **Verify TX output** on `/dev/ttyACM0` at 19200 baud

## 🚨 Important Notes

- The TX port (`/dev/ttyACM0`) is automatically initialized on startup
- All received telemetry is automatically forwarded to the TX port
- The frontend expects 4-element arrays for sensor data (automatically generated)
- Packet loss and data rate are calculated automatically
- Mission time starts when first packet is received

## 🔍 Debug Information

Check the console output for detailed logging:
- `[SERIAL]` - Serial port operations
- `[PARSED]` - Packet parsing results
- `[TX]` - TX port transmission
- `[TELEMETRY]` - Key telemetry values
- `[ERROR]` - Error messages

The system is now ready to receive telemetry data from your Arduino and forward it to the specified TX port while providing real-time visualization in the frontend. 
