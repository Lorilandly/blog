---
title: "How to easily setup your wireguard"
date: 2024-10-10
---

I have been hosting my own VPN's for at least 5 years. Why do I have this obsession? Because like all my Chinese friends, I'll need to use VPN to access the internet outside when I get back to my motherland. It's not a choice, it's a necessity. 

I wasn't interested in wireguard in the first place since wireguard cannot be used to curcumvent the GFW(great fire wall). GFW blocks any traffic that it consider to be VPN. To cross GFW, a protocol needs to focus on conseal and disguise, act as if it's just normal TCP traffic out there. Speed shouldn't be a top priority. Wireguard, on the other hand, uses UDP and focuses heavily on speed and connectivity, screaming and shouting "I'm Wireguard!" as it goes across the network. You will be lucky if your wireguard server didn't get banned by the GFW the next day you set it up.

However, if you don't care about crossing the GFW, wireguard is AWESOME. And it doesn't work like any other VPN that I've worked with before. It doesn't categorize hosts into client and servers, everyone is equal. It uses public/private key encryption, allowing it to be secure and achieve 1RTT roundtrip. It is made for connecting multiple local networks, so you could make your servers at different geological locations appear as if they are in the same network.

### TBC...
