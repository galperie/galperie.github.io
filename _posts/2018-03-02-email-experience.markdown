---
layout: post
title: "Entering the world of Email :mailbox_with_mail:"
date: '2018-03-02 02:30:02 -0400'
categories: email
---

Personally, as a web developer creating and working on emails is bonkers. There are the typical css issues I struggle with, "how do I get this thing to be next to this other thing but also under it on mobile screens ". But emails take these problems, flip them, inverse them, and inflate them: it takes away javascript and its fancy libraries, and forces you to make it work for Outlook 2016 and the gmail app on iPhone 5s and yahoo mail (all of which render differently and add its own special code). It gets tricky. It makes you resort to all sorts of solutions like putting everything in a million html tables and relying on inline css.

But along the way it’s really forced me to learn and understand some fundamental css and html, learn how email clients actually work, and get introduced to a very passionate and small community I didn’t even know existed.

This blog post is for some observations I’ve found about working on email and will be updated as I go.

- Can Spam laws
- Loading JavaScript
- Alternatives to JavaScript
- Email tracking
- How email clients work

### Can Spam :rice_ball:

So everyone has had to deal with spam email at one point or another. It could be a phising email trying to get you to send over your password or some company that will not stop sending you promotional ads.

According to the Federal Trade Commission (FTC), an email can be marked as spam if it:

- Uses false or misleading header information
- Uses deceptive subject lines
- You do not identify the message as an ad (and it is)
- You don't tell recipients where you're located
- You don't tell recipients how to opt out of receiving future emails from you.
- You don't honor opt-out requests promptly
- You don't monitor what others are doing on your behalf (aka you can't hire another company to take all legal responsibility if you fail to comply)

5 and 6 I think is the tricky part.

Some email clients have gotten scary good at catching what is Spam. To the point you might never even see Spam in your inbox because it’s sent straight to the spam folder. These auto detections of spam make content creators very careful with how often they send emails and their content. If a major provider like gmail marks you as spam, you’re in trouble. You’ll have to fix the issue and submit a request to be remarked.

If you are curious and wanna deep dive into the laws more here it is from the source itself (https://www.ftc.gov/tips-advice/business-center/guidance/can-spam-act-compliance-guide-business).

### Loading JavaScript

Another law to help protect users from malicious emails is the prevention of JavaScript loading on the email open. Imagine if this was ok. You could have the content changing on load and bypassing the email clients spam filtering and checks. You could maliciously change links and trick people. This is great as a user but makes it sometimes tricky for developers. You cannot depend on javascript any more.