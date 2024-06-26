---
title: 'SPADE: Spotify Powered AI DJ Experience'
title: 'SPADE: Spotify Powered AI DJ Experience'
author: 'Sayan Bhatia & Dhilan Patel'
date: '*EEC172 SQ24*'


toc-title: 'Table of Contents'
abstract-title: '<h2>Description</h2>'
abstract: 'Alexa is often used to play music in the household. However, only newer Alexa’s are accompanied by a visual screen. Our project attempts to solve this problem.
Alexa is also not powered by today’s state of the art large language models and we allow for a cohesive DJ experience using GPT-4. We created an AI powered spotify system that can change what the user is listening to, poll ChatGPT for more information about the song, speak said information over Alexa, and display all of the song information and user status on an OLED.

<br/><br/>

<h2>Video Demo</h2>
<div style="text-align:center;margin:auto;max-width:560px">
  <div style="padding-bottom:56.25%;position:relative;height:0;">
    <iframe style="left:0;top:0;width:100%;height:100%;position:absolute;" width="560" height="315" src="https://www.youtube.com/embed/BOhCVAxeftI?si=R8y37Bqj9OpNV9br" title="YouTube video player" frameborder="0" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
</div>
'
---



# Design

## System Architecture

### Software Architecture

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 400px;">
    For the software component, we had two primary functions, 
Receive new data (modify playback, ex. seek, skip, pause) from the CC3200 
Process this data to execute certain actions crucial to the overall integration and product experience (AI commentary played through Alexa, and the song playback modified)

Here is how each component plays a role in the software architecture:

 - AWS Lambda: The central data processing component that ran all API-code in the cloud. Our whole Lambda function executed several actions in this order:

   - Receive modify playback data from AWS IotCore and the shadow for our IoT Thing
   - Constantly retrieve the current playing song from Spotify API
   - Modify playback based on retrieved data from Shadow
   - Send current playing song to GPT-4, a LLM hosted by OpenAI, asking for an interesting fact that intrigues the user
   - Receive commentary from GPT-4 and send commentary to Alexa to play

 - AWS IotCore Shadow: Used to retrieve new updates and data from our CC3200

 - Spotify API - The service we used to control the song and playback connected to our Spotify Account

 - ChatGPT API - Used to generate ‘DJ Commentary’ for the current song. This commentary is later played by Alexa as a medium to give the information to the user

 - Alexa Voice Third-Party API - A Text-To-Speech service that allows us to send text and play it on the Alexa


  </div>
  <div style="display:inline-block;vertical-align:top;flex:0 0 600px;">
    <div class="fig">
      <img src="media/software_architecture.png" style="width:100%;height:100%;" />
    </div>
  </div>
</div>

## Hardware Architecture

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 400px;">
    The IR Sensor and remote is for the user to wirelessly send commands to the DJ, such as changing the song and displaying information on the OLED. The Pushbutton is used to change stylistic settings of the system, such as the font layout and color. The 802.11 Wifi module connects with AWS to receive all of the Spotify song information, as well as change the user’s information when they want to change what they are playing through the DJ system. The OLED Display shows all of the song information as well as some graphics for the user’s song statuses. The Secondary display/Device is another peripheral that shows all of the sentences generated by chatGPT. The CC3200 brings all of these peripherals together and does the main processing to determine what actions to take, as well as how to communicate with these separate systems.

  </div>
  <div style="display:inline-block;vertical-align:top;flex:0 0 600px;">
    <div class="fig">
      <img src="media/hardware_architecture.png" style="width:100%;height:100%;" />
    </div>
  </div>
</div>

## Functional Specification

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 300px;">
    
Our product is designed to manage and display song information on an OLED display, utilizing various input methods and commands to update or change the displayed information. When the system starts, it initiates a waiting period of 5 seconds to stabilize before checking for any input. After this waiting period, the system checks if the push button is pressed. If the button is pressed, the system changes the format and color of the display and updates it accordingly. If the button is not pressed, the system then checks for inputs from an IR remote.

If the IR remote is used, the system differentiates between AWS and UART commands. For an AWS command, the system sends a prompt to AWS to perform a song change. For a UART command, the system sends the information via a UART wire. If the IR remote is not used, the system proceeds to retrieve song information from AWS. Once the song information is retrieved, the system checks if there is new song information available. If there is new song information, the system updates the display with the new song title, album, and artist. If there is no new song information, the system loops back to the start state.
  </div>
  <div style="display:inline-block;vertical-align:top;flex:0 0 500px">
    <div class="fig">
      <img src="media/FSM_hardware.png" style="width:90%;height:auto;" />
      <span class="caption">Hardware State Diagram</span>
    </div>
  </div>
</div>

# Implementation

### Software

To ‘invoke’ our Lambda function we used AWS StepFunctions which allowed us to constantly run our Lambda function every 10 seconds to retrieve current playing song info and data updates from AWS Iot Core. The Finite State Machine Diagram above displays the step function utility which essentially invoked Lambda, waits 10 seconds, and runs again. Our FSM times out after 20 minutes to minimize spending and cloud resource usage.

To authenticate with Spotify, we had to follow the Authentication Flow specified by Spotify. This requires us to do the following:
 - Authenticate to get an access code and refresh token
 - Use the refresh token in our environmental variables and retrieve an access token with appropriate permissions, specifically: user-read-playback-state, user-modify-playback-state.
 - Use the access token to modify and read playback from our Spotify Account

To GET and POST data to the shadow in AWS IotCore we used boto3 to establish a connection to the client and sent information or received information while specifying the name and region of our ‘thing’. When modifying playback we would read from the shadow whether, ‘seek’, ‘skip’, or ‘pause’ were True. The state that was true was then relayed to spotify to modify the playback accordingly. 

Making requests to ChatGPT and Alexa was a simple POST request with the songInfo and commentary respectively. We package the requests library and added as a layer to our lambda function to make API requests to these APIs. When making a request to GPT-4 we would package the song name, album name, and artists names received from Spotify into a neat string with a prompt asking for an interesting fact.

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:0 0 400px'>
    <div class="fig">
      <img src="media/step_function.png" style="width:auto;height:auto" />
      <span class="caption">AWS Step Function FSM</span>
    </div>
  </div>
</div>

### Hardware

- **IR Sensor and Remote**:
  - Hardwired the IR circuit to a GPIO pin on the microcontroller
  - Used interrupts to decode the timing of the buttons on the remote
  - Based on the timing characteristic, perform that respective command in the software in the GPIO input’s handler.

- **Pushbutton**:
  - Already soldered onto the microcontroller
  - Used a software interrupt to determine when the button was pressed
  - Whenever the button was pressed, cycled between several color and format characteristics of the display, implemented in the loop of the button’s handler.

- **802.11 Wifi Module**:
  - Mostly implemented via software
  - Connected to the local network on the program’s initialization
  - Every 5 seconds, the module connects to AWS IOT to pull the song information from the shadow, then gives it to CC3200
  - If the CC3200 has a command to send to AWS IOT, it prompts the wifi module to send that information over to update the shadow.

- **OLED Display**:
  - CC3200 updates the display every 5 seconds if there is new information to be processed
  - Uses SPI to send pixel information regarding various icons (pause, play, loop) and character data for the song title

- **Secondary Display/Device**:
  - UART connection via output pin that can be connected to another device
  - In our case, we used a second CC3200 to take the information in and display it on another OLED. However, this wire can go to any device that can process UART as an input
  - Whenever a command is sent from the main CC3200, it takes a string representing the paragraph to be sent, and sends it over UART protocol to a receiving device.

- **CC3200**:
  - Processes all of the tasks to be done from the peripherals, connects their interactions together
  - When the GPT button is pressed on the remote, prepare and send the paragraph over UART
  - When buttons 1-4 are pressed, upload that specific command to AWS to update Spotify data
  - Sets up the 5-second intervals to retrieve new data
  - When a new song is detected, update the OLED display via SPI
  - When the pushbutton is pressed, interrupt the current task to update the font and format.


# Challenges

The main challenge we had to deal with was getting the commands to interact with the AWS IOT. We were able to push information to the IOT fine, but it was hard to parse what we were getting in order to determine the information on the song. For some reason, we could not access a value in a specific variable, but we could access the whole directory in the IOT. So, we solved this problem by accessing every value at once, and starting to read the information into a variable (for instance, the song title was the first variable and was always the 257th character in the string) character by character until a certain identifier was encountered. Once that identifier was encountered, we skipped a certain number of characters and moved on to reading for the next variable (ex. artist name) until another identifier was found. Once we read the last variable needed, we just threw away the rest of the string. We did not have this issue with writing variables for some reason, it was just with reading.

We also had issues with wiring all of the peripherals together, as we ran into conflicts with the allowed pins in our Pin mux configuration. It took us a while to try different pins for each peripheral, UART, and SPI until we got everything to work together.

Another challenge we ran into was figuring out how to display the spotify image data onto the OLED. We wanted to make a second menu which exclusively displayed such images, as the adafruit library on arduino can display image files sent to it. This proved to be a challenge however, as we had to take the base64 encoded image, convert it into 3 separate RGB bitmaps, and then write a function that can identify the bitmaps to produce a color intensity on each pixel. We gave up on this implementation because it would have detracted from our other functionalities, and we may have not been able to even finish it with the given time.


With regards to software we had a couple main hurdles that were eventually overcome. First, authenticating with Spotify turned out to be less intuitive than we expected. When authenticating with Spotify, Spotify typically expects the user to authenticate and allow ‘access’ for the Third-Party to access their account data via the web and a frontend. Since this was not possible, it made it hard to find a way to authenticate and retrieve both the access code and refresh token. We eventually found a way to manually authenticate ahead of time which involved using a frontend with a callback URL to return back to our code. From here we received the access code and refresh token which we used in our code.

Another challenge we encountered was sending the AI commentary to Alexa. Alexa does not allow text-to-speech through any official API. Since we were doing our analysis with a state of the art LLM not present in Alexa at the moment, we had to find a way to send our own generated commentary to Alexa. To accomplish this we used a third party voice plugin which connected to our Alexa account to play text. This was mostly handled by the enabling of this plugin except for a POST request written in our code made to the plugin itself with the commentary.


# Future Work

The first thing we would like to get done in the future is being able to periodically refresh the album cover on the OLED display. This was hard for us to achieve within the deadline, as we would have done it if we had extra time.

The second thing we wanted to work on was implementing an ultrasound sensor to periodically pause and play the song on Spotify. The idea was for the user to hold their hand out in front of the DJ as if telling it to wait, and AWS would keep the song paused as long as their hand was up there. Once the hand moved, the song would resume.

Another feature we would like to work on is getting the information to display on a larger display. This would have been a higher priority if we got the album covers to display since 128x128 does not show enough detail on most artist’s covers.



# Finalized Bill of Materials


<table style="border-collapse:collapse;">
<thead>
  <tr>
    <th><p>No.</p></th>
    <th><p>PART NAME</p></th>
    <th><p>DESCRIPTION</p></th>
    <th><p>Qty</p></th>
    <th><p>SUPPLIER / MANUFACTURER</p></th>
    <th><p>UNIT COST</p></th>
    <th><p>TOTAL PART COST</p></th>
    <th><p>Purpose</p></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><p>1</p></td>
    <td><p>TI CC3200</p></td>
    <td><p>MCU, for processing tasks and connecting peripherals</p></td>
    <td><p>1-2</p></td>
    <td><p>TI</p></td>
    <td><p>$65 - $70</p></td>
    <td><p>$65 - $140</p></td>
    <td><p>Control Remote and Local Devices, process UART information</p></td>
  </tr>
  <tr>
    <td><p>2</p></td>
    <td><p>OLED Display</p></td>
    <td><p>128x128 RGB OLED Display, SPI protocol</p></td>
    <td><p>1-2</p></td>
    <td><p>Adafruit</p></td>
    <td><p>$40.00</p></td>
    <td><p>$40 - $80</p></td>
    <td><p>Display information, including GPT data</p></td>
  </tr>
  <tr>
    <td><p>3</p></td>
    <td><p>Remote</p></td>
    <td><p>General-purpose remote control</p></td>
    <td><p>1</p></td>
    <td><p>Various</p></td>
    <td><p>$43 - $47</p></td>
    <td><p>$43 - $47</p></td>
    <td><p>Allow user inputs</p></td>
  </tr>
  <tr>
    <td><p>4</p></td>
    <td><p>IR Transceiver</p></td>
    <td><p>For IR communication</p></td>
    <td><p>1</p></td>
    <td><p>Adafruit</p></td>
    <td><p>$1.95</p></td>
    <td><p>$1.95</p></td>
    <td><p>Receive signals from the remote</p></td>
  </tr>
  <tr>
    <td><p>5</p></td>
    <td><p>AWS Account</p></td>
    <td><p>Account for cloud services</p></td>
    <td><p>1</p></td>
    <td><p>AWS</p></td>
    <td><p>Free</p></td>
    <td><p>Free</p></td>
    <td><p>Cloud computing and storage</p></td>
  </tr>
  <tr>
    <td><p>6</p></td>
    <td><p>Wires</p></td>
    <td><p>Various connecting wires</p></td>
    <td><p>12-15</p></td>
    <td><p>Various</p></td>
    <td><p>$2 - $8</p></td>
    <td><p>$2 - $8</p></td>
    <td><p>Connect components together</p></td>
  </tr>
  <tr>
    <td><p>7</p></td>
    <td><p>Spotify Account</p></td>
    <td><p>Subscription to Spotify</p></td>
    <td><p>1</p></td>
    <td><p>Spotify</p></td>
    <td><p>Varies</p></td>
    <td><p>Varies</p></td>
    <td><p>Access to Spotify services</p></td>
  </tr>
  <tr>
    <td><p>8</p></td>
    <td><p>AWS Lambda and Step Function Costs</p></td>
    <td><p>Free trial runs in the starter account</p></td>
    <td><p>1</p></td>
    <td><p>AWS</p></td>
    <td><p>Free</p></td>
    <td><p>Free</p></td>
    <td><p>Run code in the cloud</p></td>
  </tr>
</tbody>
</table>
