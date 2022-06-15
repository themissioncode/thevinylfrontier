# thevinylfrontier
NFC Jukebox for Goonhilly 60


Intro
I have always loved a good jukebox, i found it would make or break a good pub here in the UK. Sadly theyve become quite a rarity now with the likes of streaming services such as spotify. The Vinyl Frontier is a 3rd in a series of projects we designed for particpants to engage with at special events at Goonhilly Earth Station in Cornwall. 


The Vinyl Frontier project uses tactile behaviour to bring back the joy of playing music with open source technology that you can build yourself. The use cases ive found for this would be great in a social environment such as party or pub where you pick up coasters printed with your favourite album/song/genre of music and place them on the box much more fun than sharing a screen or hassling the person controlling the music to put some music on.


The project uses a Raspberry PI micro computer and some easily accesible off the shelf components to read NFC tags (the same technology you find on tap and pay services in your phone) The tags trigger the raspberry pi to play audio files, web streams, podcasts, youtube playlists or your favourite Spotify streams and playlists. The raspberry PI uses some brilliant open source software called Phoniebox which is a web based application that runs on the PI.
Phoniebox operates on the PI and in the cloud as a webserver so you can access all the controls and settings via your laptop or phone once setup which ommits the need for a screen to be connected to the PI.


I have made this particular use case to be totally wireless by using a bluetooth module and onboard rechargeable LIPO battery - but using imagination a number of use cases can be applied. 

Parts
1 x Raspberry PI 4/ Should work with other models

1 x PN532 NFC Hat https://thepihut.com/products/nfc-hat-for-raspberry-pi-pn532

1 x LIPO Battery Hat - https://thepihut.com/products/lipo-battery-hat-for-raspberry-pi

1 x Bluetooth Headphone Adapter https://www.amazon.co.uk/EasyULT-Bluetooth-Transmitter-Receiver-Headphones/dp/B088K5WLX4/ref=sr_1_10

Large NFC Tags - Use 38mm tags for better range https://zipnfc.com/38mm-bullseye-smartrac-nfc-stickers-clear-wet-inlay-nxp-ntag213.html

Coasters - Any custom printer that prints coasters https://www.tradeprint.co.uk/personalised-coasters

Standoffs - these were used to raise the height of the NFC hat so it could recieve the NFC signal through the lid https://www.amazon.co.uk/Litorange-Standoff-Threaded-Motherboard-Assortment/dp/B07TLWZ4T7/ref=asc_df_B07TLWZ4T7/ 

Case
The parts i used above were a case of trial and error to fit into the wooden box that i used here https://www.amazon.co.uk/Transomnia-Photo-Straight-Edged-Fram007/dp/B008RA6SCO/ref=asc_df_B008RA6SCO/

There are many ways of delivering this project and i think getting creative with its presentation and housing is the fun thing. You could effectivly house this in all sorts of boxes, old Hi-Fi and even an old irreparible Jukebox that would be really cool!

Software

1. This project relys on the brilliant open source http://phoniebox.de/index-en.html. If you are looking for the latest stable version, use the install script for Raspberry Pi OS Stretch. There is a brilliant tutorial here https://github.com/MiczFlor/RPi-Jukebox-RFID/wiki/INSTALL-stretch
2. Ensure you follow the particular thread for installing the PN532 Hat. Yoiu can run a test on the install here https://www.waveshare.com/wiki/PN532_NFC_HAT#Raspberry_Pi_examples

