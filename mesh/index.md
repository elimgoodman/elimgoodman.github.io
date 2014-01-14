---
layout: page
title: Mesh
---
{% include JB/setup %}

#Overview

The design intention of Mesh is to combine the best features of modern IDEs (autocomplete, integrated debugging, powerful refactoring tools) with the power, flexibility, and "user-hackability" of Emacs. The end goal is to enable proficient programmers to create and modify programs with greater efficiency, less risk, and less frustration than traditional text-based tools. 

##Motivating Principle

As programs grow, they become riskier to change. However, the majority of programming involves changing existing programs. Even when writing something from scratch, programmers are constantly engaged in the act of discovering more about the domain being modeled, which necessitates altering the program being written to better reflect reality. If programmers can't make large-scale changes with confidence, programs enter a complexity "death-spiral", wherein simplifying the program becomes too dangerous, so necessary simplifications don't occur, which only makes things more complex and harder to change, ad infinitum.

##Design Beliefs

1. Mental energy is the most valuable asset that programmers have - Mesh's number one goal is to help programmers avoid spending any of their precious mental energy on anything extraneous.
1. A tight coupling between the editor and the language being edited allows the computer to "understand" the program on a deeper level, and thus offer more assistance. 
2. The program being built, editing environment, and execution environment should be accessible as data to the host language, and should be manipulable as such. 
4. By imposing well-chosen constraints upon themselves, programmers can achieve more efficiency and clarity using less mental energy and incurring less risk.

##Explorations
- [Crash Course](/mesh/full-walkthrough.html): we walk through doing some basic tasks in Mesh
- [Concepts](/mesh/workspace.html): bridging the gap between editor and code 
- [Workspaces](/mesh/workspace.html): how the UI of Mesh environment is organized 
- [Tags and Triggers](/mesh/workspace.html): mechanisms to add safety incrementally