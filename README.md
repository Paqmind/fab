# fab

Frontend Architecture Brainstorming

## Data Load

### Overview

One of the biggest unsolved questions.

#### Proposed solutions

1. graphQL + Relay 
2. Falcor

#### Speculation 1 

Declarative implementation should be built upon imperative one.<br/> 
The problem: no viable imperative implementation were found.

#### Speculation 2 

ORM history. We can compare HTTP data load with ORM (SQL) data load.<br/> 
With the first one being harder (distributed system).

SQL ORM answers: what to load, how to load<brs/>
HTTP "ORM" answers: what to load, how to load, from where to load

If we agree ORM is kinda failed idea we really should be super dubious about GraphQL / Relay. 

Something can work for simplest cases but fail (require ugly drawbacks / workarounds) for real ones.

#### Example 

Frontend DB has 4 "partial" models, 2 "missing" and 4 "ready-to-use". The system needs to request
missing data... but how exactly? There are a few options. We can request 6 "not-ready" models in one request
or we can request 2 "missing" with one request and 4 "partial" with another. Or we can request all 10 
in one shot to "refresh" those 4 "ready-to-use" (as we have an opportunity). It depends on our criterias, our backend.
We may be not in control of all the criterias.

In HTTP-1.1 we always strive to minimize requests.<br/> 
Which may be counter-productive with HTTP-2.0.

**The question:** network channel or browser memory will be a real bottleneck? Or both?

### Load strategies

Intuitions about laziness are applicable here:

https://github.com/maxsnew/lazy

> Maybe you have 100 different graphs you want to show at various times, each requiring a decent amount of computation. Here are a couple ways to handle this:

> 1. Compute everything up front. This will introduce a delay on startup, but it should be quite fast after that. Depending on how much memory is needed to store each graph, you may be paying a lot there as well.

> 2. Compute each graph whenever you need it. This minimizes startup cost and uses a minimal amount of memory, but when you are flipping between two graphs you may be running the same computations again and again.

> 3. Compute each graph whenever you need it and save the result. Again, this makes startup as fast as possible fast, but since we save the result, flipping between graphs becomes much quicker. As we look at more graphs we will need to use more and more memory though.

> All of these strategies are useful in general, but the details of your particular problem will mean that one of these ways provides the best experience. 

Cool. Now to the data load.<br/> 
We have at least four possible strategies. All Benefits and Drawbpacks below are **possible**.

#### 1. Full preload. 

All required data is loaded upfront. Frontend-only: filtering, pagination, etc.<br/>
The old-school jQuery architecture.

Benefits: UI performance

Drawbacks: memory, initial load delay

Suitable for: small data, games

#### 2. AJAXy

Data is loaded for every render cycle. Backend-only: filtering, pagination, etc.<br/>
The old-school "backend-driven" architecture.

Benefits: minimal memory usage (a waste of browser resources actually).

Drawbacks: UI performance, network flood

Suitable for: ???
   
#### 3. Load & Cache combine strategies (a family of them)

Smart solution. Arguably supersedes 2. (but not 1.)

#### 4. Above + garbage collection (can free memory). 

Tough one. Arguably supersedes 3.

---

Realistic app will probably require a mix of strategies for different datasets.

### Components

In React everything is component-centric.

React team actually promotes two mutually exclusive ideas: 

1. Component is (mostly) for DOM representation (answers JSX, answers component as a `render` function)
2. Component is not (only) for DOM representation (React Component vs Web Component, marketing duel with Google)

Problem with DOM-centric approach: browser is not limited to DOM. 

Imagine browser used to mine bitcoins.<br/>
Data comes from HTTP. Data goes to HTTP. <br/>
**Nothing** is rendered at all. 

Today we have tons of APIs which are not DOM-specific or are even not DOM-related<br/>
Audio APIs. Canvas. WebGL. Service Workers... – just for example.

Imagine an invisible player. To play an unstoppable background noise.
In React you *can* represent such a player as a Component.
But you *must* put it in the component tree.

**Thus implying it's a VDOM-related. Which it is not.**

So it seems a Frontend App can be better viewed as a *real* app and a Browser as an OS.

No available React architecture is fractal (because component can't be reused as an app).<br/>
I believe React components were called so mostly to strike Google's backed WebComponents.<br/>
The chosen imperative OOP-style is not enough to emulate an app.

In CycleJS components are **real** mini apps. Behold:

**Intuitions:**

```hs
-- Haskell early API
main :: [Request] -> [Response] -- lists are lazy so stream-like
```

```hs
-- in other words
main :: Stream Request -> Stream Response
```

so anything which wants to be plugged into main "loop" is

```hs
component :: Stream Request -> Stream Response
```

Compare to CycleJS

```hs
main      :: Observable a -> Observable a
component :: Observable a -> Observable a 

-- or

main      :: Map String (Observable *) -> Map String (Observable *) 
component :: Map String (Observable *) -> Map String (Observable *)
```

Unless I'm wrong: Haskell changed this API to monadic due to static typing complexities (note the `*` above).
`*` was "solved" through Union types hardcoded to the language (so not extendable).
Due to dynamic typing, we don't have this type of problems in JS .
