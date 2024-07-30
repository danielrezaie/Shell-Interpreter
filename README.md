# Shell Interpretter
A small repository explaining how Shells interpret numbers.

## The Story
When using commands in a Shell, may it be in an NT or Unix system, any arguements you give to it needs to be interpreted. A funny quirk is that the ip address you  give it can be in tons of different formats like:
- Dot-Seperated (127.0.0.1),
- Hex (0x7F000001) and
- Denary (2130706433).

It's funny how people learn things by stumbling upon them and doing little tests.
I started doing some a ping test when I saw my ISP's Fibre line provider climbing up a ladder on my local telephone pole and changing out what seemed like Fibre patch cables to see if he was working on my house's cable.

I started by pinging google.com, which through the work didn't drop out once, when I checked out the ping to be 18mS which was interesting to see, so I pinged a few more random places, one being [1.1.1.1](1.1.1.1) which gave me a 12mS ping. This lead me to thinking about my DNS settings and wondering who my ISP's default DNS server is or if they host their own.

```
danielrezaie@m1mac ~ % ping google.com
PING google.com (216.58.201.110): 56 data bytes
64 bytes from 216.58.201.110: icmp_seq=0 ttl=116 time=19.671 ms
...
64 bytes from 216.58.201.110: icmp_seq=18 ttl=116 time=19.749 ms
^C
--- google.com ping statistics ---
19 packets transmitted, 19 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 15.606/18.118/19.749/0.994 ms
```

```
danielrezaie@m1mac ~ % ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1): 56 data bytes
64 bytes from 1.1.1.1: icmp_seq=0 ttl=58 time=12.730 ms
...
64 bytes from 1.1.1.1: icmp_seq=6 ttl=58 time=12.564 ms
^C
--- 1.1.1.1 ping statistics ---
7 packets transmitted, 7 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 12.214/12.856/14.066/0.719 ms
```

I then pinged my own [website](rezaie.co.uk) which is hosted by a longtime friend Jaden who runs [Fyfeweb](fyfeweb.com)[1] which gave me a ping similar to google's. 

```
danielrezaie@m1mac ~ % ping rezaie.co.uk
PING rezaie.co.uk (45.84.57.2): 56 data bytes
64 bytes from 45.84.57.2: icmp_seq=0 ttl=56 time=17.750 ms
...
64 bytes from 45.84.57.2: icmp_seq=9 ttl=56 time=17.671 ms
^C
--- rezaie.co.uk ping statistics ---
10 packets transmitted, 10 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 16.329/17.350/18.412/0.739 ms
```

I gave it a thought for a moment and remembered that Fyfeweb has a server in [LONAP](lonap.net) so maybe Google's and Fyfeweb's closest Servers are both in LONAP while CloudFlare's 1.1.1.1 DNS server may have one located closer to me.


As usual I got bored and pinged 2.2.2.2 out of curiousity which didn't reply to my ping requests :(

```
danielrezaie@m1mac ~ % ping 2.2.2.2
PING 2.2.2.2 (2.2.2.2): 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
^C
--- 2.2.2.2 ping statistics ---
4 packets transmitted, 0 packets received, 100.0% packet loss
```

So I marched forth trying to remember CloudFlare's secondary IP address which I managed to guess as 1.1.1.2[2] and got a ping similiar to 1.1.1.1.

```
danielrezaie@m1mac ~ % ping 1.1.1.2
PING 1.1.1.2 (1.1.1.2): 56 data bytes
64 bytes from 1.1.1.2: icmp_seq=0 ttl=58 time=13.647 ms
...
64 bytes from 1.1.1.2: icmp_seq=4 ttl=58 time=11.601 ms
^C
--- 1.1.1.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 11.329/12.193/13.647/0.802 ms
```

Continuing on my journey of being bored, I wanted to see what my internal loop-back[3] ping was but accidentally I wrote 127.0.01 instead of 127.0.0.1 but weirdly the Shell responded by pinging 127.0.0.1, which I thought was strange

```
danielrezaie@m1mac ~ % ping 127.0.01
PING 127.0.01 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.089 ms
...
64 bytes from 127.0.0.1: icmp_seq=5 ttl=64 time=0.139 ms
^C
--- 127.0.01 ping statistics ---
6 packets transmitted, 6 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.089/0.122/0.139/0.018 ms
```
## Reverse Engineering
So seeing what just happened I wondered if I can leave all the decimal places out but thought it might think 127001 was 127.1.0.0 so decided to use 127000000001 as maybe it would use it as 127.000.000.001

```
danielrezaie@m1mac ~ % ping 127000000001
PING 127000000001 (145.202.54.1): 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
^C
--- 127000000001 ping statistics ---
4 packets transmitted, 0 packets received, 100.0% packet loss
```
but sadly it didn't work so I tried some my original idea of 127001...

```
danielrezaie@m1mac ~ % ping 127001 
PING 127001 (0.1.240.25): 56 data bytes
ping: sendto: No route to host
...
Request timeout for icmp_seq 9
^C
--- 127001 ping statistics ---
11 packets transmitted, 0 packets received, 100.0% packet loss
```

HUH? It's pinging something? It started pinging 0.1.240.25 which I thought was weird so I dug some more and realised it was using 127001 as Hex.

1. It took 127001 and turned it to Hex
    127001 >>>>> 0x1F019
2. Padded it to 4 Bytes
    0x1F019 >>>>> 0x0001F019
3. Broke it down to its 4 individual Bytes
    0x0001F019 >>>>> 0x00 0x01 0xF0 0x19
4. Finally converted it back to denary
    0x00 0x01 0xF0 0x19 >>>>> 0 1 240 25
which was what my shell took it as.

## Testing
Seeing this I wanted to see if I could correctly changed 127.0.0.1 to work as Hex. So I used some websites[4] and reverse logic:
1. I changed 127.0.0.1 to Hex
    127 0 0 1 >>>>> 0x7F 0x00 0x00 0x01
2. I smushed all 4 Bytes together
    0x7F 0x00 0x00 0x01 >>>>> 0x7F000001
3. I skipped step 2 since 0x7F000001 was already 4 Bytes and instead tested the Hex values we just found
```
danielrezaie@m1mac ~ % ping 0x7f000001
PING 0x7f000001 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.089 ms
...
64 bytes from 127.0.0.1: icmp_seq=7 ttl=64 time=0.113 ms
^C
--- 0x7f000001 ping statistics ---
8 packets transmitted, 8 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.079/0.099/0.122/0.013 ms
```
4. Seeing that it worked so far, I took 0x7F000001 and converted it to denary
    0x7F000001 >>>>> 2130706433
Finally taking our new value and pinging it we can see that our experiment worked :D

```
danielrezaie@m1mac ~ % ping 2130706433
PING 2130706433 (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.101 ms
...
64 bytes from 127.0.0.1: icmp_seq=11 ttl=64 time=0.116 ms
^C
--- 2130706433 ping statistics ---
12 packets transmitted, 12 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.054/0.103/0.159/0.029 ms
```





## References

[1]: A popular and well-serviced Webhosting business who do a lot of other things too

[2]: 1.1.1.2 is CloudFlare's Malware-filtered version of 1.1.1.1, so technically not it's secondary IP which is 1.0.0.1

[3]: 127.0.0.1 is a loop-back address that all deviced that abide by atleast RFC1122, **which came out in 1989!!!**

[4]: https://www.rapidtables.com/convert/number/decimal-to-hex.html and https://www.rapidtables.com/convert/number/hex-to-decimal.html
