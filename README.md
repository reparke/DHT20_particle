

# DHT20

DHT20 I2C library for Particle devices


## Description

This is a DHT20 I2C library for Particle devices (e.g. Photon, Argon, Boron) based on the [Arduino library](https://github.com/RobTillaart/DHT20) created by [Rob Tillaart](https://github.com/RobTillaart)

The library must be initiated by calling the **begin()** function, 
or **begin(dataPin, clockPin)** for **ESP32** and similar platforms.

Thereafter one has to call the **read()** function to do the actual reading,
and with **getTemperature()** and **getHumidity()** to get the measured values.
Calling these latter again will return the same values until a new **read()** is called.

The **read()** call of this sensor is blocking for 80+ milliseconds (datasheet 7.4)
so the library also has a asynchronous interface. See below.

Since 0.1.3 and 0.1.4 the performance of **read()** has been optimized, 
still blocking but less long for about 45 milliseconds.


### Connection

Always check datasheet 

Front view
```
          +--------------+
  VDD ----| 1            |
  SDA ----| 2    DHT20   |
  GND ----| 3            |
  SCL ----| 4            |
          +--------------+
```

### Tested

Verified to work with Particle Argon



## Interface


### Constructor

- **DHT20(TwoWire \*wire = &Wire)** constructor, using a specific Wire (I2C bus).
- **bool begin(uint8_t dataPin, uint8_t clockPin)** begin for ESP32 et al, to set I2C bus pins.
- **bool begin()** initializer for non ESP32. Returns true if connected.
- **bool isConnected()** returns true if the address of the DHT20 can be seen on the I2C bus.


### Core

- **int8_t read()** read the sensor and store the values internally. 
It returns the status of the read which should be 0.
- **float getHumidity()** returns last Humidity read.
- **float getTemperature()** returns last Temperature read.


### Offset

- **void setHumOffset(float offset)** set an offset to calibrate (1st order) the sensor.
- **float getHumOffset()** return current offset, default 0.
- **void setTempOffset(float offset)** set an offset to calibrate (1st order) the sensor.
- **float getTempOffset()** return current offset, default 0.


### Asynchronous interface

There are two timings that need to be considdered, 
- time between requests = 1000 ms
- time between request and data ready = 80 ms

The async interface allows one to continue processing after a **requestData()** has been made. 
Note that there should be at least **1000 milliseconds** between subsequent requests.

With **bool isMeasuring()** one can check if a new measurement is ready.
If so the sensor can be read with **readData()**.

To interpret the read bits to temperature, humidity and status one needs to call **convert()** as last step.


- **int requestData()** signals the sensor to make a new measurement.
Note there must be at least 1000 milliseconds between requests!
- **int readData()** does the actual reading of the data.
- **int convert()** converts the read bits to temperature and humidity.

See the example **DHT20_async.ino**


### Status

- **uint8_t readStatus()** forced read of the status only.
This function blocks a few milliseconds to optimize communication.
- **bool isCalibrated()** idem, wrapper around **readStatus()**
- **bool isMeasuring()** idem, wrapper around **readStatus()**
- **bool isIdle()** idem, wrapper around **readStatus()**
- **int internalStatus()** returns the internal status of the sensor. (for debug).

|  status bit  |  meaning                   |
|:------------:|:---------------------------|
|    7         |  1 = measurement, 0 = idle |
|  6 - 4       |  unknown                   |
|    3         |  1 = calibrated, 0 = not   |
|  2 - 0       |  unknown                   |


#### Experimental 0.1.4 resetSensor

Use with care, as this is not tested.

- **uint8_t resetSensor()** if at startup the sensor does not return a status of 0x18, 
three registers 0x1B, 0x1C and 0x1E need to be reset. 
See datasheet 7.4 Sensor Reading Process, point 1.
There is no documentation about the meaning of these registers.
The code is based upon example code for the AHT20 (from manufacturer).


### Timing

- **uint32_t lastRead()** last time the sensor is read in milliseconds since start.
- **uint32_t lastRequest()** last time a request is made to make a measurement.


### Return codes

| name                        |  value  |  notes  |
|:----------------------------|:-------:|:--------|
| DHT20_OK                    |    00   |  OK
| DHT20_ERROR_CHECKSUM        |   -10   |  values might be OK if they are like recent previous ones.
| DHT20_ERROR_CONNECT         |   -11   |  check connection
| DHT20_MISSING_BYTES         |   -12   |  check connection
| DHT20_ERROR_BYTES_ALL_ZERO  |   -13   |  check connection
| DHT20_ERROR_READ_TIMEOUT    |   -14   |
| DHT20_ERROR_LASTREAD        |   -15   |  wait 1 second between reads


## Operation

See examples


## Future

#### must


#### should


#### could

- improve unit tests.
- investigate 
  - sensor calibration (website aosong?)
- investigate optimizing timing in readStatus()
  - delay(1) ==> microSeconds(???).
- separate changelog.md
- connected flag?

#### won't

- **void setIgnoreChecksum(bool = false)** ignore checksum flag speeds up communication a bit
- **bool getIgnoreChecksum()** get checksum flag. for completeness.
- 

## Contributing

Here's how you can make changes to this library and eventually contribute those changes back.

To get started, [clone the library from GitHub to your local machine](https://help.github.com/articles/cloning-a-repository/).

Change the name of the library in `library.properties` to something different. You can add your name at then end.

Modify the sources in <src> and <examples> with the new behavior.

To compile an example, use `particle compile examples/usage` command in [Particle CLI](https://docs.particle.io/guide/tools-and-features/cli#update-your-device-remotely) or use our [Desktop IDE](https://docs.particle.io/guide/tools-and-features/dev/#compiling-code).

After your changes are done you can upload them with `particle library upload` or `Upload` command in the IDE. This will create a private (only visible by you) library that you can use in other projects. Do `particle library add DHT20_I2C_Particle_myname` to add the library to a project on your machine or add the DHT20_I2C_Particle_myname library to a project on the Web IDE or Desktop IDE.

At this point, you can create a [GitHub pull request](https://help.github.com/articles/about-pull-requests/) with your changes to the original library. 

If you wish to make your library public, use `particle library publish` or `Publish` command.

## Maintainer

Rob Parke
