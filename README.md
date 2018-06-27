# Ubidots

## Introduction
The Ubidots Plugin allows CraftBeerPi 3.0 to send sensor data to Ubidots.com  

## Add a Device on Ubidots.com
1. Login to your Ubidots.com account
2. Write down you "Default token", found under Your Name - API Credentials
3. Add a New Device and write down your "API Label"

## Use
1. Download this Plugin from CraftBeerPi 3 Add-On page
2. Under Parameters add your API Token (ubidots_token) and Device Label (ubidots_label)
4. Restart CraftBeerPi

The first eight sensors will be updated to the channel every minute. 
On Ubidots your Ubidots label name will be updated with your brewery name, and fields will get the sensor names along with actors and target temperature.
