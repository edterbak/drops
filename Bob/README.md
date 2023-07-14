### Robo Vacuum cleaner underneath the kichen cabinets
I have created a nice place for Bob, our roborock s5 max cleaner. </br>
</br>

![Bob cave](https://github.com/edterbak/drops/blob/main/Bob/bob_cave1.png?raw=true).
</br>
This build:</br>
- Includes 2 buttons which ALWAYS allows you to open/close the hatch. Even without internet. (You only need power ofcourse)</br>
- Includes relay board (wifi) so you can control it remote/ automated.</br>
- Includes pwm module so you can set your prefered speed to open/close the cave.</br>
- is idiot proof in control. wifi signal open/close together with manual buttons open/close wont kill it.</br>
- is idiot proof in control. pushing open/close at the same time wont kill it.</br>
- Relay board controlled through wifi (MQTT) through ESP-Home. </br>
- Uses voice command to open/close cave: (Alexa/Google > webhook > node red)</br>
</br></br>
See below a movie how it functions and responds after asking alexa to show bob.</br>
https://github.com/edterbak/drops/raw/main/VID20220101143654.mp4
</br>

### Components used
<b>Actuator:</b></br>
type: 180N 50mm per second, 150mm stroke</br>
price: 42.33 Euro</br>
link: https://nl.aliexpress.com/item/4000849922418.html?spm=a2g0o.order_list.order_list_main.15.221d79d2QEp7vz&gatewayAdapt=glo2nld 
</br>
</br>
<b>Power supply:</b></br>
type: 12V, China, 36W</br>
price: 7.29 Euro</br>
link: https://nl.aliexpress.com/item/1005002728038351.html?spm=a2g0o.order_list.order_list_main.20.221d79d2QEp7vz&gatewayAdapt=glo2nld
</br>
</br>
<b>limit switch:</b></br>
type: R Handle 25MM</br>
price: 2.32 Euro</br>
https://nl.aliexpress.com/item/1005001701734198.html?spm=a2g0o.order_list.order_list_main.25.221d79d2QEp7vz&gatewayAdapt=glo2nld
</br>
</br>
<b>4ch-relay board:</b></br>
price: 12.55 Euro</br>
Note: 2CH would be sufficient. I intend to use the other 2 for some cool 12V light effects inside the 'cave'. </br>
https://nl.aliexpress.com/item/4001084228258.html?spm=a2g0o.order_list.order_list_main.30.221d79d2QEp7vz&gatewayAdapt=glo2nld
</br>
</br>
<b>PWM motor regulator:</b></br>
price: 4.33 Euro</br>
Note: this little unit comes with 2 nice buttons to mount next to the door. </br>
https://nl.aliexpress.com/item/32963777534.html?spm=a2g0o.order_list.order_list_main.35.221d79d2QEp7vz&gatewayAdapt=glo2nld
</br>
</br>
<b>Hinge:</b></br>
price: 12.99/pcs Euro</br>
link: https://www.mijnijzerwaren.nl/hang-en-sluitwerk/46262-klepbeslag-links-m-veer-kb57-vern.html
</br>
</br>
Total price: 94.80 Euro
</br>

### Design
Make sure you know how much space you need for the robot. </br>
Include the charger, assume the robot is on the charger. </br>
That total length combined with the hight of the cleaner, is what you need to know befor you deside if you open the door inward or outward. </br>
In my case, I could not 'pull' the door open inward, because that would not fit with the cleaner behind it. </br>
</br>
If the door opens outward, choose the correct hinge. If the cabinet is directly above the opening door, and can not go higher, you need to choose the right hinge to alow such movement. </br>
</br>
Choose the correct actuator stroke-length.</br>
If it is too short, the door wont be opened/closed fully.</br>
If it is too long, the door is closed while the actuator is not at its end yet. </br>
Idealy, you want the fully extended actuator connected to the opened door position.</br>
To solve a too large closing stroke, I used an limit-switch to prevent it killing itself. </br>
Note: Actuators are very powerfull. If the closing stroke is not done or interupted when the door is closed, it will simply pull something off to close. It is very forcefull. </br> 
</br>
### Ellecrical Wiring
![Schema](https://github.com/edterbak/drops/blob/main/Bob/2023-01-06%2021_43_18-Drawing3.vsdx.png?raw=true).
</br>
putting the power supply and 4CH relayboard on a solid piece of wood.</br>

<img src="https://github.com/edterbak/drops/blob/main/Bob/IMG-20211013-WA0018.jpg?raw=true" height="300">
Note that the wire color coding is incorrect. But this is what I had available. (Should be black/red)

</br></br>
### programming
Below is the code flashed into the 4ch relay board, using ESP-Home. </br>
The trigger to open the door is received through MQTT topic. </br>
- esp01relay/switch/relay_1/state (value "OPEN"/"CLOSE")</br>
- esp01relay/switch/relay_2/state (value "OPEN"/"CLOSE")</br>
</br>
After the trigger has been received, it executes the open/close sequences.</br>
- close relay </br>
- wait 5 seconds</br>
- open relay again. </br>
This for both relays, only relay 1 is used to power actuator for opening, relay 2 is for closing.</br>
</br>

```
esphome:
  name: esp01relay
  platform: ESP8266
  board: esp01_1m

wifi:
  ssid: "xxxxx"
  password: "xxxxxx"
  manual_ip:
    static_ip: x.x.x.x
    gateway: x.x.x.x
    subnet: 255.255.255.0

# Enable logging
logger:

# Enable Home Assistant API
#api:
#  password: "xx"

ota:
#  password: "xx"

mqtt:
  broker: x.x.x.x
  username: "x"
  password: "x"
  on_message:
    - topic: esp01relay/switch/relay_1/state
      payload: "OPEN"
#      qos: 0
#      retain: false
      then:
        - switch.turn_on: relay1
    - topic: esp01relay/switch/relay_2/state
      payload: "CLOSE"
#      qos: 0
#      retain: false
      then:
        - switch.turn_on: relay2

# Sensors with general information. Nice to have
sensor:
  # Uptime sensor.
  - platform: uptime
    name: Relay Uptime

#  # WiFi Signal sensor. (Optional)
#  - platform: wifi_signal
#    name: Relay WiFi Signal
#    update_interval: 60s

uart: # Defines the communication to the relayboard
  baud_rate: 115200 # speed to STC15L101EW
  tx_pin: GPIO1
  rx_pin: GPIO3

#------------------------------------------------------------------------------
#------------------------------------------------------------------------------
# reference for switches on relay board!
#      - uart.write: [0xA0, 0x01, 0x01, 0xA2] # 1 ON: close connection
#      - uart.write: [0xA0, 0x01, 0x00, 0xA1] # 1 OFF: open connection
#
#      - uart.write: [0xA0, 0x02, 0x01, 0xA3] # 2 ON: close connection
#      - uart.write: [0xA0, 0x02, 0x00, 0xA2] # 2 OFF: open connection
#
#      - uart.write: [0xA0, 0x03, 0x01, 0xA4] # 3 
#      - uart.write: [0xA0, 0x03, 0x00, 0xA3] # 3 
#
#      - uart.write: [0xA0, 0x04, 0x01, 0xA5] # 4 
#      - uart.write: [0xA0, 0x04, 0x00, 0xA4] # 4 
#------------------------------------------------------------------------------
#------------------------------------------------------------------------------

switch: # Optional extra switch to restart the board.
  - platform: restart
    name: esp01relay-restart
    id: restart_switch

#------------------------------------------------------------------------------
# Relay 1: This relay is used to open the door
# The sequence of switching is defined here.
  - platform: template
    name: 'Relay 1' # ON/OFF 
    id: relay1
    turn_on_action:
      - uart.write: [0xA0, 0x01, 0x01, 0xA2] # 1 ON switch
      - delay: 5s
      - switch.turn_off: relay1
      - uart.write: [0xA0, 0x01, 0x00, 0xA1] # 1 OFF switch
    turn_off_action:
      - uart.write: [0xA0, 0x01, 0x00, 0xA1] # 1 OFF switch
    optimistic: true
    restore_state: false

#------------------------------------------------------------------------------
# Relay 2: This relay is used to close the door
# The sequence of switching is defined here.
  - platform: template
    name: 'Relay 2' # ON/OFF  
    id: relay2
    optimistic: true
    restore_state: false
    turn_on_action:
      - uart.write: [0xA0, 0x02, 0x01, 0xA3] # 2 ON switch
      - delay: 5s
      - switch.turn_off: relay2
      - uart.write: [0xA0, 0x02, 0x00, 0xA2] # 2 OFF switch
    turn_off_action:
      - uart.write: [0xA0, 0x02, 0x00, 0xA2] # 2 OFF switch

```
</br>

![Actuator with limit-switch](https://github.com/edterbak/drops/blob/main/Bob/IMG20230304103959.jpg?raw=true)
Actuator with limit-switch on the inside of the hatch</br></br>
![PWM board connected](https://github.com/edterbak/drops/blob/main/Bob/IMG20230304104559.jpg?raw=true)
Connection to the PWM board. Including the limit switch.</br>
The PWM board has two manual buttons. I have put the limit-switch between one of the plug <-> button lines.
