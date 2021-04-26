# M5StackUIFlowCode
Code to get humidity, pressure and temperature from an ENV II sensor using M5Stack and publishing the data to AWS over MQTT
Developing a Environment Monitor using M5Stack, AWS and Grafana


CPSC-8810 Building a Environment Monitor Using M5Stack, AWS and Grafana

Developing a Environment Monitor using M5Stack, AWS and Grafana

Objectives:

In this lab you will:

- Learn how to setup the M5Stack
- Register a thing in AWS IoT Core
- Create a Rule in AWS IoT Core
- Create a TimeStream DB in AWS
- Visualize TimeStream Data in Grafana
- Code using UI Flow in Blocky or MicroPython

Requirements:

- **An AWS account**

[https://aws.amazon.com/](https://aws.amazon.com/)

- **M5 Stack device** [https://devices.amazonaws.com/detail/a3G0h000007djMLEAY/M5Stack-Core2-ESP32-IoT-Development-Kit-for-AWS](https://devices.amazonaws.com/detail/a3G0h000007djMLEAY/M5Stack-Core2-ESP32-IoT-Development-Kit-for-AWS)

or

optionally any IOT device that can communicate over MQTT protocol and send data to AWS

- **ENV II Sensor**

[https://m5stack.hackster.io/products/env-ii-unit-with-temperature-humidity-pressure-sensor-sht30-bmp280](https://m5stack.hackster.io/products/env-ii-unit-with-temperature-humidity-pressure-sensor-sht30-bmp280)

- **A Grafana account**

[https://grafana.com/](https://grafana.com/)

# **CHAPTER 1: Setting up the device**

**Pre-requisites for configuring the M5Stack**

1. Download and Install the CP210x USB to UART Bridge VPC Driver

Steps to install the driver: [https://m5stack.github.io/UIFlow\_doc/en/en/base/Update.html](https://m5stack.github.io/UIFlow_doc/en/en/base/Update.html)

Links to download the required software:

  1. [https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)
  2. [https://shop.m5stack.com/pages/download](https://shop.m5stack.com/pages/download)

1. Download and Install the M5 Burner: [https://shop.m5stack.com/pages/download](https://shop.m5stack.com/pages/download)

**Burning the UIFlow Firmware onto the M5Stack**

1.

- Double-click to open the M5Burner
- ① Select the corresponding device type in the left menu
- ② Select the firmware version you want
- ③ Click the download button to download.

![](RackMultipart20210426-4-15d0ali_html_c307d323f6a4b544.png)

In our case we will be downloading the UIFlow\_Core2 latest version to burn onto the M5Stack

2. Then connect the M5 device to the computer through the Type-C cable, select the corresponding COM port (For MAC devices the port may read something like /dev/tty.SLAB\_USBtoUART), the baud rate can use the default configuration in M5Burner,

    - ④ Click Erase on the right to erase Flash, close the current page when finished.
    - ⑤ Click &quot;Burn&quot; to start burning, you can input WIFI configuration information during burning.

3. When the burning log prompts  **Burn Successfully** , it means that the firmware has been burned.

**Configuring the WIFI on the M5Stack**

1. Click the power button on the left side of the device to turn it on. The UIFlow Logo appears on the screen.

![](RackMultipart20210426-4-15d0ali_html_f622af03a893e4d1.png)

2. After entering the main page, press the  **Setup**  button on the screen.

![](RackMultipart20210426-4-15d0ali_html_880211372ac5fff2.png)

3. In the  **WiFi**  option, press the start button of the config Wi-Fi by web option, and the device will automatically restart, where selected Wi-Fi is the last connected WiFi.

![](RackMultipart20210426-4-15d0ali_html_4e2a0a04a7c7bf3e.png)

4. After the device jumps, the WiFi Config page will be displayed.

![](RackMultipart20210426-4-15d0ali_html_103b7c00493da354.png)

5. Follow the prompts to connect to the SSID hotspot through the WiFi of the mobile phone or computer.
  1. Open the browser to visit  **192.168.4.1** and enter the WiFi information in the pop-up page to configure the network successfully.
  2. After the configuration is successful, the device will automatically restart. And enter the programming mode.

![](RackMultipart20210426-4-15d0ali_html_d56ccc7e32d86b13.png)

_Connect to the SSID shown on the M5Stack through your computer or Mobile_

![](RackMultipart20210426-4-15d0ali_html_a212fa68de0b36b6.png)

_Enter the SSID (Name of the WIFI) and password of the Wifi you want to connect your M5Stack to. (Note: The M5Stack may not allow you to connect to public WIFI&#39;s such as EduRoam or Clemson Guest, or even any 5Ghz networks. You may connect to your mobile hotspot or any private 2.4 Ghz Wifi such as your home WIFI)._

**Connecting the M5Stack to UIFlow**

1. The device automatically restarts on a successful WIFI Connection and display the following screen:

![](RackMultipart20210426-4-15d0ali_html_ca64641e385b81e5.png)

Note: Network programming mode is a docking mode between M5 device and UIFlow web programming platform. The screen will show the current network connection status of the device. When the indicator is green, it means that you can receive program push at any time. Under default situation, after the first successful WiFi network configuration, the device will automatically restart and enter the network programming mode

2. Follow the instructions displayed on the device and go to [flow.m5stack.com](http://flow.m5stack.com/) on your computer

3. Type in the matching API Key into UIFlow and select the device type.

![](RackMultipart20210426-4-15d0ali_html_72df91ef0d0db8a1.png)

**Note:** API KEY is the communication credential for M5 devices when using UIFlow web programming. By configuring the corresponding API KEY on the UIFlow side, the program can be pushed for the specific device. The user needs to visit [flow.m5stack.com](http://flow.m5stack.com/) in the computer web browser to enter the UIFlow programming page. Click the setting button in the menu bar at the upper right corner of the page, enter the API Key on the corresponding device, select the hardware used, click OK to save and wait till it prompts successfully connecting or displays connected in the bottom left on the status bar.

**References links for setting up the M5Stack**

1. [https://docs.m5stack.com/en/quick\_start/core2/m5stack\_core2\_get\_started\_MicroPython](https://docs.m5stack.com/en/quick_start/core2/m5stack_core2_get_started_MicroPython)

# **CHAPTER 2: SETTING UP THE DEVICE IN AWS**

**Registering a Thing in AWS**

1. After you create your account in AWS, login to the console and navigate to the AWS IOT Core Dashboard.

2. In the [AWS IoT console](https://console.aws.amazon.com/iot/home), in the side navigation pane, choose  **Manage** , and then choose  **Things**.

3. If a  **You don&#39;t have any things yet**  dialog box is displayed, choose  **Register a thing**. Otherwise, choose  **Create**.

4. On the  **Creating AWS IoT things**  page, choose  **Create a single thing**.

5. On the  **Add your device to the device registry**  page, enter a name for your IoT thing (for this example enter the following as your device name, &quot; **EnvThing&quot;** ), and then choose  **Next**.

You can&#39;t change the name of a thing after you create it. To change a thing&#39;s name, you must create a new thing, give it the new name, and then delete the old thing.

1. On the  **Add a certificate for your thing**  page, choose  **Create certificate**.

2. Choose the  **Download**  links to download the certificate for the thing with the extension \*\*\*\*\*\*.cert.pem, and the private key. We will be using these certificates later.

**Important: This is the only time you can download your certificate and private key.**

1. Choose  **Activate**.

2. Choose  **Attach a policy**.

3. For now, we will not attach any policy and Choose **Register a thing.**

**Reference link for Registering a thing**

1. [https://docs.aws.amazon.com/iot/latest/developerguide/iot-moisture-create-thing.html](https://docs.aws.amazon.com/iot/latest/developerguide/iot-moisture-create-thing.html)

**Creating a policy**

In the project we will create a policy that allows us to connect with AWS, and publish and subscribe to messages.

1. In the [AWS IoT console](https://console.aws.amazon.com/iot/home), in the side navigation pane, choose  **Secure** , and then choose  **Policies**.
2. Enter the following name to your policy &quot;M5ToAWSPolicy&quot;
3. Since we will be writing our own policy in the **Add Statements** section Choose **Advanced Mode** and copy/paste the code give below.
4. Replace \&lt;Region-Name\&gt; with the name of the region in which you are working such as &quot;us-east-1&quot;
5. Replace \&lt;Account-No\&gt; with your AWS account number which can be found on the Top right of the AWS Console by clicking on your account name
6. Replace \&lt;ThingName\&gt; with the name you gave your thing while registering it in the previous step (In our case the name is **EnvThing).**

Code to Copy:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Connect",
      "Resource": "arn:aws:iot:<Region-Name>:<Account-No>:client/<ThingName>"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Subscribe",
      "Resource": "arn:aws:iot: :<Region-Name>:<Account-No>:topicfilter/<ThingName>/sub"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Receive",
      "Resource": "arn:aws:iot: :<Region-Name>:<Account-No>:topic/<ThingName>/sub"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": [
        "arn:aws:iot: :<Region-Name>:<Account-No>:topic/<ThingName>/env/pub"
      ]
    }
  ]
}

```
Example Code:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Connect",
      "Resource": "arn:aws:iot:us-east-1:945484123456:client/EnvThing"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Subscribe",
      "Resource": "arn:aws:iot:us-east-1: 945484123456:topicfilter/EnvThing/sub"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Receive",
      "Resource": "arn:aws:iot:us-east-1: 945484123456:topic/EnvThing/sub"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": [
        "arn:aws:iot:us-east-1: 945484123456:topic/EnvThing/env/pub"
      ]
    }
  ]
}

```
**Attaching the created Policy to the certificate**

1. In the [AWS IoT console](https://console.aws.amazon.com/iot/home), in the side navigation pane, choose  **Secure** , and then choose  **Certificates**.
2. Click on the certificate you just created.
3. Then click on **Actions** and Click on **Attach Policy** from the drop-down menu.
4. Select the policy we just created and click **Attach**.

**References links about AWS Policies**

1. Attach a policy to a client certificate: [https://docs.aws.amazon.com/iot/latest/developerguide/attach-to-cert.html](https://docs.aws.amazon.com/iot/latest/developerguide/attach-to-cert.html)
2. Publish/Subscribe Policy Examples: [https://docs.aws.amazon.com/iot/latest/developerguide/pub-sub-policy.html](https://docs.aws.amazon.com/iot/latest/developerguide/pub-sub-policy.html)

**References**

[**https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers**](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)

[**https://shop.m5stack.com/pages/download**](https://shop.m5stack.com/pages/download)

# **Chapter 3: Programming and Publishing data to AWS**

With the thing registered, certificates downloaded, and the relevant policies attached to the certificate we are now ready to program and publish data to AWS.

**Setting up the coding environment**

1. After connecting the M5Stack to UIFlow in chapter 1 we are now ready program, just hit the **refresh device status** button ![](RackMultipart20210426-4-15d0ali_html_c701b7dd72cf6279.png) in the status bar on the bottom to verify that the device is still connected.

2. Connect the ENV II sensor to the Port A of the M5 Stack using the grove cable provided. (Port A is the port besides the charging port and it&#39;s the only Port the sensor will work on)

3. In the UI Flow screen click on the **+** sign below **Units**

![](RackMultipart20210426-4-15d0ali_html_f78f6ce0bc33f365.png)

![](RackMultipart20210426-4-15d0ali_html_7c75bbb8a965dcba.png)

**Setting up the layout**

1. We will first need 7 labels for this project. These labels can be easily added by dragging and drop the label component from the top left onto the M5Stack screen in the UIFlow editor.

![](RackMultipart20210426-4-15d0ali_html_bb0c545c6fe8318c.png)

The labels we need our as follows:

  1. Label 0 - T:
  2. Label 1 - P:
  3. Label 2 - H:
  4. Label 3 – 23 (Temperature value that will be dynamically populated)
  5. Label 4 – 900 (Pressure value that will be dynamically populated)
  6. Label 5 – 52.73 (Humidity value that will be dynamically populated)
  7. Label 6 - 2021-04-26 16:53:14 Mon (Timestamp value that will be dynamically populated

![](RackMultipart20210426-4-15d0ali_html_5873af015a72198c.png)

**Coding the solution**

1. Setting up connection to AWS

![](RackMultipart20210426-4-15d0ali_html_3bf416fd9cb23ffb.png)

  1. After setup add the AWS connection block and put in the following values as required:
    1. **Init things name:** Add the name with which you had registered your thing in AWS. In our case the name was &quot;EnvThing&quot;
    2. **Host:** This is the endpoint to which you will be connecting to. This can be found by going to the AWS IoT Core dashboard.
      1. In the [AWS IoT console](https://console.aws.amazon.com/iot/home), in the side navigation pane, choose  **Things** , and then click on the name of your thing **(&quot;Envthing&quot;).**
      2. In the side navigation pane of your thing choose **Interact.**
      3. Copy and paste the endpoint given under HTTPS

![](RackMultipart20210426-4-15d0ali_html_f9e304cbd060b5a9.png)
    1. **Port:** Put in 8883 as the port number. (8883 is the default port number for a secure mqtt connection)
    2. **Keepalive:** This is the time for which the connection is to be kept alive. We took a large number like 300 so that the connection is active for a longer period of time.
    3. Hit the plus button to add files. Add the private key and certificate that we had downloaded while registering the thing.
    4. **Keyfile:** From the drop down list select the added private key file.
    5. **certFile:** From the drop down list select the added certificate key file.

  1. Add the AWS Start block

2. Coding the business logic

![](RackMultipart20210426-4-15d0ali_html_4dd925884a556cf5.png)

3. Final code

**Blocky Code**

![](RackMultipart20210426-4-15d0ali_html_cb0c201cfe412d65.png)

**MicroPython Code**
```
**from** m5stack **import** \*
**from** m5stack\_ui **import** \*
**from** uiflow **import** \*
**from** IoTcloud.AWS **import** AWS
**import** json

**import** unit

 screen = M5Screen()
 screen.clean\_screen()
 screen.set\_screen\_bg\_color(0x000000)
 env20 = unit.get(unit.ENV2, unit.PORTA)

 env = None

 label0 = M5Label(&#39;T :&#39;, x=75, y=30, color=0xffffff, font=FONT\_MONT\_48, parent=None)
 label1 = M5Label(&#39;P : &#39;, x=75, y=110, color=0xffffff, font=FONT\_MONT\_48, parent=None)
 label2 = M5Label(&#39;H :&#39;, x=75, y=190, color=0xffffff, font=FONT\_MONT\_48, parent=None)
 label3 = M5Label(&#39;23&#39;, x=150, y=30, color=0xffffff, font=FONT\_MONT\_48, parent=None)
 label4 = M5Label(&#39;900&#39;, x=150, y=110, color=0xffffff, font=FONT\_MONT\_48, parent=None)
 label5 = M5Label(&#39;52.73&#39;, x=150, y=190, color=0xffffff, font=FONT\_MONT\_48, parent=None)
 label6 = M5Label(&#39;2021-04-26 16:53:14 Mon&#39;, x=0, y=0, color=0xffffff, font=FONT\_MONT\_14, parent=None)

 aws = AWS(things\_name=&#39;EnvThing&#39;, host=&#39;a1zerv8ymde1w8-ats.iot.us-east-1.amazonaws.com&#39;, port=8883, keepalive=3000, cert\_file\_path=&quot;/flash/res/890a3328c5-certificate.pem.crt&quot;, private\_key\_path=&quot;/flash/res/890a3328c5-private.pem.key&quot;)
 aws.start()
 rtc.settime(&#39;ntp&#39;, host=&#39;us.pool.ntp.org&#39;, tzone=0)
**while** True:
 label6.set\_text(str(rtc.printRTCtime()))
 label3.set\_text(str(env20.temperature))
 label4.set\_text(str(env20.pressure))
 label5.set\_text(str(env20.humidity))
 env = {&#39;temperature&#39;:(env20.temperature),&#39;pressure&#39;:(env20.pressure),&#39;humidity&#39;:(env20.humidity),&#39;device\_id&#39;:&#39;sensor\_02&#39;,&#39;time&#39;:(rtc.printRTCtime())}
**if** (env20.temperature) \&gt; 24:
 rgb.setColorAll(0xff0000)
**else** :
 rgb.setColorAll(0x33cc00)
 aws.publish(str(&#39;EnvThing/env/pub&#39;),str((json.dumps(env))))
 wait\_ms(2)
 ```

4. Hit the run button on the top right of the UIFlow IDE.

5. You should be able to see the values for temperature, humidity, pressure and the time being constantly updated.

**Checking to see if your device is publishing to AWS over MQTT**

1. In the [AWS IoT console](https://console.aws.amazon.com/iot/home), in the side navigation pane, choose  **Test** , and then choose  **MQTT test client**.

2. In the subscribe to a topic section, enter the topic filter from the code:

&quot;EnvThing/env/pub&quot; and hit Subscribe.

3. You should be able to see data constantly being sent and the output should look like this:

![](RackMultipart20210426-4-15d0ali_html_5bc983eea5a3fa77.png)

We have therefore successfully written our code, ran the code on the M5Stack and published data to AWS over MQTT.
