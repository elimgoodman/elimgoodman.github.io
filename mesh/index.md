---
layout: page
title: Mesh
---
{% include JB/setup %}

#Overview

The design intention of Mesh is to combine the best features of modern IDEs (autocomplete, integrated debugging, powerful refactoring tools) with the power, flexibility, and "user-hackability" of Emacs and Smalltalk. The end goal is to give proficient programmers the ability to craft *an environment* that will allow them to create and modify programs with greater efficiency, less risk, and less frustration than traditional text-based tools. 

##Motivating Principle

As programs grow, they become riskier to change. However, the majority of programming involves changing existing programs. Even when writing something from scratch, programmers are constantly engaged in the act of discovering more about the domain being modeled, which necessitates altering the program being written to better reflect reality. If programmers can't make large-scale changes with confidence, programs enter a complexity "death-spiral", wherein simplifying the program becomes too dangerous, so necessary simplifications don't occur, which only makes things more complex and harder to change, ad infinitum.

**Mesh aims to enable us to manipulate programs that are of progressively greater levels of complexity, and counteract the complexity death spiral.**