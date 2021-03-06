# Tasmota_QS-WiFi-D01-TRIAC
Tasmota QS-WiFi-D01-TRIAC Wifi Dimmer Module Configuration  
Tasmota Program Version	9.5.0 at the time of writing this  

Reference #1 https://github.com/arendst/Tasmota/issues/5737  
Reference #2 https://templates.blakadder.com/qs-wifi_D01_dimmer.html

**What is missing or not very clear in the Referenced links above**

1) The GPIO pin numbering has changed in the current version    
   Refer to https://tasmota.github.io/docs/GPIO-Conversion/ for changes in the GPIO numbering.  
   the new version template is below (The old version still works but it automatically updates with the new version when applied)  
   **_Template {"NAME":"QS-WiFi-D01-TRIAC","GPIO":[0,3200,0,3232,0,0,0,0,0,352,416,0,0],"FLAG":0,"BASE":18}_**

2) After applying the template the Module needs to be set to the newly created template  
   In the web interface go to Main Menu -> Configuration -> Configure Module, Select "QS-WiFi-D01-TRIAC(0)" from the drop list and save  
   **OR**  
   In the console type **_Module 0_** to select the newly created template.

3) In the console type **_Baudrate 9600_** to set the Baud Rate for the serial tx/rx pins, The default value is 115200 and does not work for this device.  
   After setting the Baudrate you can test if the Power on/off works using the below commands in the console.
   
   Power OFF  
   **_SerialSend5 FF550005DC0A_**  
   
   Power On  
   **_SerialSend5 FF55FF05DC0A_**  
   
   To change the brightness, replace the xx below with hexadecimal values between 00 and FF ( Decimal 0-255 )  
   **_SerialSend5 FF55xx05DC0A_**  
   
   SerialSend6 can also be used with comma seperated decimal values as setup in the **rule 1** below. 

4) For this device to work with the above template we need to set up rules to send the correct serial commands on power off/on and on the use of the dimmer  
   the below rule works perfectly for my device. ( Updated on 22 Sep 2021 to retain dimmer level after device restart. )    
   ```
   rule1 
      on Dimmer#State do backlog scale1 %value%,1,100,16,255; event decdimmer; endon
      on Dimmer#Boot do backlog scale1 %value%,1,100,16,255; event decdimmer; endon
      on event#decdimmer do SerialSend6 255,85,%var1%,5,220,10 endon
      on Power1#State=0 do SerialSend6 255,85,0,5,220,10 endon 
      on Power1#State=1 do SerialSend6 255,85,%var1%,5,220,10 endon  
   ```
   Remember to turn on the rule by running **_rule1 1_** in the console  
   **Note :** **_scale1 %value%,1,100,16,255;_** in the above rule, scales the dimmer input from a range of 1-100 received to a range of 1-255 required for the serialsend6 command to the device.  
   I used 16 instead of 1 because that is the lowest value at which my attached LED switches on without any flickering, your device might need a different starting value.

  5) If you need to quickly reset the device without wiping the wifi credentials use the command **_reset 6_** in the console. 
   
Reference #3 https://tasmota.github.io/docs/Commands/  
Reference #4 https://tasmota.github.io/docs/Rules/  
   

**_Update ( 23 Sep 2021 )_**

6) "Switch S" on the device can be used with a basic momentary push button switch to turn the device on and off in case of a wifi failure.  
    For this to work GPIO13 needs to be set to "Button" instead of "Counter" in the Template OR Configuration.  
    For some reason it does not work when GPIO13 is configured as a "switch", possibly a hardware limitation.  
    
   **Template {"NAME":"QS-WiFi-D01-TRIAC","GPIO":[0,3200,0,3232,0,0,0,0,0,32,416,0,0,1],"FLAG":0,"BASE":18}**

   Run the below commands in the console to create and enable rule2 to handle the switch functions. 
   
   ```
   Backlog ButtonTopic 0;  SetOption73 1; SetOption1 1
   Rule2 1
   Rule2
      ON button1#state=10 DO power1 toggle ENDON
      ON button1#state=11 DO power1 toggle ENDON
      ON button1#state=12 DO power1 toggle ENDON
      ON button1#state=13 DO power1 toggle ENDON
      ON button1#state=14 DO power1 toggle ENDON
      ON button1#state=3  DO power1 toggle ENDON
   ```
   This rule just toggles the power state on button press for now, It can be used along with backlog to restore the brigtness to full if needed.     
   
   replace "power1 toggle" with "backlog SerialSend6 255,85,255,5,220,10; power1 toggle" on all rule2 lines to restore to full brightness when using the switch. 
   
   I am looking for a way to handle the dimmer function from the same rule but that does not seem possible with the available commands at this time. 


   
