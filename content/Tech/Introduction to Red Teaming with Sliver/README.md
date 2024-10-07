
---
title: "README"
draft: false
tags:
  - 
---

This series is an introduction to red teaming or offensive operations using the Sliver C2 framework. My goal here is to provide a guide that strikes a balance between the following:

1. Introduce Sliver as a C2 framework, for someone already familiar with red team operations
2. Introducing red team operations to those only familiar with traditional pentesting etc.

There are already many excellent examples of 1, such as from [Dominic Breuker](https://dominicbreuker.com/post/learning_sliver_c2_01_installation/#series-overview),  [Red Siege](https://redsiege.com/blog/2022/11/introduction-to-sliver/), and [eversinc33](https://eversinc33.com/posts/getting-started-with-sliver/), not to mention the [official documentation](https://sliver.sh/docs). However, these posts or series tend to concentrate on getting sliver up and running and giving an overview of its features for someone already accustomed to red team operations, and looking for an intro to a Cobalt Strike alternative with many details left unsaid.

Regarding 2, on the other hand, while there are very good intros out there such as [Husky's](https://taggartinstitute.org/p/responsible-red-teaming) and [Rasta's](https://training.zeropointsecurity.co.uk/courses/red-team-ops), this isn't quite my goal either. It would both be redundant and frankly too much extra work for me to treat this as a generalized introduction to things like CONOPS, infrastructure, what we might call the "pentest-red-team-spectrum," etc. 

Rather, my goal is to fall somewhere in the middle. I hope this series is useful to a range of people, from those already experienced with offensive ops but looking to learn Sliver specifically, to those who are already relatively familiar with the basics of offensive security but want to move in the direction of more sophisticated offensive ops. Therefore, this series doesn't aim to be at all comprehensive, but to give you a glimpse of how all these things fit together, in the hope you're inspired to learn and hack more. One way I've tried to achieve this is by moving back and forth between the flow of an attack/operation rather than a topical overview on the one hand, and technical details on the other.

In light of the above, I think it's important to clarify that I will not be providing a step-by-step guide to setting up a specific environment. While I will be describing my environments, I'm assuming you can figure out how to replicate those environments in the relevant ways. For example, while at some points I will be using [GOAD](https://github.com/Orange-Cyberdefense/GOAD), I won't be providing details about how to get Vagrant to work correctly with WinRM. My goal is to demonstrate things that are mostly independent of the specific target environment anyway, at least to the degree that's ever possible. That disclaimer out of the way, I do provide some detail in setup(TODO).

Finally, like the rest of this website, this series should be considered a living document, which is subject to change at any time.

