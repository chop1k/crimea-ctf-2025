# Wireshark

Task from reverse engineering category with forgotten name.

## Input

[Pcap file](../Payloads/task.tar.gz).

## Goal

Find flag in the pcap file.

## Solution

Open the file in Wireshark, use the default http filter.

![1](../Images/wireshark_1.png)

Check contents of filtered queries.

![2](../Images/wireshark_2.png)

Guess what's there?

## The flag

```text
crimea_ctf{CARdingc@rderS_iP_rePoRt_St@NdARD}
```
