# robot_workshop

## Preparation

Follow [this section](https://code.visualstudio.com/docs/devcontainers/containers#_installation) to get vscode or cursor setup with the devcontainer extension.

Then open this folder in vscode, and a pop up for opening the workspace in a devcontainer will show.

This will download and install a bunch of packages needed for what we want to do.



## Technology

<details>
    <summary>Why Git?</summary>
    <p>Git is a tool that helps us keep track of changes in our code. It allows us to save versions of our work, collaborate with others, and go back to previous versions if something goes wrong. Think of it as a time machine for your code!</p>
    <details style="margin-left:20px;">
        <summary>Why do we want version control?</summary>
        <p>Version control helps us keep track of every change we make to our code. If we make a mistake, we can go back to an earlier version. It also makes it easy to work with others, since everyone can see what changes have been made and when.</p>
    </details>
    <details style="margin-left:20px;"=>
        <summary>What does "adding" mean?</summary>
        <p>When you "add" a file in Git, you're telling Git to start tracking changes to that file. This is the first step before saving your changes (committing).</p>
        <pre><code>git add filename.txt
</code></pre>
        <p>Or to add all changed files:</p>
        <pre><code>git add .
</code></pre>
    </details>
    <details style="margin-left:20px;">
        <summary>What does "committing" mean?</summary>
        <p>"Committing" means saving a snapshot of your changes in Git. Each commit is like a checkpoint you can return to later. You should commit often, with a short message describing what you changed.</p>
        <pre><code>git commit -m "Describe what you changed"
</code></pre>
    </details>
    <details style="margin-left:20px;">
        <summary>What is branching?</summary>
        <p>Branching lets you work on new features or experiments without changing the main code. You can try out ideas safely, and later combine (merge) your changes back into the main branch if you want.</p>
        <p>Create a new branch:</p>
        <pre><code>git checkout -b my-feature-branch
</code></pre>
        <p>Switch to an existing branch:</p>
        <pre><code>git checkout main
</code></pre>
    </details>
    <details style="margin-left:20px;">
        <summary>How do I make a pull request on GitHub?</summary>
        <p>A pull request is a way to ask for your changes to be added to a project on GitHub. You push your branch to GitHub, then open a pull request. Others can review your code, discuss it, and approve it before it gets added to the main project.</p>
        <p>Push your branch to GitHub:</p>
        <pre><code>git push origin my-feature-branch
</code></pre>
        <p>Then, go to the repository on GitHub and click "Compare & pull request" to start the process.</p>
    </details>
</details>

<details>
    <summary>Why Docker?</summary>
    <p>Docker lets us package our code and all the things it needs to run (like libraries and tools) into a single container. This means our code will work the same way on any computer, making it easier to share and set up.</p>
</details>

<details>
    <summary>Why Devcontainers?</summary>
    <p>A devcontainer is a special setup that uses Docker to create a ready-to-use coding environment. When you open this project in VS Code with the devcontainer extension, it automatically sets up everything you need, so you can start coding right away without worrying about installing lots of software.</p>
</details>

<details>
    <summary>Why ROS 2?</summary>
    <p>ROS 2 (Robot Operating System 2) is a set of tools and libraries that help us build robot applications. It makes it easier to control robots, process sensor data, and make robots do useful things. ROS 2 is widely used in both research and industry.</p>
</details>

<details>
    <summary>Why Simulation?</summary>
    <p>Simulation lets us test our robot code in a virtual world before trying it on a real robot. This is safer and faster, and helps us find and fix problems early. In this workshop, we'll use simulation to see how our code would work on a real robot, without needing any hardware.</p>
</details>