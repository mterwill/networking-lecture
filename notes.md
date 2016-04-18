# Networking Lecture Notes
Matt Terwilliger | mterwil@umich.edu

For University of Michigan's EECS 398 (Computing for Computer Scientists)
lecture on 4/15/16. Note this is by no means complete and oversimplifies some
processes for clarity. It does, however, provide an accurate high-level overview
of important networking concepts.

Recording available at http://leccap.engin.umich.edu/leccap/viewer/r/WNFhEi

Jumping off point: what happens when you type google.com in your browser?

## Request
Our browser generates a request to Google's servers. This contains key
information like what page we want and on which protocol we are communicating.
The request heads out over a TCP port (more on this later).

## We can have a server too!

    cd Desktop
    mkdir website
    echo "Hi c4cs!" > index.html
    python -m SimpleHTTPServer

- Listening on 0.0.0.0 means every available network interface
- 8000 is the port. If you visit this in a browser it won't work unless you
  specify the port. Why? Convention.
- Lesson: just because you have a valid IP/DNS name doesn't mean you can type it
  into the web browser and have it work.

## DNS translation
- We want to get to google.com, but computers don't understand google.com, they
  understand numbers. Humans don't like numbers. What protocol helps us get
  there?
- `nslookup google.com`
    - Where did it get this server? When you connect to a network you
        automatically get some DNS servers. 
    - `cat /etc/resolv.conf`
    - There are many DNS servers. What happens if we use a different one?
    - `nslookup google.com 208.67.222.222`
- Non-authoritative answer: I don't know, ask somebody else:
    - Recursively! If we take a DNS name backwards:
        - com registry knows who has data for google
        - google nameserver knows what www is
        - www is nothing special
        - who knows who com is?
        - distributed 13 'root zone servers'
        - `dig +trace www.google.com`
        - This is the authoritative answer!

One way or another we get an IP address for Google.

## Routing
- Now we have this IP address. How do we get there?
- `traceroute google.com`
    - `whois 1e100.net | less`
- But how did it know to get there?
    - Same concept as DNS we recursively go until we know what to do.
- `route -n get google.com`
    - Show umich 35.2.0.1 
    - UMnet backbone map, Merit = comcast
- `netstat -nr`

## Response
Our request arrives at Google's servers. They do whatever sort of backend
processing to serve us a page and send it back out over the wire. How does this
work?  When we sent our request, we included an ephemeral __source port__.

- So Google has our source port and our source IP address and is able to
  complete the exact same process to get the data back to us.

The data's delivered and our browser renders the page!

## More on this port nonsense
When we made our initial request to Google we did it over port 80. Why port 80?
Because that's the port HTTP communicates over. How did we know that? It's
defined by IANA, [Internet Assigned Numbers Authority](http://goo.gl/bWH2FK).

Basically everything on the Internet is just an agreed upon set of standards
that programmers follow.

## SMTP
A great example of this is SMTP (Simple Mail Transfer Protocol). This is the
defined protocol through which email is sent.

How do all of these servers know how to communicate with one another? Let's send
Pat an email.  Just like HTTP, in order to send an email we need to make a
request to a server.  Let's use the telnet application (not protocol), which we
can use to rudimentarily/interactively follow the TCP stream:

    Wayfarer:~ mterwilliger$ telnet aspmx.l.google.com smtp
    Trying 64.233.166.27...
    Connected to aspmx.l.google.com.
    Escape character is '^]'.
    220 mx.google.com ESMTP o8si25130477wjo.165 - gsmtp
    helo c4cs
    250 mx.google.com at your service
    mail from:<mterwil@umich.edu>
    250 2.1.0 OK o8si25130477wjo.165 - gsmtp
    rcpt to:<ppannuto@umich.edu>
    250 2.1.5 OK o8si25130477wjo.165 - gsmtp
    data
    354  Go ahead o8si25130477wjo.165 - gsmtp
    From: Matt Terwilliger <mterwil@umich.edu>
    Subject: Hi Pat
    We are learning about networking today!
    .
    250 2.0.0 OK 1460322388 o8si25130477wjo.165 - gsmtp
    quit
    221 2.0.0 closing connection o8si25130477wjo.165 - gsmtp
    Connection closed by foreign host.

This is a cute way to see that computers just communicate with other computers
across the Internet in predetermined ways, it's nothing special.

## Extra material

### OSI Model
- Layer 7: Application (DNS, FTP, HTTP, NTP)
- Layer 6: Presentation ()
- Layer 5: Session ()
- Layer 4: Transport (TCP, UDP)
- Layer 3: Network (IP, ICMP)
- Layer 2: Data link (ARP, MAC)
- Layer 1: Physical (Wireless, Bluetooth, Ethernet)

Most of the stuff we've talked about is built on TCP (L4). Why? The lower you go
the dumber the technologies are.

- Each layer is just another level of abstraction
- So cool that we're able to send 0s and 1s over hundreds of miles of
  copper/fiber optic cable and have them arrive intact.
- TCP guarantees packet arrival and ordering, etc. ( __TODO:__ Write a blurb
  about TCP vs UDP)
- WiFi, Ethernet, etc. drop/corrupt packets all the time. Think about all of
  the interference.

TCP makes sure the user doesn't have to deal with this, so that's why protocols
like HTTP are implemented on top of it.

### To consider
- DNS is nothing special, just another L7 service
- Firewalls/NAT/private network routing
    - ARP
    - Try to go to VM from another machine (won't work)
    - RFC 1918 (private networks)/NAT/virtual interface
- DHCP
- Explain loopback
- L3 vs L4
    - Each machine has one IP and thousands of ports
    - Each service has it's own port
    - Some ports are privileged or reserved
- How far down the rabbit hole of routing... Talk about local networks, and/or
  more advanced routing protocols? Probably too complicated and would be more
  confusing than helpful.
- Something about ping
- Maybe add a case when this stuff doesn't work like you'd expect
- Analyzing networks tcpdump/wireshark
