
---
title: "Part 1 - Introduction"
draft: false
tags:
  - 
---
 
# Introduction to the introduction

Welcome to the series! Here, we'll discuss some basic setup and environment considerations both so you have an idea of where to start and so that you have some idea of what we're attacking. As mentioned in the [[content/Introduction to Red Teaming with Sliver/README|README]], I'm don't aim to be comprehensive or step-by-step here, but I don't want you to be disoriented either.

# Preliminary considerations

For the purposes of this series, redirectors, secure C2 server deployments etc. are out of scope. 

Let me repeat: we will *not* be discussing how to deploy Sliver in an OPSEC-safe way or, frankly, a way that is appropriate for a real operation. Those issues - redirectors, OPSEC-safe red team infrastructure, etc. - is covered elsewhere.

# Sliver installation

There are a variety of ways to install Sliver. The basics are detailed in the official documentation [here](https://sliver.sh/docs?name=Getting+Started) and [here](https://sliver.sh/docs?name=Linux+Install+Script). For reasons that will become clear later in this series, you want the server running on Linux even if you use macOS or Windows for the client.

To install from the latest official binary package on Linux regardless of distribution, use the methods linked above. It's worth noting, however, that Kali, for example, does package Sliver, which can therefore be installed with `apt install sliver`. As is common for rapidly evolving projects such as a C2 framework, there's a tradeoff: it's more convenient to manage installation and especially updates with the distro package 