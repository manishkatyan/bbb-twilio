# bbb-twilio
Integrate Twilio into BigBlueButton so that users can join a meeting with a dial-in number.

The built-in WebRTC-based audio in BigBlueButton is very high quality audio. Still, there may be cases where you want students to be able to dial into the conference bridge using a telephone number.

For example, some students might have poor connectivity and may face cracking or delayed audio during an online class. Or, some online classes are just lectures that students just need to listen to.

If you have been running BigBlueButton servers for a while, you might have received complains from your users about poor audio quality.  

Well, integrating a phone number into your online classes is a great way to mitigate such audio issues. 

# Twilio + BigBlueButton
We are going to get a phone number from [Twilio](https://www.twilio.com/) and configure FreeSWITCH accordingly to receive incoming calls via session initiation protocol (SIP) fromTwilio.

Why Twilio? You get a free trial account to test the full functionality with a free dial-in phone number. Documentation is extensive, logs are detailed and it just works! Use my [referral link to get $10 credit when you upgrade](https://www.twilio.com/referral/VfQyDw).

# Setting up Your Twilio Elastic SIP Trunk

Login to your Twilio account. 

I am going to list-out key steps that you need to perform:
1. Termination URL: Give a unique identifier (example - your project name) to identify your Termination SIP URI. You will use this URI in setting-up FreesSWITCH later. 
2. Originating URI: Give the public IP of your BigBlueButton server in the following format: sip:BBB_PUBLIC_IP
3. IP Access Control Lists: Go to Elastic SIP Trunking > Authentication > IP / CIDR Access Control Lists to add public IP your BigBlueButton server. You can add your public IP in the following format: BBB_PUBLIC_IP / 32
4.  Credential Lists: Add a creential. Enter a name and use your Twilio Username and Password.


That's all you would need to setup a phone number for your BigBlueButton classes. For additional information, check-out getting started guide on [Elastic SIP Trunking](https://www.twilio.com/docs/sip-trunking).

# Configure your FreeSWITCH
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

**Update Firewall Rule**

I am assuming 5060 is the default SIP port. We need to open this port for UDP connections.
```sh
$sudo ufw allow 5060/udp
```

You can verify external_sip_port in /opt/freeswitch/conf/vars.xml

