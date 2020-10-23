One might wonder how all the different integrations and applications fit into to the LoRa ecosystem, I did too. Hence why I wrote some notes on how to set it up properly without all the headache.

### Hardware
Since we will be using the TinyLoRa library to communicate with TTN we need a compatible transceiver. As of writing that is the following Hope RF RFM95/96/97/98(W). In my case I'm using an Adafruit Feather M0 Radio which uses the RFM95. The antenna connection varies between boards so make sure you've got the right antenna and don't use the radio without an antenna as that will cause serious damage. 

### Configuring TTN
Once you have a TTN account you need to add an Application. Navigate to your TTN Console and press the add application button. You will be brought to the following page

![Add application](https://raw.githubusercontent.com/Lukas1337/Lukas1337.github.io/master/assets/images/add-application.png)