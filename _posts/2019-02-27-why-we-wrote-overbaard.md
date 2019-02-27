---
layout: post
author: <a href="https://github.com/kabir">Kabir Khan</a>
---
A few years ago we decided on the Red Hat JBoss Enterprise Application Platform team to 'go Agile'. I volunteered to
help with our process for tracking feature requests and to look into tooling. As we use Jira for our issue tracker we needed 
something that could work with Jira.

As a bit of background, we are the biggest middleware product at Red Hat. There are 50+ major components/areas of 
responsibility. The number of
engineers on the core team varies but it typically around 50+ people. In addition we consume numerous other large and 
successful JBoss projects (e.g Hibernate, RestEasy), as well as other external projects (Apache ActiveMQ, Apache CXF).
So as you can imagine there are a lot of people involved, and that is just from the engineering side! Then of course
every new feature needs testing by our QE team, it needs to be documented by the docs team. Add to this that we are 
spread throughout the world, spread over all timezones and both hemispheres.

Also, as people have different areas of expertise in our application server, it became apparent that the normal daily
standup meeting setup wouldn't really work for us. Due to our size, most topics during these standup meetings would
be irrelevant to most people on such a hypothetical call at any time, so it would be unfair to force people in unlucky
timezones out of bed to join. So, was key to us to come up with a process that could handle communication between all 
these people efficiently.

<h2>Initial Agile attempt</h2>
In our first attempt of the process, we came up with a sequential workflow. One of the major things we came up with 
was the introduction of an analysis document which would get upfront agreement between engineering and QE about the
scope of the feature. We ended up with something like 23 states! Without summarizing them all here, we had the 
following high-level categories:
<ul>
    <li><b>Backlog</b> - Self-explanatory</li>
    <li><b>Analysis</b> - This was a handful of states for Engineering to do analysis, QE to do their analysis and a 
    few states inbetween</li> 
    <li><b>Design/POC</b> - Experimental work</li>
    <li><b>Development</b> - As EAP is based on the open source <a href="http://wildfly.org">WildFly</a> application 
    server, we first needed to do the work in the wildfly branch, get it reviewed, merged and then backported to the 
    EAP product branch. For the product branch the reviewing and merging needed to happen again. TL;DR: We ended up
    with a lot of states for this.</li>
    <li><b>QE</b> - States for QE to implement and run their tests to verify the feature.</li>
    <li><b>Docs</b> - States for our docs team to write the docs and have them verified by QE.</li>
    <li><b>Verified</b> - All done!</li>
</ul>
    
Keen readers will note that this isn't really 'Agile', but that isn't the topic for this post. I'll write how we evolved
to what we have now at a later date so stay tuned. It was our starting point, and led to the development of Overbård as
we will see below.

<h2>Jira Agile</h2>
As we were stuck with using Jira, the obvious starting point was to use Jira's built in Agile support. (I said 'stuck with',
but we generally really like Jira, and it is nice to have a common issue tracker for all our open source projects, and
resulting products.)

<h3>Too narrow</h3>
I set up the states for our workflow in Jira Agile and noticed that there were a few problems caused by our 
(admittedly insane) number of states. Basically, it tries to fit the whole board into the width of your browser. 
This makes it impossible to see the issues - you can't even see the issue keys! If only they had made the board
scrollable or something?!?

<img src="/assets/images/posts/2019-02-27-why-we-wrote-overbaard/jira-agile.png"/>

We experimented with splitting the board into a four separate boards, each containing a sub-set of the states. This 
worked up to a point, but wasn't really workable because of what I mention in the next section 

<h3>Maintenance nightmare</h3>
To make the board more useful for each team, we experimented with adding filters and swimlanes for a few things. Take 
components for example, of which we have 50+. To set up a board that does swimlanes by components, you need to define
a query along the lines of `component=XXX` for every component. And after the investigation in the previous section 
we don't have one 'base' board, we have 4 to fit the states onto the screen. So we need to copy the 4 base boards into
ones for swimlane by component and enter the same 50 JQL queries in each.

Still, the set of components is quite stable, and we don't add or remove them that often. We do however use labels 
**a lot** for ad-hoc communication about what is important. So we would need to identify all the labels that are 
important, create copies of the 4 boards and then add JQL to pick out the issues with each label. Then we would have to 
repeat the exercise once someone decided that 'My-Important-Label' also deserved a filter/swimlane.

I've mentioned swimlanes so far, similarly to add filters for components and labels you need to enter a JQL query for 
each for each copy of the board. 

Also, we have reworked our workflow a few times, which means adding and removing states. Imagine all the work that would be
for the combination of things we have discussed so far. And I have skipped other things that we are also interested in such as:
Fix Version, Type, Priority, and whatever custom fields we are using.


<h2>External tools</h2>
Despite Jira Agile's problems, it has a nice feature that the data is the issue. Other tools seemed too 'board-centric'
so that if you created a board the issue 'belonged' to that, and being able to change the view wasn't really possible
(e.g. to go from a plain board to one using swimlanes by component). I evaluated a lot of tools, and might have missed
some, but I didn't really find what we were after.

Also, for the tools that promised to integrate with Jira, the integration didn't seem that instantaneous and brittle in
several cases.

<h2>Introducing Overbård</h2>
After some discussion we decided to investigate implementing a read-only board that could display 
the data better than what we found. The requirements were:
* To properly display a lot of states, we did this via:
** Horizontal scrolling
** Collapsible headers
* To avoid configuration to provide filters and swimlanes. Instead we inspect the data that exists in the issues to 
provide the options for filters/swimlanes
* Do as much work on the client as possible to make it fast. All filtering and other massaging of the view happens
on the client.
* Integrate tightly with Jira. We wrote it as a Jira plugin to have full access to the Jira SDK.

To play with this, check out our read-only <a href="https://overbaard.github.io/demo" target="_blank">demo</a>, or 
take a look at the below animation:

<video autoplay="autoplay" loop="loop" width="400" height="300" align="center">
  <source src="/assets/images/posts/2019-02-27-why-we-wrote-overbaard/animated.mp4" type="video/mp4" />
  <!-- <img src="video.gif" width="400" height="300" /> -->
</video>
  

Thanks for reading!








