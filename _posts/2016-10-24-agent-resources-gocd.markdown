---
title:  "Agent Resources in GoCD"
categories: [GoCD, CI]
tags: [GoCD, CI]
---

Lately I've been working with [GoCD](https://go.cd). It's an open source continuous integration tool which has both driven me crazy and blown my mind in countless ways over the past 8 months or so. It isn't as well-known as Jenkins or Travis, but some of its core principles are fundamentally different to (and in my opinion, better than) any other CI tool I'm aware of.

[![]({{ site.baseurl }}images/gocd_agent_resources/interest.png)](https://www.google.com/trends/explore?date=2008-01-01%202016-11-15&q=GoCD,Jenkins%20CI,Travis%20CI,Circle%20CI,Codeship){: .center-image}
Google searches for popular CI tools
{: .caption}

The graph above paints a pretty accurate picture of this space. A few years ago, everyone (including us) was using Jenkins. There has been some effort to improve Jenkins (e.g. by adding a plugin for pipelines), but it has pretty much been succeeded by Travis. Circle CI is also proving to be a dominant force, and seems to be the service which is attracting all the cool kids.

Having moved to GoCD, I don't regret the decision. Even though there are at least 4 other tools which are more popular, I'm a firm believer in GoCD's vision of pipelines as a first-class citizen. While it still lacks in some areas like the UI, these are being actively worked on. The project is still young, and major new features are still being added in the monthly releases.


### Agents and Resources

GoCD operates with one server node and multiple worker nodes. The central server hosts the web interface and keeps track of pipelines and artifacts. Workers check in on a specific port, and ask for jobs to run. If you're thinking that this sounds like a cluster management system, you're right! In fact, we use the Docker containers provided by the GoCD team, and there has even been some initial work done on [elastic agents](https://plugin-api.go.cd/current/elastic-agents/).

This approach isn't unique to GoCD, but the tool does go a bit further with **agent resources**. Basically, each job can be tagged with things like `python3-dev`, `windows` or `libcurl`. While being just text, they can be extremely powerful. The server will only give a job to an agent if that agent also has those tags. I made an animation to demonstrate.

![]({{ site.baseurl }}images/gocd_agent_resources/arch.gif){: .center-image}

The white block in the lower right is the server node. The blue and purple blocks are agents with two different tags.

During a recent refactor of our CI system, I took the opportunity to redefine how these tags were set up, and what was installed on our agents. This was sorely needed because when we initially set our agents up, we had only just begun to play with Docker. We had two different agents which were both overly complicated, had large Dockerfiles and took hours to build. So I decided to hop over to the opposite end of the spectrum. I was going to adopt the microservice philosophy and build many agents, all of which performed one very specific function and therefore had small, easily manageable Dockerfiles. These 'micro-agents' were going to be *amazing*.

### What I learned

This scheme worked great for a while. I had agents for various languages, one that knew how to build Docker images, one that knew how to deploy to Google Cloud Platform, one that could build our documentation and more...

Then I came across a situation where I needed to build the documentation and deploy it in the same unit of work. This was impossible, because no one agent could do both tasks. While you generally want to split up building and deploying, sometimes it just doesn't make sense. GoCD is quite particular about what pipelines/stages/jobs should be, and swimming against that current is a *very* bad idea. I know this from experience, so rewriting my pipelines by splitting jobs up wasn't an option.

It became clear that there's a trade-off. You either get manageable agents or sane pipelines. Not both.

Going forward, I will always recommend that the pipelines configuration comes first. Design them without thinking about agents, and work backwards from there. Afterwards, if there are groups of jobs which are clearly mutually exclusive, then create an agent type for each one. Even with this approach, you might find that there are a few things which *all* agents need. Docker's layers can help you out here, because you can define a general 'base agent' which the other types build off.

Here's a diagram of the agents that I ended up with.

![]({{ site.baseurl }}images/gocd_agent_resources/agents.png){: .center-image}



### PS

GoCD also has a concept of 'environments', which function similarly to resources. I did try to make use of the feature, but it didn't quite suit my use case.
