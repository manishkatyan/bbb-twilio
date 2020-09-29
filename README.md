# bbb-twilio
Integrate Twilio into BigBlueButton so that users can join a meeting with a dial-in number.

The built-in WebRTC-based audio in BigBlueButton is very high quality audio. Still, there may be cases where you want students to be able to dial into the conference bridge using a telephone number.

For example, some students might have poor connectivity and may face cracking or delayed audio during an online class. Or, some online classes are just lectures that students just need to listen to.

If you have been running BigBlueButton servers for a while, you might have received complains from your users about poor audio quality.  

Well, integrating a phone number into your online classes is a great way to mitigate such audio issues. 

# Twilio + BigBlueButton
We are going to get a phone number from [Twilio](https://www.twilio.com/) and configure FreeSWITCH accordingly to receive incoming calls via session initiation protocol (SIP) fromTwilio.

Why Twilio? You get a free trial account to test the full functionality with a free dial-in phone number. Documentation is extensive, logs are detailed and it just works! Use my [referral link to get $10 credit when you upgrade](www.twilio.com/referral/VfQyDw).  
