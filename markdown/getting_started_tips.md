# Getting Started with Full Stack

## Find a mentor

Pair up with someone who's already done full-stack work. They'll save you hours of guessing at patterns and conventions. Seriously — the amount of tribal knowledge in this stack is non-trivial.

## Start with low-pressure work

Pick non-time-sensitive issues for your first full-stack attempts. The learning curve is real, and figuring out SQLBoiler for the first time under a deadline is not a good experience.

It's also useful to communicate to your manager that you expect this learning curve. This helps make the situation a bit less stressful.

## Understand the code you write

Writing code is getting easier. Understanding it is still on you. Even if a skill generates your DAO or handler, read through what it produced. You'll catch errors faster and build intuition that compounds over time.

## Don't bite off more than you can chew.

Similar to the above, it's good to be gradual about this learning curve. If you're new to all this, try implementing the proto with stubs and instead of having the skill do the complete implementation. Once you understand the proto and the basic API server layer, you can go a level deeper to the RPC and then to the database layer. This is a gradual process. I myself don't understand a ton beyond the API level layer, and even then my understanding is not that deep. I know things are changing really quickly, but we should still remember that we're humans and that this work is difficult. We're not going to master it in a day.

## Use code review heavily

Before March 20th: lean on the code review skill as much as possible. After that deadline: make sure a human is reviewing your code. Either way, don't ship what you haven't had reviewed.

## Fullstack knowledge can improve your frontend code

When you work across the stack, it becomes easier to notice certain improvements. For example, you'll be better aware of how to optimize the API for your frontend use case, and you might even bring some frontend best practices to the backend.

For example, after a bug fix related to a path being inputted incorrectly in the API server field masks, I thought, "Why isn't this typed?". I added a package that allowed for typesafe field masks.

## Backend work can be heavier on your computer

Backend work hits your machine harder than frontend. Building the API server alone can take a while, sometimes more than 10 minutes and can even slow your computer down. If you're using the deployment skill, it should be smart enough to only build the services you actually need.

## Don't be hard on yourself

The backend engineers who started doing frontend work weren't great at it immediately either. They got better. You will too.
