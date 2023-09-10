Torrent Analyze is a problem with packet capture that should be analyzed
Depicted is my problem analysis and solution.

**Description**

Author: Mubarak Mikail

SOS, someone is torrenting on our network. One of your colleagues has been using torrent to download some files on the company’s network. Can you identify the file(s) that were downloaded? The file name will be the flag, like picoCTF{filename}. Captured traffic.

Hints
1) Download and open the file with a packet analyzer like Wireshark. https://www.wireshark.org/
2) You may want to enable BitTorrent protocol (BT-DHT, etc.) on Wireshark. Analyze -> Enabled Protocols
3) Try to understand peers, leechers and seeds. https://www.techworm.net/2017/03/seeds-peers-leechers-torrents-language.html
4) The file name ends with .iso


**Recon**

I used Wireshark, which is my goto tool for packet captures. This file is definitely a capture of a torrent file.

A torrent is a peer-to-peer way to share files with others. It relies on the popularity of a torrent to work. A person hosts a file to share, which is then uploaded in chunks to other people. These people who have downloaded the document should then host a portion of that file. This is often done during and after a torrent has been downloaded. Those that have no uploaded the equivalent of what they have downloaded are called leechers, and those who upload more than they download are called seeders. Leechers are often thought poorly of by those who torrent, as they gain form others hosting the files but don't contribute back to the network. Torrenting itself is legal within the United States, however with its peer to peer nature it is often used to transfer illicit files such as copyrighted materials. For this reason, an understanding of torrenting is necessary to catch those on a network doing illegal things. Please don't use the torrent network for malicious or illegal operations. It can be used for many viable reasons.

Looking at the protocol, there is an indentifier hash which could identify a torrent. https://en.wikipedia.org/wiki/BitTorrent This is stored within BitTorrents DHT protocol. Hint 4 shows that the file is a .iso file. This is how operating systems are typically packaged, and if a person were to download a fully packaged verson of an OS such as Ubuntu Linux or Linux Mint, this is how they would most likely do it. We will be looking for an OS download. This most likely means that this torrent is a more popular torrent which could have the identifier hash easily searchable on the internet.

**Analysis**

Opening the packet in Wireshark, we can finally start searching for what we think could be the awnser to the problem. First, make sure you have enabled the BitTorrent protocol in Wireshark (like hint 2 shows how to do). Then, you can filter your packets by the packet filter "bt-dht". This should filter down the packets displayed from 58551 packets down to only 296 packets. Going through a few packets, you can find some that contain the string "info_hash". 
Then, you can use the find a packet functionality shown with the magnifying glass up top to find what you wish to look for. Search packet details for the string "info_hash". 
One of the packets had the value: e2467cbf021192c241367b892230dc1e05c0580e
Looking that value up online found the link: https://linuxtracker.org/index.php?page=torrent-details&id=e2467cbf021192c241367b892230dc1e05c0580e
The .iso file was ubuntu-19.10-desktop-amd64.iso
If we found the .iso file name, we found the flag! It was: picoCTF{ubuntu-19.10-desktop-amd64.iso}
