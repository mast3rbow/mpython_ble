This module is a driver library for BLE applications that provides higher-level abstractions based on MicroPython bluetooth . Provides a friendly interface for primary users of Bluetooth BLE, and users can easily implement BLE applications.

This module is suitable for MicroPython and related projects derived from it, such as the control board mPython .

Managed Documentation: https://mpython-ble.readthedocs.io/
github: https://github.com/labplus-cn/mpython_ble
According to the application and role played by Bluetooth BLE, the module designs the following major functions:

Peripheral
Central equipment (Centeral)
Serial port transparent transmission (Uart)
Human-computer interaction device HID
quick start
First, you need to upload the mpython_ble library to the MicroPython file system.

Before instantiating the BLE Peripheral device, configure the GATT Profile and register some services and characteristics. The profile is used to describe what kind of application services the device contains. The Bluetooth standard protocol defines common services and characteristics based on commonly used Bluetooth devices, so that BLE devices can be better identified and compatible. You can configure the profile according to the 16-bit UUID in GATT, or use the 128-bit UUID to define your own service.

Taking the thermometer as an example, we may need to add the following services to the profile:

Health Thermometer: The official 16-bit UUID identification code of the standard temperature service in the Bluetooth standard protocol is 0x1809; there is also a mandatory option Temperature Measurement feature (0x2A1C) that needs to be added.
Device Information : Used to describe device information. Add the Manufacturer Name String feature to describe the name of the manufacturer.
Instance the Health Thermometer Service object and add the Temperature Measurement Characteristic:

>>> from mpython_ble.services import Service
>>> from mpython_ble.characteristics import Characteristic
>>>
>>> health_thermometer_service = Service(UUID(0x1809))
>>> temperature_measurement_charact = Characteristic(UUID(0x2A1C), properties='-rn')
>>> health_thermometer_service.add_characteristics(temperature_measurement_charact)
Instance Device Information Service object and add Manufacturer Name String Characteristic:

>>> device_info_service = Service(UUID(0x180A))
>>> manufacturer_name = Characteristic(UUID(0x2A29),, properties='-r')
>>> device_info_service.add_characteristics(manufacturer_name)
After completing the health_thermometer_service and device_info_service services, add the service to the profile:

>>> from mpython_ble.gatts import Profile
>>> profile = Profile()
>>> profile.add_services(health_thermometer_service,device_info_service)
Next, we can easily read the services and characteristics of the profile through the array index method:

>>> profile
<Profile: <Service UUID16(0x1809): <Characteristic UUID16(0x2a1c), '-rn'>>, <Service UUID16(0x180a): <Characteristic UUID16(0x2a29), '-r'>>>
>>> profile[0] # first service
<Service UUID16(0x1809): <Characteristic UUID16(0x2a1c), '-rn'>>
>>> profile[1] # second service
<Service UUID16(0x180a): <Characteristic UUID16(0x2a29), '-r'>>
>>>
>>> profile[0][0] # Read temperature_measurement Characteristic
<Characteristic UUID16(0x2a1c), '-rn'>
Now let's see how to instantiate the Peripheral BLE peripheral through the configured profile.

profile.services_uuid is a list of UUIDs of services. The adv_services parameter in Peripheral is the service that needs to be broadcast.

>>> profile.services_uuid
[UUID16(0x1809), UUID16(0x180a)]
>>>
>>> from mpython_ble.application import Peripheral
>>> ble_thermometer = Peripheral(name=b'ble_thero', profile=profile, adv_services=profile.services_uuid)
BLE: activated!
After the BLE function is activated, start broadcasting:

>>> ble_thermometer.advertise(True)
You can use the nRF Connect app on your phone to connect to Bluetooth devices

Write a value to the Temperature Measurement Manufacturer Name String feature. Since the temperature value is a floating-point type, for ease of interpretation, we will *100 the temperature value. The Bluetooth protocol stipulates that the storage order of bytes is little endian. Convert int to Bytes, use struct to pack data.

Write the temperature value and notify the connected host:

>>> import struct
>>> temperature_value = 37.36
>>> ble_thermometer.attrubute_write(temperature_measurement_charact.value_handle, struct.pack("<H",int(temperature_value*100)),notify=True)
Write manufacturer's name:

>>> ble_thermometer.attrubute_write(manufacturer_name.value_handle, b'mpython')
At this time, you can read the written value on the host side.

./images/introduction_nrfconenct.png

References
Bluetooth protocol
HID OVER GATT
HID Usage Tables
contribute
Part of the source code of mpython_ble refers to the following projects, thanks to the author for his contribution:

Adafruit_CircuitPython_BLE
walkline_Micropython BLE
