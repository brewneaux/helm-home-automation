# zigbee2mqtt Helm Chart

This chart deploys the [zigbee2mqtt application developed by koenkk](https://github.com/Koenkk/zigbee2mqtt) in Kubernetes.

## Hardware Setup

You must have your Zigbee stick aavailable in a Kubernetes node.  The node should be labeled with `zigbeeDeviceMounted: "true"` by doing 

```sh
kubectl label nodes worker2.k8s zigbeeDeviceMounted=true
```

This allows Kubernetes to schedule the pod only on the node that has the zigbee stick available.

Find the group that has access to that stick (usually dialout), and reference its group number by looking at `/etc/group` on the node.  You _must_ set the podSecuriyContext to have the runAsGroup as this number, and then runAsUser = 1 for root.

## A note on upgrades

Only one application can access a serial device at a time, therefore, the Deployment is set to the "Recreate" strategy.  This means it will trash the old pod before starting the upgrade.  Therefore, it is highly recommended to back up all of your settings before doing an upgrade.

# Configuration

The following table lists mosts of the configurations for the zigbee2mqtt chart.  Some custom options are available, specifically in the ingress, so please review the chart itself.


| Parameter | Description | Default |
| --------- | ----------- | ------- |
| zigbeeDevicePath | The path where the device is mounted | /dev/zigbee |
| logLevel | The log level for zigbee2mqtt | debug |
| permitJoin | Whether to allow devices to join the network. See https://www.zigbee2mqtt.io/guide/configuration/zigbee-network.html | false |
| serial.disable_led | disable LED of the adapter if supported (default: false) | false |
| serial.adapter | # Optional: adapter type, not needed unless you are experiencing problems (default: shown below, options: zstack, deconz, ezsp) | null |
| serial.baudRate | # Optional: Baud rate speed for serial port, this can be anything firmware support but default is 115200 for Z-Stack and EZSP, 38400 for Deconz, however note that some EZSP firmware need 57600. |  115200 |
| mqtt.host | Required: MQTT server URL (use mqtts:// for SSL/TLS connection) | mqtt://mqtt |
| mqtt.topic | zigbee2mqtt's base_topic option: "Optional: MQTT base topic for Zigbee2MQTT MQTT messages" | zigbee2mqtt |
| mqtt.user | MQTT authentication user | mosquitto |
| mqtt.password | MQTT authentication password | mosquitto |
| mqtt.clientId | MQTT client ID | zigbee2mqtt |
| additionalEnvironmentVariables | Any additional environment variables to pass into the zigbee2mqtt container - see Advanced Configuration below | {} |
| persistence.enabled | Enable persistant volumes - default of false uses an empty dir | false |
| image.repository | Image repository | koenkk/zigbee2mqtt |
| image.pullPolicy | Image pull policy | IfNotPresent |
| image.tag | Which image tag to use | latest |
| imagePullSecrets | Pull Secrets | [] |
| nameOverride | | "" |
| fullnameOverride | | "" |
| podAnnotations | | {} | 
| podSecurityContext.runAsGroup | Must be the group number that has access to the USB device, see above | 18 |
| podSecurityContext.runAsUser | Must be root | 1 |
| securityContext.privileged | Should be set to true to allow the pod to mount the device | true |
| ingress | See values file/chart | enabled: false |
| Resources | See values file | {} | 


## Advanced Configuration Options

This chart supports most of the common zigbee2mqtt options, but there are many more than can be set.  In order to do so, add the configuration you wish to change to the `additionalEnvironmentVariables` key in your values file, such as 

```yaml
additionalEnvironmentVariables:
  ZIGBEE2MQTT_SERIAL_RTSCTS: true
``

