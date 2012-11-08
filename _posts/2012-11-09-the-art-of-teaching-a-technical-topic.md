---
layout: post
title: The art of teaching a technical topic
author: achentir
tags:
- google
- talk
- android
---

Recently I have been asked to give a couple of talks during the Google Extended 
Day in Algiers (Algeria). Google Extended are events happening all over the world 
that Google sponsors and co-organizes with the local communities to allow the developers 
who couldn't make it to their annual event (Google IO) to connect with other developers, 
learn about new trends/topics, try new hardware and talk to Googlers. 
I was invited as a speaker to the event, in order to teach the localcommunity how to 
program an Android application . 

To do that, I had to give three talks: 

 * The first was called Android Overview and was a kind of 101 session. 

 * The second was called Android DevTools and it was about introducing the different tools
   used in order to: develop, debug, test and deploy your Android application.

 * The third one was called Advanced Android Development and half of it was a live coding 
   session and the other half was about explaining the best practices that need to be 
   followed when building a mobile application in general and an Android application 
   in particular.

Now, this was literally my first experience as a teacher and I found the task very challenging and fun.  In order to share that experience, I decided to come up with a list of good practices to consider when teaching any technical topic and here they are:

-> Set very clear expectations from the beginning
My talks had to be 45 mins long max (that includes the Q/A sessions). That's a very short amount of time and no matter how focused your presentation is, it is impossible to teach all the fundamentals that your audience will need to build their first application within that timespan. On the other hand, your audience comes to your talks hoping to learn everything they need to magically build their first MVP (Minimal Viable Product) during the event Hackathon. So, if you don't want to disappoint your audience it is important to set very clear expectations at the very beginning of your talk.  Ultimately you want your audience to understand some fundamental notions and then take it from there to learn more by reading the online documentation. Your goal here, is to be the ice breaker between them and that topic. They need to grab your hand before making jumping from one side of the river to the other, and once they made that jump you can just let them go wonder by themselves.

Choose your content carefully
Because the time is limited you have to make sure that every instant of your presentation is worth an important information. Prior to your presentation, you have to triage the different things that you want to teach. A good process would be:

1/  Brainstorm and write down all  the notions that you think should be taught.
2/ Classify those notions within 3 categories: Mandatory/Important/Optional. 
3/ Prepare your normal presentation with the mandatory and important notions first. 
4/ Take your time explaining the notions by their order of importance.
5/ Prepare some extra-slides with the optional notions in case you have extra-time or if they come up in the Q/A session.

-> Have a clear narrative
There is a notion in writing called: the Tree-act structure (link: http://en.wikipedia.org/wiki/Three-act_structure). It is basically a model used in writing a modern storytelling which can be found in a wide range of  domains: Drama, poetry, comics, novels, movies, video games, to name a few. This notion basically says that to tell a good story we have to structure it in 3 acts:

Acte 1: The Setup, which is where all the major characters of the story are introduced.
Acte 2: The Confrontation, which is where all the story, its characters and conflict are all established.
Acte 3: The Resolution, which represents the final confrontation of your story or dénouement.

The nice thing about this structure is that it provides a clear path to bringing some knowledge to an audience. In the case of Android, we could imagine the 3 following acts:

Acte 1: Setting up the context, introducing the software architecture, explaining some fundamentals notions like the Android Runtime, what's Dalvik and how it works,  how does Android manage memory etc.  This is probably the most boring part of the talk but it's so important that you have to find an entertaining way for presenting it.

Acte 2: Use the notions taught during Acte 1 to explain how to build an Android application.

Acte 3: Do a demo.

-> Be prepared to fail
This is a no brainer and kind of related to any type of presentation that involves demoing something. To put it simply: Expect everything to fail! That includes: Internet connexion will go down during the event, your favorite IDE will crash, your project won't compile for some odd reasons etc. When preparing your presentations you have to prepare at least 1 backup plan for each failure scenario. Internet will go down? Then download everything that you need so that you can use it offline (example: save the result of an API call in JSON in a file). Afraid that your favorite IDE will freeze and crash many times during the demo? Then be ready to use Vi/Emacs/Sublime Text/ or whatever you love and be ready to use the command line to build and deploy your project.
Now, if you experience a demo fail during your presentation then instead of apologizing and running away try to discover what is going on live by thinking loudly and by showing them the tools that you use in order to track down bugs and fix them. Once you recovered from that failure, pass the details along to the audience, explain why it failed and how it is possible to avoid it in the future. This is how you should apologize, by making the failure useful.

-> Know your audience
In order to shape the right presentation, it is important to know who you are going to talk to.  Are they students? Are they professionals with a lot of experience? Are they technical at all? Obviously, your material as well as the vocabulary that you are going to use will vary depending on who your audience is.

-> Make sure that your knowledge is correct
There is nothing more embarrassing that teaching something wrong.  Now, I suppose that you have been given the responsibility to teach a large group of people about a topic because you clearly have some experience on it. But here is the thing, because you are experienced and because you are using this technology everyday at work you are probably making a lot of assumptions about how certain things work and those assumptions might be a little off. There is a great quote by Yogi Bhajan that says: "If you want to learn something, read about it. If you want to understand something, write about it. If you want to master something, teach it".  Because your audience will drink your words as if it was water, you can't afford being too vague you have to own your topic and master it. For example if you want to pass some details on how Dalvik works, you could take an hour reading the official specification rather than trust whatever is written on Wikipedia.

-> Be funny & entertaining
You, as a teacher, have great power and that comes with great responsibility. As a matter of fact, the way you are going to teach is likely going to determine whether your audience will develop a passion for that topic or will develop an aversion for it. Everyone has some bad memories about that weird/boring/tyrannical teacher who made every instant of its classes, feel like a very bad nightmare. Let's pretend he was teaching physics, as a consequence  it is likely that a non negligible number of students will associate physics to this teacher and because they hated him, they will hate physics for their entire life.  The moral of that, is that by making your presentation pleasant to watch and entertaining you are eventually going to create passion for whatever you are teaching.  Passion is important, because this is the reason that will motivate your audience to spend their free time reading the official documentation or the specification and building interesting stuff.

-> Only teach topics that you are interested in
This point is slightly related to the previous one. If you want to entertain and interest your audience, you have to show some passion on the topic you are presenting. If you are bored, your talk will be boring and your audience will not follow.

-> Take advantage of the situation to find talent
When teaching a topic to a group of people, you'll likely encounter three 4 types of persons: 
Those who don't pay too much attention (wrong room may be?)
Those who pay attention but struggle at understanding the content.  
Those who understand everything.
Those who understand everything and who want to know even more details about each topic. By their questions, they sometimes force you to think hard and they might as well teach you something new.

This is the ideal situation for you to track down the persons who fit within the last category. Go talk to them after the sessions or during the party, find out who they are, where they work and hire them!

-> Determine success
It is important for you, to determine and measure the success of your interventions. That way you will know what to keep and what to change next time (and yes there will be a next time).
In the case of teaching Android to beginners, I defined success by the following metrics:
The number of questions asked during Q/A.
The quality of the questions asked during Q/A.  Are they silly questions, like: What is the difference between an Activity and Intent again? Or are they more specific and clever? 
Whether the attendees knew how to start building their application during the Hackathon and whether they followed the platform guidelines.

By the way besides all of these points, you should also consider all the good practices that you need to follow to do a great presentation.

Final words: why you should care about doing it well?
Teaching is hard, doing it well is even harder but if you are successful are it the Return Of Investment is very precious to you and your company:

1/ Attendees will be happy and will associate their happiness to your products. 
2/ These guys will ultimately be your best ambassadors, they will talk about you and your company to their friends and colleagues. 
3/ You will attract talented engineers to your team. If you show passion and a certain level of mastery on the topic that you are presenting, it is likely that the talented engineers who attended your talks will develop sympathy for your company and will try to know more about it and eventually they will want to join you there!
4/ If your presentation was good, it will be shared hundred or even thousands of times .  It will go viral!
5/ The Q/A session might bring some interesting discussions from which you can learn new things.

And finally, if you can, take some time to network, socialize and to talk to other developers or even non-technical participants. A simple discussion during an event like that can be the origin of your next big idea!

So go out, teach what you know and do it well!

\- [Amokrane Chentir](https://github.com/Amokrane/)

Amokrane Chentir - Client Software Engineer
