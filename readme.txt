How to work MQTTSN Telemed:

NOTE: All of these programs, besides the publish-queue directory, are publically available code gotten off of github. publish-queue contains existing, publically available code but has been altered in order to build a queue for messages when connectivity is down. I've included them here to make things easier for you, but I would advise that for a finilized system it's recommended that these repositories are gotten directly from github in order to obtain the latest code. These worked for me but this is a far from perfect program.

1. RSMB
Navigate to the org.eclipse.mosquitto.rsmb/rsmb/src directory.
Type "make" on a Linux based computer (Mac mini was used when I set it up).
If there are errors (mine had a few) pertaining to IPV6_ADD_MEMBERSHIP member, add the following code above the function declarations in Socket.c:

#ifndef IPV6_ADD_MEMBERSHIP
#ifndef IPV6_JOIN_MEMBERSHIP
#define IPV6_ADD_MEMBERSHIP IPV6_JOIN_GROUP
#define IPV6_DROP_MEMBERSHIP IPV6_LEAVE_GROUP
#endif
#endif


This should solve any issues compiling the code on a Mac system.

After the source has been compiled, run the broker as usual, by typing in the location of the executable, which in this case would be:
~/MQTTSN-Telemed/org.eclipse.mosquitto.rsmb/rsmb/src/broker_mqtts

It should be noted that both the "broker_mqtts" and the "broker" executable accomplish the same task, so neither one is required over the other. This will run the broker on the current machine, usually tying it to IP address 0.0.0.0 and listens to port 1883. If the broker fails to run, make sure there isn't an already existing broker or program listening on that port. If there is, obviously kill it and try to run the broker again.


2. ruby-em-mqtts
This program contains the MQTTS gateway needed to create the MQTT-SN connection over UDP. From any location type:
sudo gem install em-mqtts
This will install the gateway as a ruby gem on the system. I had no problem doing this, as long as ruby was installed, but I know that installing a third party unverified program as a gem on the system isn't exactly the safest thing in the world. However you want to install it, just make sure the following terminal command works:
em-mqtts-gateway -A [IPADDRESS OF BROKER]
If the broker is on the same machine then there is no need to include the -A argument. I would advise intalling this gem on the same machine as the broker, but more on that later.

3. MQTTSN Tools
Navigate to the mqtt-sn-tools director and type "make" to compile and create the executables. I would recommend creating these executables on the same machine as the gateway and the broker. From this folder you can run the subscription client by typing:
/MQTTSN-Telemed/mqtt-sn-tools/sqtt-sn-sub -i "SUBSCRIBER" -k 3000 -t hello/world
The -i assigns a client id, for easy debugging on the broker, -k is a kill timer, and -t is the topic. Once you get your hands deeper into the program this topic obviously should change, but it's currently hard coded into the publisher client so for now this will work just fine.
Once this command is typed you should see both the broker and the gateway register its existance.

4. mqttSN_pub
NOTE: This is the code that I would install on a seperate machine. That way the UDP packets can be easier to track and document. It'll work just fine on the same computer but it sort of isn't the point of our program.
Navigate to the publish-queue directory and open the mqttSN_pub.c file. Line 20 contains this code:
const char *mqtt_sn_host = "10.255.61.51";
This is the IP address of the gateway, so change it to the appropriate address.
Save the file, type "make" to create the executable, and when the rest of the system is set up run the executable as before.
This program has a five second timer, and ever five seconds will publish a message (which is just an incrementing number as a string) to the gateway. The gateway will take the message and pass it to the broker as a TCP packet. The broker will then hand that packet back to the gateway (since it's on the same machine) which it pass it to the MQTTSN Subscriber, to be shown on the command line. If you disconnect the publisher from the internet, the program will queue the messaages while there is no connection. After a few messages are queued feel free to reconnect anytime and the messages will be published in the order that they were queued.


I want to say that this is a very imperfect system and program. It doesn't have any ability other than the blind publishing and queueing of messages. But take this as a start to what we need to do, as all the code is there to very easily create what we is needed.
