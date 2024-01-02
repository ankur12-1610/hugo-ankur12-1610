---
title: "Build your own Github Action and publish to the Github Marketplace!"
read_time: true
date: 2022-01-25
cover:
    image: "https://cdn.hashnode.com/res/hashnode/image/upload/v1643109105478/H-HnvafL8.png?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "github-actions"
    relative: false # To use relative path for cover image, used in hugo Page-bundles
tags:
 - github-actions
 - github
 - automation
---

### Introduction:

Automation, complexity reduction, reproducibility, and maintainability are all advantages that can be realized by a continuous integration (CI) pipeline. With GitHub Actions, you can build these CI pipelines.

> "Automate, customize, and execute your software development workflows right in your repository with GitHub Actions. You can discover, create, and share actions to perform any job you'd like, including CI/CD, and combine actions in a completely customized workflow." - [Github Docs](https://docs.github.com/en/actions)

We all use Github Actions for code analysis, adding awesome Readme features and doing many other things. When I was first introduced to Github Actions I was dumbstruck and from then onwards it became my favorite section to explore. I started exploring it, used it in my profile Readme and it was really awesome.

But the problem was I was using the products which were created by others and it kind of hurt me😅. So I was like - "Let's create a Github Action on own and then publish it in the marketplace so that others can use it". And guess what I did, you can check it on Github. In this blog we are going to make a Github Action so tighten your seat belt, it'll be fun.

<iframe src="https://giphy.com/embed/BpGWitbFZflfSUYuZ9" width="380" height="300" class="giphy-embed"></iframe>

### Prerequisites:

* A Github account
    

### Steps:

* Create a Github repository with a license (any, I've used MIT License).
    
    ![createrepo.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643099582428/Rm7OkwDcT.png)
    
* Clone the repo to your local machine and open it in your favorite IDE.
    
* Setup a node project with:
    

```plaintext
npm init
```

and install all the required packages by following commands:

```plaintext
npm i @actions/core
npm i @actions/exec
npm i @actions/github
npm i @vercel/ncc
```

`@vercel/ncc` is a Simple CLI for compiling a Node.js module into a single file.

* After installing packages make sure to add this script in `package.json`:
    

```plaintext
  "scripts": {
    "build": "ncc build src/main.js -o dist"
  }
```

This script is important as it runs `@vercel/ncc` which compiles all the files to a single file `dist/index.js`.

* Now, we create an `action.yml` file in our repository. The `action.yml` is the metadata file and defines the inputs, outputs, and main entry point for our Action.
    

The `action.yml` file has to include:

* `name`: The name of your Github Action.
    
* `description`: A short description of what your action is doing.
    
* `author`: The name of the author who created this Github Action.
    
* `inputs`: The input parameters which you can pass into your script.
    
* `runs`: Defines what the action will execute (in our case, it'll run docker).
    
* `branding`: How your Github Action will look like when it is launched in the marketplace.
    

The `action.yml` file will look like-

```plaintext
name: Demo Issue Action 🛰
description: Comments on the newly opened issue.
author: ankur12-1610 <anknerd.12@gmail.com>

inputs:
  GITHUB_TOKEN:
    description: Github token
    required: true
  ISSUE_COMMENT:
    description: Comment to be added on opening an issue
    required: true

branding:
  color: blue
  icon: tag

runs:
  using: 'docker'
  image: 'Dockerfile'
```

* Now, create a folder namely `src` and in that create a file `main.js`. This file includes the main script of the action, it is like the brain of the action.
    
* In our case, we are creating a Github Action which will comment on every issue created. For this, we need `@octokit/rest` npm package. It is the GitHub REST API client for JavaScript.
    

> For its API endpoints and other details go through its [official docs](https://octokit.github.io/rest.js/v18).

* So my `main.js` will look like this:
    

```plaintext
const core = require('@actions/core')
const github = require('@actions/github')

export async function run() {
    try {
    const { context } = require('@actions/github')

    //taking input from the core
    const GITHUB_TOKEN = core.getInput('GITHUB_TOKEN')
    const ISSUE_COMMENT = core.getInput('ISSUE_COMMENT')
    
    if ( typeof GITHUB_TOKEN !== 'string' ) {
      throw new Error('Invalid GITHUB_TOKEN: did you forget to set it in your action config?');
    }
    
    if ( typeof ISSUE_COMMENT !== 'string') {
        throw new Error('Invalid ISSUE_COMMENT field, it should be a string field')
    }
    const octokit = github.getOctokit(GITHUB_TOKEN) 
    const { issue } = context.payload
    const payload = context.payload.issue
    //comment on issue
    await octokit.rest.issues.createComment({
      ...context.repo,
      issue_number: issue.number,
      body: ISSUE_COMMENT,
    })

  } catch (e) {
    core.error(e)
    core.setFailed(e.message)
  }
}

run()
```

* The last step would be to create a file namely `Dockerfile`.
    

> If you are not familiar with Docker and Dockerfiles, check out [Dockerfile support for GitHub Actions](https://docs.github.com/en/actions/creating-actions/dockerfile-support-for-github-actions).

The `Dockerfile` will look like:

```plaintext
FROM node:latest

COPY . .

RUN npm install --production

ENTRYPOINT ["node", "/dist/index.js"]
```

That's it. We've created our won Github Action. Now to test it, create a workflow file in `.github/workflows` and this code:

```plaintext
name: 'Demo Issue Action 🛰'

on:
  issues:
    types:
      - opened

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: 🔃 Install
        run: |
          yarn
      - name: 🧱 Build
        run: |
          yarn build
      - uses: ./
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ISSUE_COMMENT:  'thanks for raising the issue'
```

### How others can use my Action?:

The answer is simple just paste the following code in the `.github/workflows` directory in required repo:

```plaintext

name:  'Demo Issue Action 🛰'

on:
  issues:
    types:
      - opened

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: ankur12-1610/demo-action@main
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ISSUE_COMMENT:  'thanks for raising the issue' #enter custom message
```

And now we are all perfect to launch it in the Github Marketplace.

### Publish the Action to the Marketplace:

* Go into the the `New Release` section and you will find this window:
    
    ![launch.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643108476459/G_Pp3FGQN.png)
    

Create new release tag, give release heading to the Action and publish it ;)

* Finally, we can find it on https://github.com/marketplace by searching "Demo Issue Action 🛰”.
    

![marketplace.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643108651599/bk-ehQas_.png)

### Conclusion:

Thank you for reading, hope you enjoyed the article! Queries and feedback are most welcome :) kindly leave them below.

Follow me on [Twitter](https://twitter.com/ankurrap) | [LinkedIn](https://www.linkedin.com/in/ankur-patil-a112a3202/) for more web development-related tips and posts.

That's all for today! You have read the article till the end.

<iframe src="https://giphy.com/embed/6oB3X3W6MYM3C" width="480" height="268" class="giphy-embed"></iframe>