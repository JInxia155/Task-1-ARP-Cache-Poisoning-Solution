Download Link : https://programming.engineering/product/task-1-arp-cache-poisoning-solution/

# Task-1-ARP-Cache-Poisoning-Solution
Task 1: ARP Cache Poisoning Solution
ARP cache poisoning, also known as ARP spoofing, is a type of network attack that involves an attacker sending fake Address Resolution Protocol (ARP) messages to a local area network. The goal of the attacker is to associate their own MAC address with the IP address of another device on the network, allowing them to intercept and modify network traffic.

Task 1 action items:

    A method: constructing an ARP request packet on host ‘Attacker’ and sending it to host ‘Client’.

    B method: constructing an ARP reply packet on host ‘Attacker’ and sending it to host ‘Client’.

    C method: construct an ARP gratuitous packet (special ARP request packet) on host ‘Attacker’.

The goal of each action item is to map ‘Attacker’s’ MAC address to ‘Server’s’ IP address in ‘Client’s’ ARP cache.

For executing the task we are creating a demo network, consisting of three virtual machines. All three machines are on the same LAN. Graph 1: (both graph and IP list are relevant for all the other tasks)

The IP / MAC addresses for each machine are:

    Client 10.0.2.10 / 08002776D632

    Server 10.0.2.12 / 080027EB6E8A

    Attacker 10.0.2.14 / 0800277BBAD7

Lastly, before starting the attacks described in the action items, we retrieve the current state of the ARP table from the Client’s machine, for the ability to compare the results later on.

Task-1A (using ARP request)

    On Attacker machine we create the code of the ARP request, using scapy:

The script is creating two layers to construct an arp packet and to send it via sedp() to the link layer.

We set the source ip address to be that of the server with the mac source of the attacker.

    Then make the file executable:

    Run the script:

    Check status of arp table on Client machine:

As desired we can see that we mapped the server’s IP address to the attacker’s mac address.

Task-1B (using ARP reply)

● Resetting to default state Client’s arp table:

    Similarly to the previous task, creating python script but this time with additional parameter in ARP function:

op stands for operation, and 2 defines the operation to be “reply”.

● Making the file executable and sending the packet:

    hwlen represents the length of the hardware (MAC) address in bytes.

    plen is protocol length

        When checking the arp table on client’s machine we see the MAC address updated:

Task-1C (using ARP gratuitous packet )

● Again resetting the table on the Client’s machine:

● Creating the python script:

An ARP gratuitous packet is a type of ARP packet that is sent by a host on a network to announce its presence and/or to update its mapping in the ARP cache of other hosts on the same network. Unlike a regular ARP request or response, a gratuitous ARP packet is not triggered by an incoming packet or by the cache timeout, but is instead sent proactively by a host

To create this packet we changed the destination mac address to ff:ff:ff:ff:ff:ff which means it will be sent broadcast.

● Checking on Client’s machine:

We can see that the cache poisoning was successful.

Summary

All three methods were successful, as confirmed by checking the ARP table on the client machine after each attempt. Our expectations for the tasks were met, as the tests were conducted on machines without significant security defences, and the behaviour observed matched that of ARP packets. Through the process, we learned how to construct ARP packets with Scapy and define their layers. However, we did encounter

some challenges, such as initially writing incorrect code, which we resolved by delving deeper into the documentation. In Task-1B, we initially did not achieve the desired result as the client’s ARP table did not contain a row with the server IP, meaning that the reply did not update anything. We discovered that the IP entry must already be in the table for the attack to succeed. Aside from these challenges, the tasks went smoothly and matched our expectations.

Task 2: MITM Attack on Telnet using ARP Cache Poisoning

MITM stands for “Man-in-the-Middle” attack. It’s a type of cyber attack where an attacker intercepts communication between two parties, allowing them to eavesdrop, modify or even impersonate one or both parties.

In the case of Telnet, which is an unencrypted protocol, an attacker can use ARP cache poisoning to intercept traffic between a Telnet client and server. By spoofing the MAC address of the server, the attacker can redirect traffic intended for the server to their own machine, allowing them to intercept and modify Telnet traffic without being detected. This can lead to the theft of login credentials and sensitive information.

Task 2 action items:

    Step 1: Launch the ARP cache poisoning attack on both Client and Server. Attacker should spoof Client’s IP on the server and vice versa on the Client’s machine.

    Step 2: Testing, sending ping between hosts

    Step 3: Turn on IP forwarding on Attackers machine

    Step 4: Launch the MITM attack

The goal is to use arp cache poisoning in a way the Attacker machine will get control of Clients and Servers communication on telnet.

Task 2 – step 1

● Based on the previous tasks, creating the necessary script file:

The method chosen for arp poisoning was sending an ARP request, to the Client with spoofed Servers IP and to the Server with spoofed Clients IP.

● Making the file executable and sending the packets:

We can see that the poisoning was successful and the Servers IP mapped to Attackers MAC address.

● Checking on Servers arp table:

Similarly we can see that the Client’s IP mapped to Attackers Mac here as well.

Task 2 – step 2

    For testing, sending ping from Server to Client:

    Following the send we can see this activity in Servers wireshark:

We can see ARP broadcast packets after the ping and no reply from the Client’s IP.

    The Client’s wireshark:

We are able to see the ping request but no reply is sent.

Task 2 – step 3

    Enabling IP forwarding on Attackers machine:

    Sending again a ping from Server to Client:

● Checking wireshark activity on Servers machine:

Now the reply was sent back but it is visible that the attacker’s IP is forwarding the requests!

Task 2 – step 4

● First we create a sniff-and-spoof script:

The function checks whether the captured packet is a TCP packet sent from the client to the server, and if it is, it modifies the payload of the packet by replacing all alphabetic characters with the letter ‘Z’. The modified packet is then sent back to the server.

If the captured packet is a TCP packet sent from the server to the client, the function simply sends the packet back to the client without modification.

The sniff function is called with a filter parameter set to ‘tcp’, which instructs Scapy to capture only TCP packets. The prn parameter is set to the spoof_pkt function, which means that spoof_pkt will be called for each captured packet.

    Next we repeat the previous steps, sending an ARP poisoning packets to both Client and Server machines:

● Again enabling IP forwarding on Attackers machine:

We also sent some activity for testing:

● Now, we turn off IP forwarding on Attackers machine:

Task 3 : MITM Attack on Netcat using ARP Cache Poisoning

This Task is similar to the previous one with the change that instead of telnet, we will use netCat and change the payload according to a different rules:

    Replace every occurrence of your first name in the message with a sequence of A’s.

    The length of the sequence should be the same as that of your first name ( we chose to use the name “Alina”)

        First we update the code from the previous task:

● Now we launch ARP poisoning again:

    Establishing netcat connection between client and server:

    Disabling IP forwarding again:

    We type some tests in the netcat, the screenshot includes all the tests, we ran the script in the last one:

We can see that we succeeded in changing the string.

Summary and reflection:

In this lab we delved deeper into the behaviour of ARP and discovered various methods for carrying out ARP poisoning attacks.

During our research, we stumbled upon a useful tool called XArp (https://www.youtube.com/watch?v=oPGZs65a3hY), which can help prevent ARP poisoning attacks. XArp actively monitors ARP traffic on the network and can detect any inconsistencies or anomalies in the ARP cache entries. If an attack is detected, XArp can take action to block the attacker’s IP or MAC address, thus preventing further damage.
