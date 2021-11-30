## Connect FreeSWITCH to Tata SIP

### Create SIP Profile

#### Change ALL-CAPS values in sip_profile_tata.xml

```sh
cp sip_profile_tata.xml /opt/freeswitch/conf/sip_profiles/external/tata.xml
```

### Create a Dialplan

```sh
cp dialplan_twilio.xml /opt/freeswitch/conf/dialplan/public/twilio.xml
```

#### Please replace `<STD-CODE>` and `<EXTERNAL_DID>` with your STD code and DID number
