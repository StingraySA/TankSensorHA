# DIY Water Tank Level Sensor for Home Assistant

This post will help you to build, configure and set up the tank level monitor using ESPHome in Home Assistant.

This project was created with the home DIY’er in mind. Since the SMD components are so teeny tiny these days, I have opted to use through hole components to make sure that you only need a regular soldering iron to assemble.

You will require the following items to assemble the PCB:  
\* 1 x D1 Mini Node MCU V4  
\* 2 x 2K 1/4w Through Hole Resistors  
\* 2 x 1K 1/4w Through Hole Resistors  
\* 1 x HLK-5M5 / HLK-5M05 Transformers  
\* 1 x 62 Ohm 1/4w Through Hole Resistor  
\* 1 x 5mm LED (Any color you prefer)  
\* 1 x 2-pin Screw Terminal  
\* 1/2 x Weather-proof Ultrasonic Sensor Probe AJ-SR04M (One or Two depending on if you want to monitor one or two tanks)  
\* 1 x Waterproof Enclosure ( I’ve opted for a 110cm x 110cm enclosure from my local hardware store )  
\* 1 x Tube of Marine Clear Silicone

If you do not have the PCB, you can download the Gerber files in this Repo.

![](https://github.com/StingraySA/TankSensorHA/Images/PCB.png)

The circuit is pretty self explanatory. Add the resistors and the LED in their designated spots. Next install the screw terminal. Before you install the transformer or the D1 Mini, we first need to upload our program onto the D1 Mini.

Open up your Home Assistant and click on the ESPHome tab. If you do not have ESPHome installed, you can follow this video: [https://www.youtube.com/watch?v=iufph4dF3YU](https://www.youtube.com/watch?v=iufph4dF3YU)

As per the video, click on New Device and select how you are connecting to it. Note for the first time install, you will need to plug the board into a USB port on either your Home Assistant appliance or if you are using Chrome, you can use your PC. Give the device a name and then ESPHome should give you an API key. Copy this key for use later.

You should be on a page where you can enter code. Go to the last line of code, and add the following:

    sensor:
      - platform: ultrasonic
        accuracy_decimals: 4
        trigger_pin: D2
        echo_pin: D1
        name: "Municipal Water Tank Sensor"
      - platform: ultrasonic
        accuracy_decimals: 4
        trigger_pin: D4
        echo_pin: D3
        name: "Rain Water Tank Sensor"

So in my example above, I have two water tanks. One that is filled with municipal water and the second with rain water. So name them according to your setup. The first tank is connected to H1 and the second to H2. If you are only using one tank, use the first set of parameters and leave the second.

Next you can upload the code to the D1 Mini. Once the code is uploaded it should connect to your Wi-Fi network and you should be seeing some output like below when you click on Logs and then Wirelessly.

![](https://crazystingray.co.za/wp/wp-content/uploads/2024/03/image-1.png)

![](https://crazystingray.co.za/wp/wp-content/uploads/2024/03/image-2.png)

Right, so now we are receiving the distance data from the sensors. When you go back to Home Assistant you should see that a new device has been discovered. Click on the new device and select add. Next Home Assistant will ask you for the API key. Paste in the key you copied when we created the new ESPHome device. Now you should see them under Settings -> Devices and Services -> ESPHome.

When you click on the Water Tank, you should have two entities like below:

![](https://crazystingray.co.za/wp/wp-content/uploads/2024/03/image-3.png)

Now we need to do some math. You will require the following measurements:

*   The height of the tank from the bottom to the lid the sensor will be installed in.
*   The height of the sensor from the lid to the full water level.
*   The water capacity of your water tank.

![](https://crazystingray.co.za/wp/wp-content/uploads/2024/03/image-8.png)

Using a text editor, edit your templates.yaml file in Home Assistant and add the following while replacing the values with your own measured values. Don’t worry if it’s a bit wonky still. We will fine tune them later. Do this for both tanks if you are using two and amend the names and values accordingly.

    
          # Rain Water Tank
        - name: "Full Water Height Rain Tank"
          state: "151.65"
          unit_of_measurement: cm
        - name: "Sensor Height From Water Level Rain Tank"
          state: "22.982"
          unit_of_measurement: cm
        - name: "Water Tank Max Capacity Rain Tank"
          state: "2500"
          unit_of_measurement: "l"
        - name: "Water Tank Level Percentage Rain Tank"
          state: >
            {{ (((( states('sensor.full_water_height_rain_tank')|float(0) +
            states('sensor.sensor_height_from_water_level_rain_tank')|float(0)) -
            states('sensor.rain_water_tank_sensor')|float(0)*100) /
            states('sensor.full_water_height_rain_tank')|float(0)) *100)
            | round(2) }}
          unit_of_measurement: "%"
        - name: "Water Tank Current Capacity Rain Tank"
          state: >
            {{ (states('sensor.water_tank_max_capacity_rain_tank')|float(0) *
            (states('sensor.water_tank_level_percentage_rain_tank')|float(0) / 100))
            | round(2) }}
          unit_of_measurement: "l"

Save the file and click on Developer Tools in Home Assistant and click on Check Configuration. If all is good you will get a message that the config will not stop Home Assistant from starting. Scroll down and click on Template Entries. This will refresh all the values we’ve just added.

Next we need to add them to Home Assistant. The entities will all be there, so you can add them to any dashboard.

![](https://crazystingray.co.za/wp/wp-content/uploads/2024/03/image-4.png)

Now we need to install the sensors on the tanks. Install the PCB into the enclosure and connect each of the sensor wires to their respective daughter boards. Make sure the holes you’ve made into the enclosure is sealed with marine silicone. Next install the sensor into the lid of the tanks and seal them with silicone.

![](https://crazystingray.co.za/wp/wp-content/uploads/2024/03/image-5.png)

Route the wires and attach the enclosure where water will not be able to get into it. I prefer off the ground. Run a power line to the enclosure and connect it to the screw terminals and once again seal the hole you’ve made for this wire. Now we are done with the enclosure. Close it up and mount it against a tank.

![](https://crazystingray.co.za/wp/wp-content/uploads/2024/03/image-6.png)

Now that it’s mounted you can power it on and go back to Home Assistant to wait for it to connect to your Wi-Fi again. Now you can check the values that is shown in Home Assistant and modify your Template.yaml to fine tune the values so that 100% is when the tank is filled.

Note that there is one caveat to these sensors. There are some drawbacks to not using the very expensive ultrasonic sensors. One is that it is prone to false readings due to condensation. This can be alleviated by installing a mesh breather into your tank.

![](https://crazystingray.co.za/wp/wp-content/uploads/2024/03/image-7.png)

The second is that when it rains and water is flowing into the tank, it can interfere with the ultrasonic sounds and may give some weird readings. Thirdly bats are proving to be very chatty when they get close to the tanks. It doesn’t hurt them at all, they just have to screech a bit more to locate their own signals.

I have been using these sensors for almost three years now and have had no major issues with them which is why I’ve made them available to everyone 🙂
