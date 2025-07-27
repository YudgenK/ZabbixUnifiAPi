# Zabbix Template for UniFi Controller Monitoring

This Zabbix template enables monitoring of UniFi Controller devices via the UniFi API using an API key. It discovers UniFi devices (e.g., access points) and collects metrics such as status, uptime, IP address, firmware version, model, and number of connected clients.

## Features

- **Device Discovery**: Automatically discovers UniFi devices using the UniFi Controller API (`/proxy/network/api/s/default/stat/device`).
- **Monitored Metrics**:
  - **Status**: Checks if the device is online (`state = 1`) or offline.
  - **Uptime**: Tracks device uptime in seconds.
  - **IP Address**: Monitors the device's current IP address.
  - **Firmware Version**: Compares the device's firmware version with the latest available version.
  - **Model**: Retrieves the device model (e.g., `U7MSH`).
  - **Connected Clients**: Monitors the number of clients connected to each device (`num_sta`).
- **Triggers**:
  - Alerts if a device is offline (`state <> 1`).
  - Alerts if a device's firmware version is outdated.
- **Configurable Intervals**: Device data is polled every 5 minutes by default.
- **API Key Authentication**: Uses a secure API key for accessing the UniFi Controller API.

## Requirements

- **Zabbix Version**: 7.2 or later.
- **UniFi Controller**: Version supporting the `/proxy/network/api/s/default/stat/device` API endpoint and API key authentication (e.g., UniFi Controller 7.x or later).
- **Network Access**: Zabbix server must have network access to the UniFi Controller
- **API Key**: A valid UniFi API key generated in the UniFi Controller (Settings → System → API).

## Installation

1. **Download the Template**:
   - Clone this repository or download the `zbx_export_unifi.yaml` file:
     ```bash
     git clone <repository_url>
     ```
   - Alternatively, copy the template YAML file from this repository.

2. **Import the Template into Zabbix**:
   - In the Zabbix web interface, navigate to **Configuration → Templates**.
   - Click **Import** and select the `zbx_export_unifi.yaml` file.
   - Ensure the import is successful and the template appears under **Templates ZBX** group.

3. **Assign the Template to a Host**:
   - Go to **Configuration → Hosts** and select or create a host for the UniFi Controller.
   - Set the host's IP address in the **Interfaces** section (e.g., `192.168.2.103`).
   - Link the **Template UniFi Controller** to the host.

4. **Configure Macros**:
   - Navigate to **Configuration → Hosts → [Your Host] → Macros**.
   - Add or update the following macros:
     - `{$UNIFI_API_KEY}`: Set to your UniFi Controller API key (e.g., `your_api_key_here`).
     - `{$UNIFI_UPDATEURL}`: Keep the default value (`https://fw-update.ubnt.com/api/firmware-latest?filter=eq~~product~~unifi-controller&filter=eq~~channel~~release&filter=eq~~platform~~windows`) or update it if needed for your platform.

## Configuration

- **Polling Interval**: The template polls device data every 5 minutes (`delay: 5m` in the `unifi.devices` item). To change the interval:
  - Go to **Configuration → Templates → Template UniFi Controller → Items**.
  - Find the **Обнаружение устройств UniFi** item and update the **Update interval** (e.g., `1m` for 1 minute).
- **Timeout**: If you experience timeouts, increase the timeout for the `unifi.devices` item:
  - Set the **Timeout** to 30–60 seconds in the item settings.

## Usage

1. **Verify Data Collection**:
   - Go to **Monitoring → Latest Data** and select your host.
   - Check the **Обнаружение устройств UniFi** item (`unifi.devices`) to ensure it returns a JSON response with device data.
   - Verify that dependent items (e.g., `unifi.devices.status.[{#MAC}]`, `unifi.devices.clients.[{#MAC}]`) are populated.

2. **Check Discovered Devices**:
   - Navigate to **Configuration → Hosts → [Your Host] → Discovery → Обнаружение устройств UniFi**.
   - Confirm that devices are discovered with macros `{#MAC}`, `{#NAME}`, and `{#MODEL}`.

3. **Monitor Triggers**:
   - Go to **Monitoring → Problems** to view alerts for offline devices or outdated firmware.
   - Example triggers:
     - **Устройство UniFi [{#NAME}] не активно на хосте {HOSTNAME}**: Fires if a device is offline (`state <> 1`).
     - **Ожидается обновление прошивки устройства UniFi [{#NAME}] на хосте {HOSTNAME}**: Fires if the device's firmware version differs from the latest available version.

## Troubleshooting

- **No Data in `unifi.devices`**:
  - Check Zabbix server logs for errors:
    ```bash
    tail -f /var/log/zabbix/zabbix_server.log
    ```
    Look for `Zabbix.log` messages with HTTP status codes or JSON parsing errors.
  - Verify the API key and endpoint:
    ```bash
    curl -k -X GET 'https://192.168.2.103/proxy/network/api/s/default/stat/device' \
    -H 'X-API-KEY: your_api_key' \
    -H 'Accept: application/json'
    ```
    Ensure the response contains a `data` array with device information.

- **HTTP 401/403 Errors**:
  - Confirm the API key is valid (check in UniFi Controller: Settings → System → API).
  - Ensure the Zabbix server's IP is not blocked by the UniFi Controller's firewall.

- **No Discovered Devices**:
  - Verify that the `unifi.devices.discovery` rule is processing data correctly.
  - Check the preprocessing script in the discovery rule to ensure it generates `{#MAC}`, `{#NAME}`, and `{#MODEL}` macros.

- **Firmware Version Check Fails**:
  - Test the `{$UNIFI_UPDATEURL}`:
    ```bash
    curl -k https://fw-update.ubnt.com/api/firmware-latest?filter=eq~~product~~unifi-controller&filter=eq~~channel~~release&filter=eq~~platform~~windows
    ```
  - If the URL is outdated, update the `{$UNIFI_UPDATEURL}` macro with the correct endpoint.

## Contributing

Contributions are welcome! Please submit a pull request with improvements or additional metrics (e.g., monitoring WiFi channels, traffic rates). Ensure any changes are tested with a UniFi Controller and Zabbix 7.2+.
