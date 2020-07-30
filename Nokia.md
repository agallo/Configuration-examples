# Introduction

This document describes the basic steps needed in order to configure a BGP session using TCP-AO to secure the session.  This configuration is applicable to Nokia SR OS 20.5.R1 and newer versions.

Enabling TCP-AO is quite simple and has two steps: 
1. Configure the key
2. Configure the BGP neighbor

More information can be found in the [System Management Guide](https://infocenter.nokia.com/public/7750SR205R1A/topic/com.sr.system.mgmt/html/security.html?cp=21_1_4_10#CHDGCGGD).

Note: The Unix `date --iso-8601=seconds` command output can be used as the begin time, if you want the key to be valid now.

# MD-CLI HMAC-SHA-1-96 Key Configuration

This example create a HMAC-SHA-1-96 key.  Replace `<plain-text>` with the key in plain text.

```
/configure system security keychains keychain "hmac-sha-1-96 keychain" tcp-option-number receive tcp-ao
/configure system security keychains keychain "hmac-sha-1-96 keychain" tcp-option-number send tcp-ao
/configure system security keychains keychain "hmac-sha-1-96 keychain" receive entry 9 authentication-key <plain-text>
/configure system security keychains keychain "hmac-sha-1-96 keychain" receive entry 9 algorithm hmac-sha-1-96
/configure system security keychains keychain "hmac-sha-1-96 keychain" receive entry 9 begin-time 2020-07-29T15:47:03-0400
/configure system security keychains keychain "hmac-sha-1-96 keychain" send entry 2 authentication-key <plain-text>
/configure system security keychains keychain "hmac-sha-1-96 keychain" send entry 2 algorithm hmac-sha-1-96
/configure system security keychains keychain "hmac-sha-1-96 keychain" send entry 2 begin-time 2020-07-29T15:47:03-0400
```

# MD-CLI AES-128-CMAC-96 Key Configuration

This example create a AES-128-CMAC-96 key.  Replace `<plain-text>` with the key in plain text.

```
/configure system security keychains keychain "aes-128-cmac-96 keychain" tcp-option-number receive tcp-ao
/configure system security keychains keychain "aes-128-cmac-96 keychain" tcp-option-number send tcp-ao
/configure system security keychains keychain "aes-128-cmac-96 keychain" receive entry 9 authentication-key <plain-text>
/configure system security keychains keychain "aes-128-cmac-96 keychain" receive entry 9 algorithm aes-128-cmac-96
/configure system security keychains keychain "aes-128-cmac-96 keychain" receive entry 9 begin-time 2020-07-29T15:47:03-0400
/configure system security keychains keychain "aes-128-cmac-96 keychain" send entry 2 authentication-key <plain-text>
/configure system security keychains keychain "aes-128-cmac-96 keychain" send entry 2 algorithm aes-128-cmac-96
/configure system security keychains keychain "aes-128-cmac-96 keychain" send entry 2 begin-time 2020-07-29T15:47:03-0400
```

# MD-CLI BGP Neighbor Configuration

Next, add the key to the BGP session.  This step is the same for IPv4 and IPv6 neighbors, and could also be added to the group instead.

```
/configure router "Base" bgp neighbor <ip-address> authentication-keychain <keychain>
```

# Show Command Output

The following show command displays the configured BGP neighbor keychains.

```
[]
A:grhankin@er1-nyc# show router bgp auth-keychain "interoptest-aes"

===============================================================================
Sessions using key chain: interoptest-aes
===============================================================================
Group
    Peer address                        KeyChain name
-------------------------------------------------------------------------------
Juniper TCP-AO Test
    116.197.187.117                     interoptest-aes
Juniper TCP-AO Test
    2403:8100:1002:103::111             interoptest-aes
===============================================================================
```

# Complete Configuration Example and Output from June 2020 Interoperability Test with Juniper

```
/configure system security keychains keychain "interoptest" tcp-option-number receive tcp-ao
/configure system security keychains keychain "interoptest" tcp-option-number send tcp-ao
/configure system security keychains keychain "interoptest" receive entry 9 authentication-key "yzClLKIFsAVR91AobUXUT/ppPzL7bVxBrNNg" hash
/configure system security keychains keychain "interoptest" receive entry 9 algorithm hmac-sha-1-96
/configure system security keychains keychain "interoptest" receive entry 9 begin-time 2020-01-06T17:35:59.0-05:00
/configure system security keychains keychain "interoptest" send entry 2 authentication-key "yzClLKIFsAVR91AobUXUT/ppPzL7bVxBrNNg" hash
/configure system security keychains keychain "interoptest" send entry 2 algorithm hmac-sha-1-96
/configure system security keychains keychain "interoptest" send entry 2 begin-time 2020-01-06T17:35:59.0-05:00
/configure system security keychains keychain "interoptest-aes" tcp-option-number receive tcp-ao
/configure system security keychains keychain "interoptest-aes" tcp-option-number send tcp-ao
/configure system security keychains keychain "interoptest-aes" receive entry 9 authentication-key "yzClLKIFsAVR91AobUXUT/ppPzL7bVxBrNNg" hash
/configure system security keychains keychain "interoptest-aes" receive entry 9 algorithm aes-128-cmac-96
/configure system security keychains keychain "interoptest-aes" receive entry 9 begin-time 2020-06-09T00:00:00.0-04:00
/configure system security keychains keychain "interoptest-aes" send entry 2 authentication-key "yzClLKIFsAVR91AobUXUT/ppPzL7bVxBrNNg" hash
/configure system security keychains keychain "interoptest-aes" send entry 2 algorithm aes-128-cmac-96
/configure system security keychains keychain "interoptest-aes" send entry 2 begin-time 2020-06-09T00:00:00.0-04:00

/configure policy-options policy-statement "RP_EXPORT_10458" entry 10 from protocol name [direct]
/configure policy-options policy-statement "RP_EXPORT_10458" entry 10 to protocol name [bgp]

/configure router "Base" bgp neighbor "116.197.187.117" authentication-keychain "interoptest-aes"
/configure router "Base" bgp neighbor "116.197.187.117" group "Juniper TCP-AO Test"
/configure router "Base" bgp neighbor "116.197.187.117" multihop 255
/configure router "Base" bgp neighbor "116.197.187.117" local-address 124.252.255.66
/configure router "Base" bgp neighbor "116.197.187.117" peer-as 10458
/configure router "Base" bgp neighbor "116.197.187.117" family ipv4 true
/configure router "Base" bgp neighbor "116.197.187.117" export policy ["RP_EXPORT_10458"]
/configure router "Base" bgp neighbor "2403:8100:1002:103::111" authentication-keychain "interoptest-aes"
/configure router "Base" bgp neighbor "2403:8100:1002:103::111" group "Juniper TCP-AO Test"
/configure router "Base" bgp neighbor "2403:8100:1002:103::111" multihop 255
/configure router "Base" bgp neighbor "2403:8100:1002:103::111" local-address 2406:c800:e000:1::2
/configure router "Base" bgp neighbor "2403:8100:1002:103::111" peer-as 10458
/configure router "Base" bgp neighbor "2403:8100:1002:103::111" family ipv6 true
/configure router "Base" bgp neighbor "2403:8100:1002:103::111" export policy ["RP_EXPORT_10458"]

[]
A:grhankin@er1-nyc# show router bgp summary all

===============================================================================
BGP Summary
===============================================================================
Legend : D - Dynamic Neighbor
===============================================================================
Neighbor
Description
ServiceId          AS PktRcvd InQ  Up/Down   State|Rcv/Act/Sent (Addr Family)
                      PktSent OutQ
-------------------------------------------------------------------------------
116.197.187.117
Def. Instance  10458     4784    0 01d11h59m 1/0/3 (IPv4)
                         4323    0
2403:8100:1002:103::111
Def. Instance  10458     3046    0 22h54m14s 0/0/2 (IPv6)
                         2754    0

-------------------------------------------------------------------------------
```