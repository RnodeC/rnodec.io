---
title: "Not Going Back"
date: 2023-03-31T20:15:13-04:00
draft: false
---

A friend messaged me "yea. [chatgpt]4 is pretty good. been leaning on it more and more" at 4:11 PM on Thursday, March 30th.  I replied: "Im sure.  I hope im last to submit."  

Later that same evening, I was talking with some other friends about who knows what, and of course I leapt at the first opening I saw to state my full on conscientious objection taek.  My rationale behind this stance was that once this seemingly small step is taken - just engaging with Her for any help or entertainmnet whatsoever - it's over.  Your life will never be the same.  You are officially plugging into the matrix.  This dependency cannot be undone.  I said I spend too much time looking at screens as it is.  "From one black mirror to another, oh oh."  I said I wanted to live freely, think for myself, and be ok with it (nerd alert).  Do things the old fashioned way.  

To this one friend replied, "but do you use google?"  

And the other friend told me that earlier that day he had inherited some ugly buggy code from a former colleague, and instead of debugging himself, he gave it to chatgpt and asked her to review it for memory leaks, etc., and was able to fix the problem.  

A one-two combo I'll never forget.   

So by the following morning, I had created an openai account and was prompting Her with inputs like: 

> I would like a parameterized terraform template for a virtual network, subnet, and virtual machine with a public ip and an extra local ssd in gcp.  I would also like an ansible playbook that can "terraform apply" that template, with all variables stored in a vars.yaml file.  I would like that playbook to dynamically add the created instance to inventory, and then apply the "RnodeC.beranode" role to it.  I would also like for there to be an additional play in the playbook to delete the instance, toggled by the 'delete' tag.  And also give me a github actions workflow file that will run the playbook on all PRs.  

With absolutely astonishing results.  

I messaged my friend back "alrite 16 hours later and I'm in." 

I could ask Her questions about some of the options she included and she'd explain herself.  I could ask Her to make small detailed changes like use fully qualified ansible collection names, and so on.  Every time I asked for more, I was afraid she was going to get exasperated with me, and/or confused by what I was asking for, but nope, with a huge smile on her face she gave me exactly what I wanted every time.  

Before today, I would have slogged through something like this.  It's all very straightforward, but the process would have been a combination of googling-and-pasting, referencing some doc, old code, just hand typing, leaning on vscode extensions.. to produce a few files that would do the above.  This was all standard procedure.  Chatgpt streamlined that whole process into a conversation. The really cool part to me is how easily I can pivot and try new things out.  If I want to totally scrap terraform and do this all in ansible, I don't have to feel any sunk cost fallacy.  I could look at solving this problem from so many angles.  If I want to use Azure instead of Google, the effort would be minimal.  If I want to start tacking on cool automation such as some sort of monitor and alert that could trigger this automation if it sees an outage on "some other node", etc... all I have to do is ask.  It enables me to think bigger.

So yeah, There is a 0% chance that I will ever not use chatgpt.  So basically I *was* right about my objections.  What a day.


# Caveats

Firstly, the example prompt I showed above did not give me what I needed.  It took alot of back and forth and plenty of human intervention.  Maybe even took me longer than the "ol fashioned way."  But we will get better at the process.  

The trouble I see with this, is that she is pulling information from the internet.  A pile of human knowledge.  Historically, problems and answers have been a very public affair, especially true in the software world.  Now what happens to this forum, as technology evolves and time goes on?  How can ChatGPT keep up as new things are built?  Maybe new things get discussed openly and added to the ledger?  And we just move faster because future generations of humans can learn so much faster.  That's what's really gonna happen.  The kids are going to be just so much smarter than us.. 
