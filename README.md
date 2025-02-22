# MIDI HOST DRIVER
This README file contains the design notes and limitations of the
MIDI host driver, and it describes how to build an run a simple
example to test it. The driver should run on any TinyUSB supported
processor with USB Host Bulk endpoint support, but the driver and
example code have only been tested on a RP2040 in a Raspberry Pi
Pico board.

# Arduino Support
This library was originally written for RP2040 support only.
I was able to get this library to work on a Raspberry Pi Pico
board using the the Arduino IDE by copying it to my
local `libraries` directory. See the [EXAMPLES/Arduino](#arduino) section below
for more info. This was my first attempt at an Arduino project, so
I suspect there are better ways to do things. Issue reports and pull requests
are welcome. At the time of this writing, there is a known issue with
displaying Arduino Serial messages at startup. I put in some delays
that work on one of my computers, but not others. Surely, there is a better
way.

# ACKNOWLEDGEMENTS
This driver code is based on code that rppicomidi submitted to TinyUSB
as pull request #1219. The pull request was never merged and got stale.
TinyUSB pull request #1627 by atoktoto started with pull request #1219,
but used simpler RP2040 bulk endpoint support from TinyUSB pull request
#1434. Pull request #1627 was reviewed by todbot, AndrewCapon, PaulHamsh,
and rppicomidi and was substantially functional. This driver copied
the `midi_host.c/h` files from pull request #1627 and renamed them
`usb_midi_host.c/h`. It also fixed some minor issues in the driver
API and added the application driver wrapper `usb_midi_host_app_driver.c`.
The example code code is adapted from TinyUSB pull request #1627. 

# OVERALL DESIGN, INTEGRATION
Although it is possible in the future for the TinyUSB stack to
incorporate this driver into its builtin driver support, right now
it is used as an application driver external to the stack. Installing
the driver to the TinyUSB stack requires adding it to the array
of application drivers returned by the `usbh_app_driver_get_cb()`
function. The `CMakeLists.txt` file contains two `INTERFACE` libraries.
If this driver is your only application USB host driver external
to the TinyUSB stack, you should install this driver by
adding the `usb_midi_host_app_driver` library to your main
application's `CMakeLists.txt` file's `target_link_libraries`.
If you want to add multiple host drivers, you must implement
your own `usbh_app_driver_get_cb()` function and you should
add the `usb_midi_host` library to your main application's
`CMakeLists.txt` file's `target_link_libraries` instead.

# EXAMPLE PROGRAMS

## C-Code
Before you try to use this driver with TinyUSB, please make
sure the `usbh_app_driver_get_cb()` is supported in your
version of TinyUSB. This feature was introduced on 15-Aug-2023
with commit 7537985c080e439f6f97a021ce49f5ef48979c78. If you
are using the `pico-sdk` for the Raspberry Pi Pico, you
should be able to ensure you have the latest TinyUSB code
by pulling the latest master branch from the TinyUSB repo.

```
cd ${PICO_SDK_PATH}/lib/tinyusb
git checkout master
git pull
```

The `examples/C-code` directory in this project contains a simple example
that plays a 5 note sequence on MIDI cable 0 and prints out
every MIDI message it receives. It is designed to run on a
Raspberry Pi Pico board. You will need a USB to UART adapter
to see the serial interface display. You will need a 5V
power source to provide power to the Pico board and the
connected USB MIDI device. I used a Pico board wired as a
Picoprobe to provide both the USB to UART adapter and the
5V power source. Finally, you will need a microUSB to USB A
adapter to allow you to connect your USB MIDI device to the
test system.
```
cd examples\C-code
mkdir build
```
To build
```
cd build
cmake ..
make
```
To test, copy the UF2 file to the Pico board using whatever
method you prefer. Connect a microUSB to USB A adapter to
the Pico board USB connector. Connect the USB to UART adapter
to the Pico board pins 1 and 2. Connect the Ground pin of the
USB to UART adapter boardto a Ground pin of the Pico Board (I use
the Pico board debug port Ground pin). If it is different from
the ground pin of the USB to UART adapter, connect the Ground
pin of the 5V power source to a ground pin of the Pico board.
Connect the 5V pin of the 5V power source to pin 40 of the Pico
board. Make sure the board powers up and displays the message
```
Pico MIDI Host Example
```
before you attach your USB MIDI device.

Attach your USB MIDI device to the microUSB to USB A adapter
with the appropriate cable. You should see a message similar
to
```
MIDI device address = 1, IN endpoint 2 has 1 cables, OUT endpoint 1 has 1 cables
```
If your MIDI device can generate sound, you should start hearing a pattern
of notes from B-flat to D. If your device is Mackie Control compatible, then
the transport LEDs should sequence. If you use a control on your MIDI device, you
should see the message traffic displayed on the serial console.

## Arduino
The sketch file `examples\arduino\usb_midi_host_example.ino`
implements about the same functionality as the `midi_host_example.c` code
implements in C code. However it uses a soft USB Host implemented in the
RP2040 PIOs instead of the native hardware. If you are using a Raspberry
Pi Pico or Pico W board, you must wire a USB A jack to the host port
as follows:
```
Pico/Pico W board pin   USB A connector pin
23 (or any GND pin)  ->     GND
21 (GP2)             ->     D+  (via a 22 ohm resistor should improve things)
22 (GP3)             ->     D-  (via a 22 ohm resistor should improve things)
40 (VBus)            ->     VBus (safer if it has current limiting on the pin)
```
Adafruit makes a [RP2040 board](https://www.adafruit.com/product/5723) that
wires a USB A connector that should just work with the RP2040 pins GPIO pins
this example selects. However, I have not tested it. I do not know if or
how well it works. This Arduino example was tested using a Raspberry Pi
Pico board with a USB A breakout board wired as above.

If you are wiring your own USB A port, be extra careful; you don't want to
damage the attached MIDI gear because you swapped power and ground.
If you are using hardware that attaches the USB A connector for you, please
read the documentation for your RP2040 board. You may have to edit the
sketch so the code maps the Host port D+ and D- pins correctly.

To get the MIDI host example to work, first set up your Arduino IDE
per the instructions [here](https://learn.adafruit.com/adafruit-feather-rp2040-with-usb-type-a-host/arduino-ide-setup).
Make sure you can get the Device Info Example to work before on
your hardware before you attempt to try the Arduino example for this project.
That ensures your Arduino IDE and your hardware is working and you know
how to use it.

Next, copy this library to your `libraries` directory. For example, if your
Sketchbook directory is in `${HOME}/Documents/Arduino/`,
```
cd ${HOME}/Documents/Arduino/libraries
git clone https://github.com/rppicomidi/usb_midi_host
```

Next, copy the example program to your Sketchbook directory.
For example, if your Sketchbook directory is in `${HOME}/Documents/Arduino/`,
```
cd ${HOME}/Documents/Arduino
mkdir usb_midi_host_example
cp libraries/usb_midi_host/examples/arduino/usb_midi_host_example.ino usb_midi_host_example
```
Restart the Arduino IDE, and then open and run the sketch. Note that
you must set the Board speed to either 120MHz or 240MHz, and you must
choose the Adafruit_TinyUSB library. Attach a MIDI device to the USB A port.
You should see something like this in the Serial Port Monitor (of course,
your connected MIDI device will likely be different).
```
MIDI device address = 1, IN endpoint 1 has 1 cables, OUT endpoint 2 has 1 cables
Device attached, address = 1
  iManufacturer       1     KORG INC.
  iProduct            2     nanoKONTROL2
  iSerialNumber       0

```
If your MIDI device can generate sound, you should start hearing a pattern
of notes from B-flat to D. If your device is Mackie Control compatible, then
the transport LEDs should sequence. If you use a control on your MIDI device, you
should see the message traffic displayed on the Serial Port Monitor.

# MAXIMUM NUMBER OF MIDI DEVICES ATTACHED TO HOST
You should define the value `CFG_TUH_DEVICE_MAX` in tusb_config.h to
match the number of USB hub ports attached to the USB host port. For
example
```
// max device support (excluding hub device)
#define CFG_TUH_DEVICE_MAX          (CFG_TUH_HUB ? 4 : 1) // hub typically has 4 ports
```

# MAXIMUM NUMBER OF ENDPOINTS
Although the USB MIDI 1.0 Class specification allows an arbitrary number
of endpoints, this driver supports at most one USB BULK DATA IN endpoint
and one USB BULK DATA OUT endpoint. Each endpoint can support up to 16 
virtual cables. If a device has multiple IN endpoints or multiple OUT
endpoints, it will fail to enumerate.

Most USB MIDI devices contain both an IN endpoint and an OUT endpoint,
but not all do. For example, some USB pedals only support an OUT endpoint.
This driver allows that.

# MAXIMUM NUMBER OF VIRTUAL CABLES
A USB MIDI 1.0 Class message can support up to 16 virtual cables. The function
`tuh_midi_stream_write()` uses 6 bytes of data stored in an array in
an internal data structure to deserialize a MIDI byte stream to a
particular virtual cable. To properly handle all 16 possible virtual cables,
`CFG_TUH_DEVICE_MAX*16*6` data bytes are required. If the application
needs to save memory, in file `tusb_cfg.h` set `CFG_TUH_CABLE_MAX` to
something less than 16 as long as it is at least 1.

# PUBLIC API
Documentation for all public API functions are in the file `usb_midi_host.h`.
In addition to calling functions in the API, your program, whether it is written
as C code, C++ code, or an Arduino sketch, must, at a minimum, implement the
callback functions to handle asynchronous data from the attached MIDI device:
```
tuh_midi_mount_cb(uint8_t dev_addr, uint8_t in_ep, uint8_t out_ep, uint8_t num_cables_rx, uint16_t num_cables_tx)
tuh_midi_umount_cb(uint8_t dev_addr, uint8_t instance);
tuh_midi_rx_cb(uint8_t dev_addr, uint32_t num_packets);
```
See the programs in the `Examples` folder for examples of how to do this.

Applications interact with this driver via 8-bit buffers of MIDI messages
formed using the rules for sending bytes on a 5-pin DIN cable per the
original MIDI 1.0 specification.

To send a message to a device, the Host application composes a sequence
of status and data bytes in a byte array and calls the API function.
The arguments of the function are a pointer to the byte array, the number
of bytes in the array, and the target virtual cable number 0-15.

When the host driver receives a message from the device, the host driver
will call a callback function that the host application registers. This
callback function contains a pointer to a message buffer, a message length,
and the virtual cable number of the message buffer. One complete bulk IN
endpoint transfer might contain multiple messages targeted to different
virtual cables.

If you prefer, you may read and write raw 4-byte USB MIDI 1.0 packets.

# SUBCLASS AUDIO CONTROL
A MIDI device is supposed to have an Audio Control Interface, before
the MIDI Streaming Interface, but many commercial devices do not have one.
To support these devices, the descriptor parser in this driver will skip
past any audio control interface and audio streaming interface and open
only the MIDI interface.

An audio streaming host driver can use this driver by passing a pointer
to the MIDI interface descriptor that is found after the audio streaming
interface to the midih_open() function. That is, an audio streaming host
driver would parse the audio control interface descriptor and then the
audio streaming interface and endpoint descriptors. When the next descriptor
pointer points to a MIDI interface descriptor, call midih_open() with that
descriptor pointer.

# CLASS SPECIFIC INTERFACE AND REQUESTS
The host driver only makes use of the informaton in the class specific
interface descriptors to extract string descriptors from each IN JACK and
OUT JACK. To use these, you must set `CFG_MIDI_HOST_DEVSTRINGS` to 1 in
your application's tusb_config.h file. It does not parse ELEMENT items
for string descriptors.

This driver does not support class specific requests to control
ELEMENT items, nor does it support non-MIDI Streaming bulk endpoints.

# MIDI CLASS SPECIFIC DESCRIPTOR TOTAL LENGTH FIELD IGNORED
I have observed at least one keyboard by a leading manufacturer that
sets the wTotalLength field of the Class-Specific MS Interface Header
Descriptor to include the length of the MIDIStreaming Endpoint
Descriptors. This is wrong per my reading of the specification.

# MESSAGE BUFFER DETAILS
Messages buffers composed from USB data received on the IN endpoint will never
contain running status because USB MIDI 1.0 class does not support that. Messages
buffers to be sent to the device on the OUT endpont can contain running status
(the message might come from a UART data stream from a 5-pin DIN MIDI IN
cable on the host, for example), and thanks to pull request#3 from @moseltronics,
this driver should correctly parse or
compose 4-byte USB MIDI Class packets from streams encoded with running status.

Message buffers to be sent to the device may contain real time messages
such as MIDI clock. Real time messages may be inserted in the message 
byte stream between status and data bytes of another message without disrupting
the running status. However, because MIDI 1.0 class messages are sent 
as four byte packets, a real-time message so inserted will be re-ordered
to be sent to the device in a new 4-byte packet immediately before the
interrupted data stream.

Real time messages the device sends to the host can only appear between
the status byte and data bytes of the message in System Exclusive messages
that are longer than 3 bytes.

# POORLY FORMED USB MIDI DATA PACKETS FROM THE DEVICE
Some devices do not properly encode the code index number (CIN) for the
MIDI message status byte even though the 3-byte data payload correctly encodes
the MIDI message. This driver looks to the byte after the CIN byte to decide
how many bytes to place in the message buffer.

Some devices do not properly encode the virtual cable number. If the virtual
cable number in the CIN data byte of the packet is not less than bNumEmbMIDIJack
for that endpoint, then the host driver assumes virtual cable 0 and does not
report an error.

Some MIDI devices will always send back exactly wMaxPacketSize bytes on
every endpoint even if only one 4-byte packet is required (e.g., NOTE ON).
These devices send packets with 4 packet bytes 0. This driver ignores all
zero packets without reporting an error.

# ENUMERATION FAILURES
The host may fail to enumerate a device if it has too many endpoints, if it has
if it has a Standard MS Transfer Bulk Data Endpoint Descriptor (not supported),
if it has a poorly formed descriptor, or if the descriptor is too long for
the host to read the whole thing.

