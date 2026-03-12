# Full Stack Engineering at Rubrik — Speech Script

---

## 1. Intro

Hey everyone. Today I'm going to talk about full stack engineering at Rubrik — what it actually means, what tooling we've built to make it less painful, and some advice for getting started.

We'll go through the product engineer mentality shift, walk through our architecture, look at the new fullstack-orchestrator skill and some other useful skills, touch on debugging, and wrap up with practical advice.

---

## 2. Mentality Shift: The Product Engineer

Claude has added what's essentially an abstraction layer to our work. It's genuinely easier to operate in domains you're not deeply familiar with. And I think we've all seen that play out — backend engineers doing frontend work. Their early stuff is sometimes rough, but over time they've started contributing solid work.

The broader trend here is a shift from specialist to T-shaped product engineer. Instead of owning one layer and handing off to someone else, you deliver the whole feature. That has real benefits. You're not blocked waiting on a backend engineer because you are the backend engineer. You understand the intended product experience instead of just implementing a mock someone sent you.

This is the same process in the other direction. Frontend engineers picking up backend work will have a learning curve. That's expected. The tools are better now, and you'll improve faster than you think.

---

## 3. What "Full Stack" Means at Rubrik

Full stack here is not React and Node. A request comes in from the client — could be the UI, an SDK, or a direct API consumer. It hits our GraphQL API in the Scala api-server. That layer handles schema validation, RBAC checks, and routes the request through a resolver and DAL down to the Go gRPC handler. The handler is where business logic lives — validation, conversion, all of that. It talks to the DAO layer, which is an interface over SQLBoiler, and that ultimately hits a per-customer PostgreSQL database.

Six layers, three languages, dozens of files. Every new feature needs to wire through all of them. It's not conceptually hard, but there's a lot of surface area.

---

## 4. The Fullstack-Orchestrator Skill

So — six layers, three languages, dozens of files. The whole reason we built this skill is to make that as simple as possible. One entry point, one skill, and it handles the entire stack for you.

The skill is called `fullstack-orchestrator`. It's a one-stop shop.

You describe what you want to build and it plans the work with you. It asks clarifying questions, then routes to the right sub-skill — `add-dao-crud` for the data access layer, `add-rpc-handler` for the Go handler logic, `add-db-migration` for schema changes. Each sub-skill encapsulates its own reference patterns and conventions, so Claude's context isn't getting flooded with stuff it doesn't need for the current step. That's the main reason we split it up this way — avoiding context bloat.

You interact with one skill. You don't need to know which sub-skills exist or invoke them manually — the orchestrator figures out what's needed. New table required? It scaffolds the DAO. Table already exists? Skips that step.

I want to be clear — this is an MVP. A rough draft. It's going to need a lot of improvement from all of you to get it to a place where full stack development is as easy as possible. Think of all the work we've had to do on the frontend side to make development smooth — the skills, the patterns, the conventions we've built up. That hasn't really happened on the backend side yet. This is the starting point.

The detailed diagram shows the full seven-step flow: gather inputs, pick the right template, define the proto, run codegen, scaffold the DAO, add the handler, wire Scala and GraphQL, then deploy to your dev instance.

---

## 5. Other Useful Skills

A few other skills worth knowing about.

`/deployment` builds and deploys your services to a dev environment. It's smarter than a raw build — it identifies which services actually need rebuilding based on your changes, so you're not sitting through a 10-minute API server build you don't need.

`/bug-hunt` investigates bugs by tracing through code paths, recent changes, and related test failures. Give it a bug description or a JIRA ticket and it works through the relevant layers. Especially useful when something spans multiple services.

`/code-walk` is fast code navigation using Universal Ctags. It indexes Go, C++, and Python symbols across the codebase. Find definitions, list struct members, trace callers — roughly four times faster than grep-based lookups.

---

## 6. Debugging Tools

On the debugging side, two MCP servers are worth knowing about.

First, Logz. Think of it like the browser console, but for the backend. Every `log.info` and `log.error` across every service ends up in one searchable place. The Logz MCP lets you query all of that directly from Claude Code — Lucene syntax, filter by log level, time range, deployment, component. No context-switching to the Logz UI.

Second, Chronosphere. If Logz is your console, Chronosphere is your Network tab. It shows you server-side latency, error rates, and throughput — the same kind of information you'd get from the Network tab in DevTools, but from the server's perspective across all services. The Chronosphere MCP lets you query metrics directly from Claude Code too.

Both are especially useful when you're tracking down issues that span multiple services or only show up in specific deployments.

---

## 7. Advice for Getting Started

Some practical advice if you're getting into this.

Find a mentor. Claude and the guides will get you part of the way, but a mentor will make everything faster. There's still a lot of tribal knowledge at Rubrik

Start with low-pressure work. Pick non-time-sensitive issues for your first attempts. The learning curve is significant, and debugging a foreign issue under pressure is unnecessarily stressful. Another stress reduction tip: communicate to your manager that you expect this curve

Understand the code you write. Writing code is getting easier. Understanding it is still on you. Even if a skill generates your DAO or handler, read through what it produced. You'll catch errors faster and build intuition that compounds over time.

Don't bite off more than you can chew. Try implementing the proto with stubs instead of having the skill do the complete implementation. Once you understand the proto and the basic API server layer, go a level deeper to the RPC, then to the database layer. It's gradual. I don't understand everything beyond the API layer either, and my understanding isn't that deep. Things are changing quickly, but we should remember this work is difficult. We're not going to master it in a day.

Use code review. Lean on code review as much as possible — automated and human (even after the march 20th deadline).

Fullstack knowledge can also improve your frontend code. When you work across the stack, you notice optimizations you wouldn't have spotted before. You might bring frontend best practices to the backend, or optimize the API for your specific frontend use case.

---

## Other Resources

Here are some Confluence pages worth bookmarking. There's an end-to-end flow diagram that shows the full database request path including ProxySQL — useful for understanding what happens under the hood. There's also a step-by-step guide for adding basic APIs in Polaris — covers migration, proto, Go service, Scala GraphQL, the whole chain. And there's a page on getting access to a GCP dev deployment if you haven't set that up yet.

---

## Closing

You've probably seen backend devs ship questionable frontend code early on, then steadily improve. That's what your learning curve will look like too.

Thanks.
