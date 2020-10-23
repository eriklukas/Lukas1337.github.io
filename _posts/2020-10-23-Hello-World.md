One might wonder how all the different integrations and applications fit into to the LoRa ecosystem, I did too. Hence why I wrote some notes on how to set it up properly without all the headache.

### Software
This may seem like a bunch of acronyms and if you don't know what all these do that's alright, we'll cover them invidually in greater depth later on. On our board we will run TinyLoRa, a library for interfacing with the onboard radio. The data is packaged using CayenneLPP. It has a lot of neat features such as being able to be decoded natively by TTN, many payload types that will look nice on the Cayenne MyDevices page if you want to use that. TTN is a global network made up of gateways that can receive our data and upload it to the internet. IFTTT is a service that integrate different IoT services so we can send data from TTN to Adafruit IO (AIO) for example. The free version is quite limited though so beware.

### Hardware
Since we will be using the TinyLoRa library to send data we need a compatible transceiver. As of writing that would be the following Hope RF RFM95/96/97/98(W). I'm using an Adafruit Feather M0 Radio which has the RFM95. For most boards you also need an antenna, the connection varies. In my case you need to cut wire to an appropriate length and solder it to the board. Beware that using the radio without an antenna connected will seriously damage the board.

### Configuring TTN
Once you have a TTN account you need to add an Application. Navigate to your TTN Console and press the add application button.You will be brought to the following page. 

![Add application](https://raw.githubusercontent.com/Lukas1337/Lukas1337.github.io/master/assets/images/add-application.png)
Choose an appropriate name and description, no need to worry about the App EUI, and select the appropriate region for the handler. When done click add application in the bottom right.

Once an applications has been added you need to register the devices you want to be using with the application. In your application navigate to the Devices tab, press the register device button. You will be brought to the page below

![Register device](https://raw.githubusercontent.com/Lukas1337/Lukas1337.github.io/master/assets/images/register-device.png)
Choose an appropriate ID for the device and make sure Device EUI and App Key are set to "This field will be generated", the App EUI should match the one you saw when the application was created. Once registered you should be able to navigate to the device and see a screen similar to this. 

![Device registered](https://raw.githubusercontent.com/Lukas1337/Lukas1337.github.io/master/assets/images/device-registered.png)
Once registered we need to change some settings, so in the top right click settings. Change the activation method to ABP, then have the Network Session Key and the App Sessions Key be generated. For debugging it might be wise to disable Frame Counter Checks although remember to enable it again when deploying your application. Your settings should be similar to these.

![Change device settings](https://raw.githubusercontent.com/Lukas1337/Lukas1337.github.io/master/assets/images/change-device-settings.png)
Save your settings and you are done!

### Code
```CPP

// TinyLoRa with ABP to send Feather M0 Radio battery voltage 
// initially derived from https://github.com/adafruit/TinyLoRa/blob/master/examples/hello_LoRa/hello_LoRa.ino
// additional references https://www.thethingsnetwork.org/docs/devices/arduino/api/cayennelpp.html

#include <TinyLoRa.h>
#include <SPI.h>
#include <CayenneLPP.h>

// Network Session Key (MSB)
uint8_t NwkSkey[16] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };

// Application Session Key (MSB)
uint8_t AppSkey[16] = { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };

// Device Address (MSB)
uint8_t DevAddr[4] = { 0x00, 0x00, 0x00, 0x00 };

const unsigned int sendInterval = 30; // How often we will transmit in seconds

const int batteryPin = A7; // The pin for reading the battery voltage on the Feather M0 Radio
float measuredVoltage = 0; // Used for saving the battery voltage reading

// Pinout for Feather 32u4 LoRa
//TinyLoRa lora = TinyLoRa(7, 8, 4);

// Pinout for Feather M0 LoRa
TinyLoRa lora = TinyLoRa(3, 8, 4); // SPI pins used for the radio
CayenneLPP lpp(51); // CayenneLPP packet with max size 51 bytes

void setup()
{
    Serial.begin(9600);
    /* Uncomment when debugging
    while(!Serial) { // Waits for the serial port, necessary for usb native devices
      ;
    }
    /**/
    pinMode(LED_BUILTIN, OUTPUT); // Declare the built in led as an OUTPUT
    Serial.print("Starting LoRa...");
    lora.setChannel(MULTI); // Define channel
    lora.setDatarate(SF7BW125); // Set datarate https://unsigned.io/understanding-lora-parameters/
    if(!lora.begin()) { // Wait for LoRa ready
        Serial.println("Failed");
        Serial.println("Check your radio");
    while(true);
    }
    Serial.println("OK"); // When LoRa is ready
}

void loop() {
    Serial.println("Sending LoRa Data...");
    measuredVoltage = analogRead(batteryPin);
    measuredVoltage *= 2.0; // multiply by 2 because of the voltage divider
    measuredVoltage *= 3.3; // multiply by 3.3V, our reference voltage
    measuredVoltage /= 1024.0; // convert adc measurement to voltage
    lpp.reset(); // Resets the buffer
    lpp.addAnalogInput(2, measuredVoltage); // Adds an analogInput to the packet
    lora.sendData(lpp.getBuffer(), lpp.getSize(), lora.frameCounter); // Sends the data
    Serial.print("Frame count: ");
    Serial.println(lora.frameCounter); // Prints the framcount for debugging
    lora.frameCounter++; // Adds 1 to the frame counter
    // Blink LED to indicate packet sent
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);
    digitalWrite(LED_BUILTIN, LOW);
    Serial.println("Waiting before sending the next packet...");
    delay(sendInterval * 1000); // to avoid sending too often we add a delay at the bottom of the loop
}
```
Replace Network Session Key, App Session Key and Device adress with the values on TTN. Making sure you have the MSB format selected in TTN and that the values are expanded. As in the screenshot below

![Copy device id](https://raw.githubusercontent.com/Lukas1337/Lukas1337.github.io/master/assets/images/copy-device-id.png)

Upload the code to your board and hopefully, fingers crossed you'll see data coming in!