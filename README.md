# bbb-twilio
Integrate Twilio into BigBlueButton so that users can join a meeting with a dial-in number.

The built-in WebRTC-based audio in BigBlueButton is very high quality audio. Still, there may be cases where you want students to be able to dial into the conference bridge using a telephone number.

For example, some students might have poor connectivity and may face cracking or delayed audio during an online class. Or, some online classes are just lectures that students just need to listen to.

If you have been running BigBlueButton servers for a while, you might have received complains from your users about poor audio quality.  

Well, integrating a phone number into your online classes is a great way to mitigate such audio issues. 

## Twilio + BigBlueButton
We are going to get a phone number from [Twilio](https://www.twilio.com/) and configure FreeSWITCH accordingly to receive incoming calls via session initiation protocol (SIP) fromTwilio.

Why Twilio? You get a free trial account to test the full functionality with a free dial-in phone number. Documentation is extensive, logs are detailed and it just works! Use my [referral link to get $10 credit when you upgrade](https://www.twilio.com/referral/VfQyDw).

## Setting up Your Twilio Elastic SIP Trunk

Login to your Twilio account and navigate to the Dashboard to [Elastic SIP Trunking](https://www.twilio.com/user/account/sip-trunking) by clicking the dropdown menu in the left side. Click on [Trunks](https://www.twilio.com/console/sip-trunking/trunks) to provision an Elastic SIP Trunk. 

I am going to list-out key steps that you need to perform:
1. Termination URL: Give a unique identifier (example - your project name) to identify your Termination SIP URI. You will use this URI to set-up FreesSWITCH later. 
2. Originating URI: Give the public IP of your BigBlueButton server in the following format: sip:BBB_PUBLIC_IP
3. IP Access Control Lists: Go to Elastic SIP Trunking > Authentication > IP / CIDR Access Control Lists to add public IP your BigBlueButton server. You can add your public IP in the following format: BBB_PUBLIC_IP / 32
4.  Credential Lists: Add a creential. Enter a name and use your Twilio Username and Password.


That's all you would need to setup a phone number for your BigBlueButton classes. For additional information, check-out getting started guide on [Elastic SIP Trunking](https://www.twilio.com/docs/sip-trunking).

## Configure your FreeSWITCH
I am assuming you have a standard [BigBlueButton installation](https://github.com/bigbluebutton/bbb-install), where FreeSWITCH is running on the same machine as your BigBlueButton server. 

Edit /opt/freeswitch/conf/sip_profiles/external.xml and change
```xml
<param name="ext-rtp-ip" value="$${local_ip_v4}"/>
<param name="ext-sip-ip" value="$${local_ip_v4}"/>
```

To 
```xml
<param name="ext-rtp-ip" value="$${external_rtp_ip}"/>
<param name="ext-sip-ip" value="$${external_sip_ip}"/>
```

Edit /opt/freeswitch/conf/vars.xml, and change
```xml
<X-PRE-PROCESS cmd="set" data="external_rtp_ip=stun:stun.freeswitch.org"/>
<X-PRE-PROCESS cmd="set" data="external_sip_ip=stun:stun.freeswitch.org"/>
```

To

```xml
<X-PRE-PROCESS cmd="set" data="external_rtp_ip=BBB_PUBLIC_IP"/>
<X-PRE-PROCESS cmd="set" data="external_sip_ip=BBB_PUBLIC_IP"/>
```

### Update Firewall Rule

I am assuming 5060 is the default SIP port. We need to open this port for UDP connections.
```sh
sudo ufw allow 5060/udp
```

You can verify external-sip-port in `/opt/freeswitch/conf/vars.xml`

In your Twilio account, check Networking Info section under Elastic SIP Trunking. You will find Twilio IP addresses that you should whitelist for post 5060/UDP. With these rules, you won’t get spammed by bots scanning for SIP endpoints and trying to connect. 

## Connect FreeSWITCH to Twilio

### Create SIP Profile


```sh
cp sip_profile_twilio.xml /opt/freeswitch/conf/sip_profiles/external/twilio.xml
```

Download the sample SIP profile, sip_profile_twilio.xml and make the following changes:
1. Set gateway name to the Termination URL that you created above while [setting up Twilio Elastic SIP Trunk](https://github.com/manishkatyan/bbb-twilio#setting-up-your-twilio-elastic-sip-trunk). The gateway name would have the following format: my-project-name.pstn.twilio.com.
2. Set proxy the same as the gateway name.
3. Set username and password the same as your Twilio username and password. Check [Credential Lists](https://github.com/manishkatyan/bbb-twilio#setting-up-your-twilio-elastic-sip-trunk)  

After making above changes, copy SIP profile to /opt/freeswitch/conf/sip_profiles/external/

### Create a Dialplan

```sh
cp dialplan_twilio.xml /opt/freeswitch/conf/dialplan/public/twilio.xml
```

To route the incoming call from Twilio to the correct BigBlueButton audio conference, you need to create a dialplan which, for FreeSWITCH, is a set of instructions that it runs when receiving an incoming call. 

When a user calls the phone number, the dialplan will prompt the user to enter a five digit number associated with the conference.

Value of field `destination_number` is customized for Twilio so that it will accept
- the dial-in number format startig with either + or 00
- single digit nnumber between 1 and 9 for USA number. This can be changed to [1-9]{2} for France (33) or India (91)
- 10 digit number between 0 and 9 for USA number. This can be changed to [0-9]{x}, where x = number of remaining digits in the dial number    

Change ownership of this file to freeswitch:daemon
```sh
chown freeswitch:daemon /opt/freeswitch/conf/dialplan/public/twilio.xml
```

and then restart FreeSWITCH:
```sh
sudo systemctl restart freeswitch
```

Try calling the phone number. It should connect to FreeSWITCH and you should hear a voice prompting you to enter the five digit PIN number for the conference. Please note, that dialin will currently only work if at least one web participant has joined with their microphone.

To always show users the phone number along with the 5-digit PIN number within BigBlueButton, edit `/usr/share/bbb-web/WEB-INF/classes/bigbluebutton.properties` and change `613-555-1234` to the phone number provided by Twilio
```sh
defaultDialAccessNumber=613-555-1234
```

and change defaultWelcomeMessageFooter to
```sh
defaultWelcomeMessageFooter=<br><br>To join this meeting by phone, dial:<br>  %%DIALNUM%%<br>Then enter %%CONFNUM%% as the conference PIN number.
```

Save `bigbluebutton.properties` and restart BigBlueButton again. 

Each user that joins a session will see a message in the chat similar to.
```sh
To join this meeting by phone, dial:
   613-555-1234
and enter 12345 as the conference PIN number.
```

## [BigBlueButton Tech Support](https://higheredlab.com/bigbluebutton-support/)
### Are you facing difficulties with your BigBlueButton server?
Lean on our expertise to smoothly run your BigBlueButton server. We can: 
1. troubleshoot your BigBlueButton servers to improve audio and video performance,
1. install additional features such as streaming, mp4 recordings and attendance, and
1. provide managed BigBlueButton servers, starting at $12 per month

[Click here to learn more](https://higheredlab.com/bigbluebutton-support/)

## More on BigBlueButton

Check-out the following apps to further extend features of BBB.

### [bbb-jamboard](https://github.com/manishkatyan/bbb-jamboard)

The default whiteboard of BigBlueButton has limited features including no eraser. Many teachers wish to have a more features-rich whiteboard that would help them better in conducting online classes.

With BBB-Jamboard, you can easily integrate Google Jamboard into your BigBlueButton server.

Jamboard is a digital interactive whiteboard developed by Google and can be used in stead of the default BugBlueButton whiteboard. Google Jamboard has the eraser feature that has often been requested by BigBlueButton users.

### [bbb-mp4](https://github.com/manishkatyan/bbb-mp4)

With this app, you can convert a BigBlueButton recording into MP4 video and upload to S3. You can covert multiple MP4 videos in parallel or automate the conversion process.

### [bbb-admin](https://github.com/manishkatyan/bbb-admin)

Scripts for BigBlueButton admins including extracting IP of users joining, participants attendance, poll answers and many other analytics. 

### [bbb-optimize](https://github.com/manishkatyan/bbb-optimize)

Better audio quality, increase recording processing speed, dynamic video profile, pagination, fix 1007/1020 errors and use apply-config.sh to manage your customizations are some key techniques for you to optimize and smoothly run your BigBlueButton servers.

### [bbb-streaming](https://github.com/manishkatyan/bbb-streaming)
Livestream your BigBlueButton classes on Youtube or Facebook to thousands of your users.


### [bbb-recording-server](https://github.com/manishkatyan/bbb-recording-server)

With this app, you can process BigBlueButton recordings on a separate server, called BBB Recording Server. Separation of recordings from BigBlueButton (client) improves performance as all server resources are dedicated towards conducting live classes.

### [100 Most Googled Questions on BigBlueButton](https://higheredlab.com/bigbluebutton-guide/)

Everything you need to know about BigBlueButton including pricing, comparison with Zoom, Moodle integrations, scaling, and dozens of troubleshooting.


