# Zap iOS via OpenVPN lnd node
Running Zap on iOS connected to remote LND on windows via OpenVPN or Hamachi 

If you would like to have a lightening wallet on your phone that connects to your own node running on a server but you don't want to make your server public then you can run a VPN server to put both devices on the same vitual network that's hidden from the outside world. I'm new to writing tutorial, I hope you find this helpful, but if you have suggestions or questions please let me know.

Setting up OpenVPN seemed much harder than it should have for such a fundamental tool. I have a feeling the information is actively obfuscated so that barrier to entry will persuade you to go with one of the fee based VPN providers, aka third parties.

[![Preview](https://img.youtube.com/vi/ra8-WnOhoVM/1.jpg)](https://youtu.be/ra8-WnOhoVM)

Here is the stack:

Windows Server: (static-ish IP or DynDNS)
- bitcoin full node
- lnd (lighting node)
- OpenVPN 

iOS iPhone
- OpenVPN connect
- [Zap](https://github.com/LN-Zap/zap-iOS)

###Steps:
0. Run [Pierre's Lightning Node Launcher](https://medium.com/lightning-power-users/easy-lightning-with-node-launcher-zap-488133edfbd) to setup bitcoin node and lighting node.
1. Setup VPN between your node and phone (openVPN with static IP or DynDNS or mediated like Hamachi)
2. Configure LND for externalIP
3. Create Zap connection text using LNDConnect or Node launcher's "Show QR" button when it's ready"
4. Test it out with this invoice (sends me a coffee): lightning:lnbc1m1pw97ykwpp5jry9xzz9qxens0e9va72p6ek0j78wmupcrxy4zcgn0l37g650ursdpg2djhgatsypdxzupqv9hxggzvfezzqen0wgs9v5zwcqzysnqrz3q0km07rk6mev5u07dj4fcp99us0gafl2r2pq2u9hp064vtssy93mzyl547l7z60xp8xmmn8ql8vgfcvcpyhmu6xsc3ukyds8jgpuhq4p4

####Step 0: [Run the node launcher](https://medium.com/lightning-power-users/easy-lightning-with-node-launcher-zap-488133edfbd)

####Step 1: 
####Option 1: OpenVPN
[Setting up OpenVPN](https://www.reddit.com/r/OpenVPN/comments/81q2q6/guide_how_to_set_up_openvpn_server_on_windows_10/)
* I forgot to click on the EasyRSA button so I didn't get the scripts but you can also get EasyRSA from [github repo](https://github.com/OpenVPN/easy-rsa/releases) and when you run .\EasyRSA-Start.bat it will give you shell where you can type ./easyrsa and run similar scripts.

After you set the server up you need to run OpenVPN connect on your phone, email the config file to yourself and open in the ios default mail app (gmail app didn't handle the attachment correctly).

#####Option 2: Hamachi
You can install Hamachi from [LogMeIn](https://www.vpn.net)
After you create a new network you can go into the web based manager and invite a new client which will send an email to your phone with the VPN profile.

####Step 2.1: Configure LND.conf
In the node launcher's advanced page there is a link to the lnd.conf file. Open the file and add these lines for each IP you need:

> externalip=25.8.179.146 
> tlsextraip=25.8.179.146
> restlisten=25.8.179.146:8080 
> rpclisten=25.8.179.146:10009

For hamachi you will need whatever the IP address is for the computer you are running both hamachi and the LND node. To-do add image

####Step 2.2: Recreate tls.key and tls.cert

In the smae folder as lnd.conf you can delete the file tls.cert and tls.key, then restart LND, this will let LND create a new cert with "subject alt names" that include the IP addresses that you added.

####Step 3: Configure Zap
In order for Zap to work it needs the URL, certificate, and macaroon from LND. The cert enables a TLS connection, the Macaroon is the credentials to control the node. There are two ways to get this info into ZAP, one is with a QR code that the node launcher will be showing soon. The second is to run the node script from LNDConnect which will generate a text which you can paste into the app.

To run this script I found the easiest way was to:
1. install [node.js](https://nodejs.org/en/download/)
2. install [VSCode](https://code.visualstudio.com/download)
3. Create a file called app.js, paste [this code sample](https://gist.github.com/missaghi/342929aa8adb0503a1e4c4eca77db0b2) into it.
4. Run without debugging (you may have to correct the path to your macaroon and cert, check the node launcher for those.)
5. open the file created in the same folder ass app.js called lnd.txt and paste the contents into the app.

####Step 4: Hope and pray
So the first time i set this up I kept getting TLS handshake erros and Zap wouldn't connect on hamachi but it would on OpenVPN, but then an hour later it worked on both... I think LND needs some tweaks to it's cert generation to work better with externalIP which i think is comeing in the next version (see this issue: https://github.com/lightningnetwork/lnd/issues/684)

**Some other helpful tutorials:**
- https://ln-zap.github.io/zap-tutorials/iOS-remote-node-setup.html
- 
