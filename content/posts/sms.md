---
title: "SMS Behavior Analysis"
date: 2018-01-15
draft: False
#featuredImage : "img/dndprob.jpg"
---

Introduction
------------

There are a few things I enjoy in life, statistics, cool graphs,
open-source software licensed through the
[GPL](https://en.wikipedia.org/wiki/GNU_General_Public_License#Version_3),
and encrypted online communication, to name a few. This blog post aims
to hit all of those. While I don't necessarily endorse or recommend
[Signal](https://signal.org/), I have forced everyone that I talk to
with any regularity to download and use the software. Signal lets you
download plaintext backups of all your conversations, unfortunately the
backup is in XML, which is only a moderate pain to deal with. I parsed
it and put it into a dataframe that R can work with. And like that, the
introduction is out of the way and we can dive into cool graphs about my
texting behavior.

Cool Graphs
-----------

![](/sms/top_friends-1.png)

OK- I text a lot, what of it? Signal has a great desktop client, so I
can message anyone from my computer while I'm bored at work, and this is
the entirety of my texting history. The graph is filtered to only show
people I've text more than 2000 times, because if we haven't text that
much, we probably aren't friends.

![](/sms/daily_overtime-1.png)

The graph above shows every day as a line, the x axis is the hour of
day, and the y axis is the count of sent messages. The lines are colored
to show who is sending the message, me or the girlfriend, and the size
of the dot signifies how long the average message is in words. Nothing
too surprising here, clearly our days start at 8. I was hoping to see
some cooler stuff with the size of the message, but it looks like we are
pretty consistent in the number of words we use. The spikes in the daily
data may lead you to believe all we do is text, but we can plot the
average number of sent messages between us below.

![](/sms/daily_overtime2-1.png)

Interesting graph, we wake up at 8 and during lunch we don't talk as
much- clearly eating is important to us. Considering we both work on our
computers and both have the desktop client, these numbers seem
reasonable to me.

The next graph is my a count of received messages over time since we
started dating. I've used Loess smoothing help make some sense of the
graph, it's really cool stuff, you can read about it
[here.](https://en.wikipedia.org/wiki/Local_regression)

![](/sms/yearmo-1.png)

Around 2016-09 we tried using another messaging app, so there is a long
pause in the data there. We also spend the weekends together and text
infrequently, which is why we see such a jagged line. It looks like our
texting is increasing over time- subsequent blog posts might delve into
a Bayesian way of finding the change-points in our texting behavior over
time.

Below I plot the same graph as above, but this time color code my
messages versus hers. Conclusions here are that I text more than she
does, although she is certainly catching up.

![](/sms/yearmo2-1.png)

Statistics
----------

I really can't have a blog post about SMS data and call it a success
without doing some form of text analysis. There are a lot of really
advanced things you can do with words, but I was interested in a simpler
approach. I wanted to see what words are most 'Carmen', as in, words
that uniquely identify my speech patterns. If you saw these words, you
would know it's me. I'd also like to do the same for my girlfriend.
Constructing a Document Term Matrix was simple, it's a matrix where the
rows are the individual SMS messages and the columns are the words. The
values of the matrix are whether or not the word appeared in the SMS.
Once I had that, for each word, I calculate the percent of SMS's that
contained that word for both of us. For every word, I now have the
probability of seeing that word from me and seeing it from my
girlfriend. That's it, really simple, just counting and division. Below
are the 'Most Carmen' and 'Most Girlfriend' words.

    ## [1] "'Most Girlfiend' Words"

    ##       Term
    ## 1     till
    ## 2     yeee
    ## 3     okay
    ## 4      tho
    ## 5     kewl
    ## 6 annoying

    ## [1] "'Most Carmen' Words"

    ##         Term
    ## 1      cause
    ## 2        yuh
    ## 3        til
    ## 4     awhile
    ## 5      hungy
    ## 6 statistics

Note: I promise I didn't text her the word 'statistics' over and over
prior to downloading the data to artificially bring that word to the
top, but I did include to top 6 words, as opposed to 5, to make sure it
was included.
