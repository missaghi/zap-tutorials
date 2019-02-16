# Zap iOS via OpenVPN lnd node
Running Zap on iOS connected to remote LND on windows via OpenVPN

So you got your lightning node running at home and you want to connect your phone wallet to it without having to open a port to your node. With OpenVPN running on your node you can have your phone appear on it's network at all times. This will let your phone connect to the server as if it were on the same network even when you leave the house, all without exposing your RPC port to the public internet. I think OpenVPN has some paid service that will mediate the connections so that you dont have to expose a port for the VPN server but this tutorial is intended for folks that want to run their own VPN for free.

Setting up OpenVPN seemed much harder than it should have for such a fundamental tool. I have a feeling the information is actively obfuscated so that barrier to entry will persuade you to go with one of the fee based VPN providers, aka trusted third parties. I'll be working on an OpenVPN installer to make this easier one day.

[![Preview](https://img.youtube.com/vi/ra8-WnOhoVM/1.jpg)](https://youtu.be/ra8-WnOhoVM)

Here is the stack:

Windows Server: (static-ish IP or DynDNS)
- bitcoin full node
- lnd (lighting node)
- OpenVPN 

iOS iPhone
- OpenVPN connect
- [Zap](https://github.com/LN-Zap/zap-iOS)

## Steps:
0. Run [Pierre's Lightning Node Launcher](https://medium.com/lightning-power-users/easy-lightning-with-node-launcher-zap-488133edfbd) to setup bitcoin node and lighting node.
1. Setup VPN between your node and phone (openVPN with static IP or DynDNS or mediated like Hamachi)
2. Configure LND for listening on your network adapter IP
3. Create Zap connection text using LNDConnect or Node launcher's "Show QR" button when it's ready"
4. Test it out by sending a tip: https://tippin.me/@missaghi

### Step 0: [Run the node launcher](https://medium.com/lightning-power-users/easy-lightning-with-node-launcher-zap-488133edfbd)

### Step 1: OpenVPN
[Setting up OpenVPN](https://www.reddit.com/r/OpenVPN/comments/81q2q6/guide_how_to_set_up_openvpn_server_on_windows_10/)
* I forgot to click on the EasyRSA button so I didn't get the scripts but you can also get EasyRSA from [github repo](https://github.com/OpenVPN/easy-rsa/releases) and when you run .\EasyRSA-Start.bat it will give you shell where you can type ./easyrsa and run similar scripts.

After you set the server up you need to run OpenVPN connect on your phone, email the config file to yourself and open in the ios default mail app (gmail app didn't handle the attachment correctly).

### Step 2.1: Configure LND.conf
In the node launcher's advanced page there is a link to the lnd.conf file. Open the file and add these lines for each IP you need:

```
externalip=yourNetworkAdaperIPaddress
tlsextraip=yourNetworkAdaperIPaddress
restlisten=yourNetworkAdaperIPaddress:8080
rpclisten=yourNetworkAdaperIPaddress:10009
```

Note that you don't need to put your public IP here becasue on the VPN your phone will address the local IP of your servers network adapter.

## Step 2.2: Recreate tls.key and tls.cert

In the smae folder as lnd.conf you can delete the file tls.cert and tls.key, then restart LND, this will let LND create a new cert with "subject alt names" that include the IP addresses that you added.

## Step 3: Configure Zap
In order for Zap to work it needs the URL, certificate, and macaroon from LND. The cert enables a TLS connection, the Macaroon is the credentials to control the node. There are two ways to get this info into ZAP, one is by pasting the connection string scanning a QR code representaion of it. 

### Step 3 Option 1, lndconnect to generate QR and URI
You can creat the QR code by using [LNDConnect](https://github.com/LN-Zap/lndconnect) which will generate the text and QR code. I tried to install go and run the first command in the instructions "go get -d github.com/LN-Zap/lndconnect" but nothing happened... if you know go language better then this would be the most future proof option.

### Step 3 Option 2, Run node script to generate URI
The other option is to run a node script that builds the URI specified [here](https://github.com/LN-Zap/lndconnect/blob/master/lnd_connect_uri.md), but this may not be backwards compatible.

1. install [node.js](https://nodejs.org/en/download/)
2. install [VSCode](https://code.visualstudio.com/download)
3. Create a file called app.js, paste [this code sample](https://gist.github.com/missaghi/342929aa8adb0503a1e4c4eca77db0b2) into it.
4. Run without debugging (you may have to correct the path to your macaroon and cert, check the node launcher for those.)
5. open the file created in the same folder ass app.js called lnd.txt and paste the contents into the app.

## Step 4: Hope and pray
So the first time i set this up I kept getting TLS handshake erros but then an hour later it worked, probably cache, haven't been able to recreate but now you knoew... I think LND needs some tweaks to it's cert generation to work better with externalIP which i think is coming in the next version (see this issue: https://github.com/lightningnetwork/lnd/issues/684)

**Some other helpful tutorials:**
- https://ln-zap.github.io/zap-tutorials/
