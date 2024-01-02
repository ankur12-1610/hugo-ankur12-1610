---
title: "My LFX Mentorship Journey with CNCF: Kubernetes"
read_time: true
date: 2023-06-03
cover:
    image: "https://cdn.hashnode.com/res/hashnode/image/upload/v1685738766904/c5e1d5c3-b0ad-4a8a-bd3f-1475b207100d.png?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "journey-lfx"
    relative: false # To use relative path for cover image, used in hugo Page-bundles
tags:
    - LFX Mentorship
    - Open Source
    - Kubernetes
    - CNCF
    - Go Language
---

I have always wanted to participate in the [LFX Mentorship program](https://mentorship.lfx.linuxfoundation.org/) and here I am, graduated, writing this blog. These 3 months were amazing! This blog covers my experience in this mentorship.

### What’s the LFX Mentorship?

> *“The Linux Foundation Mentorship Program is designed to help developers — many of whom are first-time open source contributors — with necessary skills and resources to learn, experiment, and contribute effectively to open source communities. By participating in a mentorship program, mentees have the opportunity to learn from experienced open source contributors as a segue to get internship and job opportunities upon graduation.”*

If you are interested in participating in the LFX Mentorship programs, you can find more information at [lfx.linuxfoundation.org/mentorship/guide](http://lfx.linuxfoundation.org/mentorship/guide).

### My acceptance into the program

Once the application process began, I drafted a cover letter and I was contributing to [TestGrid](https://mentorship.lfx.linuxfoundation.org/project/ca622980-cc8c-4f18-8a74-b9a7b4b49e3a) for some time which I included in my cover letter, which I believe helped me stand out from other candidates.

I want to emphasize that contributions matter a lot. If you are interested in a particular organization, I encourage you to find ways to contribute to their community or projects. This is a great way to show your interest in the organization and your ability to add value.

I applied to 3 organizations (**Kubernetes, Harbor and Cortex**), Kubernetes accepted me first, so my applications to the other two organizations were no longer considered.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685733474885/cc6f5ca5-bc11-4ace-abbf-e90cdb3c4a18.png )

### So, what did you do during mentorship?

I contributed to [**CNCF - TestGrid: Frontend development inside Lit Component Framework**](https://mentorship.lfx.linuxfoundation.org/project/ca622980-cc8c-4f18-8a74-b9a7b4b49e3a) project.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685733733861/3e42d773-7c2c-41f4-be92-7f4e671602c8.png )

TestGrid's Core contributors [**Michelle Shepardson**](https://github.com/michelle192837) and [**Sean Chase**](https://github.com/chases2) were my mentors. I had weekly sync-up meetings with them. In this meeting, we reviewed the goals we set in our previous meetings, discussed the progress, reviewed PRs I made, and identified any new goals or objectives that I would like to work towards.

**Stepwise progress:**

**Frontend**:

The first step was to set up an existing webapp with Tailwind, after searching it was found that having normal plain old CSS is suited for Lit Framework. Therefore, we decided to build our components on top of plain old CSS.

Next, I implemented the index of TestGrid which rendered dashboard and dashboard groups components. The image shows the end result of the index.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685734786024/e99d702f-86f7-4bcd-afa9-9d0c05aa2a9d.png )

Implemented the onClick functionality to render particular dashboards when clicked on a dashboard group. Refer to this image:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685734896030/9aa1069c-85a3-4577-b77e-a7a675d73738.png )

After implementing the index, Sean asked me to write StoryBook and Unit tests, following the Test-Driven Development (TDD) methodology. This was my first time writing Storybook tests, and I found it to be a fun and rewarding experience.

I also decluttered the codebase by removing any unused or duplicate code.

The final task which revolved around fronted was to reiterate and improve the written tests.

**Backend**:

After, implementing the index, there was some problem in the API structure which made the frontend do the heavy work. Hence, compromising the user experience.

After discussing it with my mentors we decided that the API structure needs to be improved and I was set to start my GO journey, was really excited!

I really want to thank Michelle and Sean (my mentors) as they helped me get familiar with every piece of information related to API and GO. Before actually working on the codebase, I read all the resources first and got familiar with protubufs and GO.

There were two tasks assigned to me;

**Task1: (refer to issue** [**#1199**](https://github.com/GoogleCloudPlatform/testgrid/issues/1199))

The frontend required to do heavy work because the dashboards were not linked to their respective dashboard groups. The solution was simple, just add a reference to every dashboard of their dashboard group. I updated the API proto, wrote the governing functions, and finally updated the tests.

**Task2: (refer to issue** [**#1200**](https://github.com/GoogleCloudPlatform/testgrid/issues/1200))

Currently, TestGrid doesn't have an endpoint that shows the config in a short and clear way. The information we get related to a particular tab is by fetching headers or rows. The end goal was to add a new endpoint that shows all the valid and important information related to the dashboard tab.

I started by creating a new resource in the API proto. Later on, I added the governing functions which fetch the equivalent data. Finally registered a new endpoint to the server.

### What I Achieved and Scope for Improvement

I was having real fun working on this project and have **closed 5 PRs and have 1 PR in progress**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685736705764/fe6277b1-7176-4de3-8c76-a5801e67774a.png )

Currently, the Task2 PR is still a work in progress, my goal is to complete the task and get the WIP PR merged.

I am also looking forward to contributing to Kubernetes in the future.

### Is it difficult to get accepted into this?

I would say a big NO!!!

I personally started checking out the projects a few weeks before the applications started.

*It is all about how willing are you to contribute to the community and learn new things* which at the start might seem a little bit overwhelming. Before LFX, I was a beginner in GO but during my mentorship period, I worked on an API written in GO.

Don't hesitate, give it a try. You will learn a lot!

> If you’re interested in joining the LFX Mentorship like I was, [**submit your proposal here**](https://mentorship.lfx.linuxfoundation.org/#projects_accepting)!

### Graduation and concluding it all!

After 12 weeks, I successfully graduated from the program-

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685737004421/bfa55c44-e890-4b31-bd42-9eb5507d0c57.png )

I'm really grateful to my mentors [**Michelle Shepardson**](https://github.com/michelle192837) and [**Sean Chase**](https://github.com/chases2) for helping me throughout the project. Additionally, a big thanks to [**Sultan Duisenbay**](https://github.com/sultan-duisenbay)**,** who helped me a lot as well!

I would like to express my sincere gratitude to the Kubernetes, LFX, and CNCF communities. Without their support, neither this project nor this program would have been possible.

For doubts and questions feel free to contact me: [**LinkedIn**](https://www.linkedin.com/in/ankur-patil-a112a3202/) **|** [**Gmail**](http://mailto:ankur.patil.012@gmail.com/)

Thank you for reading, hope you enjoyed the article! Follow me on [**Twitter**](https://twitter.com/ankurrap) | [**LinkedIn**](https://www.linkedin.com/in/ankur-patil-a112a3202/) for more development-related posts. That's all for today! Thank you for reading :)