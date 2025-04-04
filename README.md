# forked
this is a forked version of txtnet that allows very barebones JS support. Only things like onClick attribute, redirect code, and alert() will be supported
don't expect things to work 100% of the time
 
 # TxtNet Browser
### Browse the Web over SMS, no WiFi or Mobile Data required!
<p align="center"><img src="https://github.com/lukeaschenbrenner/TxtNet-Browser/raw/master/app/src/main/ic_launcher-playstore.png" alt="App Icon" width="200"/></p>  

> Please see [this discussion post](https://github.com/lukeaschenbrenner/TxtNet-Browser/discussions/40) for updates on the status of this app's development.

TextNet Browser is an Android app that allows anyone around the world to browse the web without a mobile data connection! It uses SMS as a medium of transmitting HTTP requests to a server where a pre-parsed HTML response is compressed using Google's [Brotli](https://github.com/google/brotli) compression algorithm and encoded using a custom Base-114 encoding format (based on [Basest](https://github.com/saxbophone/basest-python)). All media content, JavaScript, and CSS are removed from the page, but websites that require JavaScript will still render and display. Only HTTP GET requests are implemented.

The app also contains TxtNet Server, a background service that enables your own phone, using its primary phone number and a Wi-Fi/data connection, to process requests by any other phones that don't have an internet connection. This means that if you have a phone that has internet access and SMS, you can host your own server without any technical knowledge at the press of a button! See "Server Hosting" below for setup instructions.

## Download
### See the **[releases page](https://github.com/lukeaschenbrenner/TxtNet-Browser/releases)** for an APK download of the TxtNet Browser client.
TxtNet Browser is currently compatible with **Android 4.4 (KitKat) and up** (yes, your old phone can still install it!)
A Google Play and F-Droid release is planned, but with low priority.

## Public Server Instances
| Country      | Phone Number | Notes     |
| :---        |    :----:   | :---        |
| United States      | +1(913)203-2719 | Uptime is low. Supports SMS to all +1 (US/Canada) numbers in addition to [these countries](https://github.com/lukeaschenbrenner/TxtNet-Browser/issues/2#issuecomment-1510506701). May be phased out by late 2025.   |
|    |      |     |

Let me know if you are interested in hosting a server instance for your area!

> ⚠️**Please note**: All web traffic  should be considered unencrypted, as all requests are made over SMS and received in plaintext by the server!

## How it works (client)

This app uses a permission that allows a broadcast reciever to recieve and parse incoming SMS messages without the need for the app to be registered as the user's default messaging app. While granting an app SMS permissions poses a security concern, the code for this app is open source and all code involving the use of internet permissions are compartamentalized to the server module. This ensures that unless the app is setup to be a server, no internet traffic is transmitted. In addition, as the client, SMS messages are only programatically sent to and recieved from a registered server phone number.
The app communicates with a "server phone number", which is a phone number controlled by a "server host" that communicates directly over SMS using Android's SMS APIs. Each URL request is sent, encoded in a custom base 114, to the server. Usually, this only requires 1 SMS, but just in case, each message is prepended with an order specifier. When the server receives a request, the server uses an Android WebView component to programatically request the website in a manner that simulates a regular request, to avoid restrictions some services (such as Cloudflare) place on HTTP clients. By doing this, any Javascript can also execute on the website, allowing content to be dynamically loaded into the HTML if needed. Once the page is loaded, only the HTML is transferred back to the recipient device. The HTML is stripped of unnecessary tags and attributes, compressed into raw bytes, and then encoded. Once encoded, the messages are split into 160 character numbered segments (maximizing the [GSM-7 standard](https://en.wikipedia.org/wiki/GSM_03.38) SMS size) and sent to the client app for parsing and displaying.

Side note: Compression savings have been estimated to be an average of 20% using Brotli, but oftentimes it can save much more! For example, the website `example.com` in stripped HTML is 285 characters, but only requires 2 SMS messages (189 characters) to receive. Even including the 225% overhead in data transmission, it is still more efficient!

#### Why encode the HTML in the first place?
SMS was created in 1984, and was created to utilize the extra bytes from the data channels in phone signalling. It was originally conceived to only support 128 characters in a 7-bit alphabet. When further characters were required to support a subset of the UTF-8 character set, a new standard called UCS-2 was created. Still limited by the 160 bytes available, UCS-2 supports more characters (many of which show up in HTML documents) but limits SMS sizes to 70 characters per SMS. By encoding all data in GSM-7, more data can be sent per SMS message than sending the raw HTML over SMS. It is possible that it may be even more efficient to create an encoding system using all the characters available in UCS-2, but this limits compatibility and is out of the scope of the project.

## Server Hosting
TxtNet Browser has been rewritten to include a built-in server hosting option inside the app. Instead of the now-deprecated Python server using a paid SMS API, any user can now act as a server host, allowing for distributed communication.  
To enable the background service, tap on the overflow menu and select "TxtNet Server Hosting". Once the necessary permissions are granted, you can press on the "Start Service" toggle to initialize a background service.  
TxtNet Server uses your primary mobile number associated with the active carrier subscription SIM as a number that others can add and connect to.  
Please note that this feature is still in early stages of development and likely has many issues. Please submit issue reports for any problems you encounter.  
For Android 4.4-6.0, you will need to run adb commands one time as specified in the app. For Android 6.0-10.0, you may also use Skizuku, but a PC will still be required once. For Android 11+, no PC is required to activate the server using [Shizuku](https://shizuku.rikka.app/guide/setup/).


##### Desktop Server Installation (Deprecated)
<details>
 The current source code is pointed at my own server, using a Twilio API with credits I have purchased. If you would like to run your own server, follow the instructions below:  
1. Register for an account at [Twilio](https://twilio.com/), purchase a toll-free number with SMS capability, and purchase credits. (This project will not work with Twilio free accounts)  
2. Create a Twilio application for the number.  
3. Sign up for an [ngrok](http://ngrok.com/) account and download the ngrok application  
4. Open the ngrok directory and run this command: `./ngrok tcp 5000`  
5. Visit the [active numbers](https://console.twilio.com/US1/develop/phone-numbers/manage/incoming) page and add the ngrok url to the "A Message Comes In" section after selecting "webhook". For example: "https://xyz.ngrok.io/receive_sms"  
6. Download the TxtNet Browser [server script](https://github.com/lukeaschenbrenner/TxtNet-Browser/blob/master/SMS_Server_Twilio.py) and install all the required modules using "pip install x"  
7. Add your Twilio API ID and Key into your environment variables, and run the script! `python3 ./SMS_Server_Twilio.py`  
8. In the TxtNet Browser app, press the three dots and press "Change Server Phone Number". Enter in the phone number you purchased from Twilio and press OK!  
</details>

## FAQ/Troubleshooting

Bugs:
- Many carriers are unnecessarily rate limiting incoming text messages, so a page may look as though it "stalled" while loading on large pages. As of now the only way to fix this is to wait!
- In congested networks, it's possible for a mobile carrier to drop one or more SMS messages before they are received by the client. Currently, the app has no logic to mitigate this issue, so any websites that have stalled for a significant amount of time should be requested again.
- In Android 12 (or possibly a new version of Google Messages?), there is a new and "improved" messages blocking feature. This results in no SMS messages getting through when a number is blocked, which makes the blocking feature of TxtNet Browser break the app! Instead of blocking messages, to get around this "feature", you can silent message notifications from the server phone number.  

|  |  |  |
|--|--|--|
|  <img src="https://github.com/lukeaschenbrenner/TxtNet-Browser/raw/master/media/silentMessages.png" alt="Silence Number" width="200"/>  |  <img src="https://github.com/lukeaschenbrenner/TxtNet-Browser/raw/master/media/Messages_Migrating_Popup.png" alt="Contacts Popup" width="200"/>   |   <img src="https://github.com/lukeaschenbrenner/TxtNet-Browser/raw/master/media/MigratingBlockedContacts.png" alt="Migrating Contacts" width="200"/>  |

## Screenshots (Outdated, TxtNet 1.0.0)

<table>  
  <tr>  
    <td> <img src="https://github.com/lukeaschenbrenner/TxtNet-Browser/raw/master/media/screenshot1.png"  alt="1" height = 640px ></td>  
    <td><img src="https://github.com/lukeaschenbrenner/TxtNet-Browser/raw/master/media/screenshot2.png" alt="2" height = 640px></td>  
   </tr>   
   <tr>  
      <td><img src="https://github.com/lukeaschenbrenner/TxtNet-Browser/raw/master/media/screenshot3.png" alt="3" height = 640px></td>  
      <td><img src="https://github.com/lukeaschenbrenner/TxtNet-Browser/raw/master/media/screenshot4.png" align="right" alt="4" height = 640px>  
  </td>  
  </tr>  
</table>

##### Demo (Outdated, TxtNet 1.0.0)

https://user-images.githubusercontent.com/5207700/191133921-ee39c87a-c817-4dde-b522-cb52e7bf793b.mp4

> Demo video shown above


## Development

### 🚧 **If you are skilled in Android UI design, your help would be greatly appreciated!** 🚧 
Feel free to submit pull requests! As you may be able to tell when using the app, UI/UX is not my forte and some areas are seriously lacking. Because of the goal of maintaining wide support for Android 4.4+, libraries like Compose aren't compatible natively.
Reach out to me if you'd like to be a major contributor in any aspect of the app - I would greatly appreciate the help.

## Future Impact
My long-term goal with this project is to eventually reach communities where such a service would be practically useful, which may include:
- Those in countries with a low median income and prohibitively expensive data plans
- Those who live under oppressive governments, with near impenetrable internet censorship

If you think you might be able to help funding a local country code phone number or server, or have any other ideas, please get in contact with the email in my profile description!

## License

GPLv3 - See LICENSE.md

## Credits

Thank you to everyone who has contributed to the libraries used by this app, especially Brotli and Basest. Special thanks goes to [Coldsauce](https://github.com/ColdSauce), whose original project [Cosmos Browser](https://github.com/ColdSauce/CosmosBrowserAndroid) was the original inspiration for this project!  
My original reply to his Hacker News comment is [here](https://news.ycombinator.com/item?id=30685223#30687202).
In addition, I would like to thank [Zachary Wander](https://www.xda-developers.com/implementing-shizuku/) from XDA for their excellent Shizuku implementation tutorial and [Aayush Atharva](https://github.com/hyperxpro/Brotli4j/) for the amazing foundation they created with Brotli4J, allowing for a streamlined forking process to create the library BrotliDroid used in this app.
