# Typescript and Test Driven Design don't fix everything, but... 

But they make you think differently about the issues at hand. 

For a long time, I thought that I did not have the time to write tests or that I'll simply do it later. It took me a few years to see how my thinking was completely backwards.

Let me quickly draw you a picture...

## I'm a slow adapter
When react made its big splash alongside angular, I had a look at it but decided to stick with vanilla Javascript for the time being. When redux added a convenient solution for managing state and storing data, I was still using local storage and other patterns for a while.

When I started using react and redux in tandem, I had a look at typescript and decided to stick with the jsx variant for the time being. And don't even get me started on test-driven design...

## What has changed? 
For once, it wasn't that I was totally against those libraries, languages and workflows. I just don't jump the wagon on the first mention of a new thing on the developers' horizon, an attitude that kept me save from framework fatigue during the recent years. 

Another reason is reality. If you are not a student, working for one of the few big IT giants or do your own thing, it is very likely for you to be locked in with some legacy code base, monolithic projects or tight deadlines. 

Starting to work with a new framework/library or even language (typescript being a superset of javascript) Might not even be possible for most of us unless we do our own projects after work or are in the luxury position to get some leeway from our bosses.

I was already comfortable with react and redux by the time I started working at Das Büro am Draht, a few years ago. We sat together as a team and later in one of our workgroups and discussed the merits of working with typescript. Management gave us time to both learn and practice working with typescript and agreed on slower deliveries while we started climbing the learning curve until we were comfortable using it for all new development.

Test-driven design is a different story altogether though. While typescript comes with a certain amount of overhead, it becomes manageable and doesn't impact development times after a while. Test-driven development on the other hand... First of all, it doesn't make sense to use it everywhere. Do you plan to test if your component actually renders that div? Do you repeat the "red until green" Cycle for every component you are going to write? Well, I don't.

Additionally TDD means you need to spend more time while developing new stuff for the first time, hacking away and writing tests later allows you to ship faster and... Probably never write those tests in the end, unless your project ever switches over from active development into a maintenance/support mode. 

TDD is hard to sell to management or your project manager and in most cases, it's even harder to get the client to agree unless you simply don't tell him and do it silently.

But today is different. I'm here today to tell you that TDD has its uses and pairs very well with typescript too. Why? It makes you think.

I can't show you our customers' business logic here but I'll try to explain it on a private project I do after hours for fun. In the following examples, I'm setting up the basics to a pseudo-AI that determines the results of a decision-making model based on configurable sets of parameters. It's basically a way to tell your heroes in a game how to pick targets and choose skills and spells based on predefined rules.


Usually, I would sit down and simply write the basics and hack away from there. If it works, keep it and expand it. If not, discard and try again. I've previously written about rapid prototyping and I'm still a big fan but this time I have a very clear idea about what I want to end up with so I can sit down and properly plan ahead using the previously mentioned tools.

I'm dividing my process into the following steps:
- paper prototype planning 
- Empty functions
- Typescript types
- Tests (red)
- Actual code
- Green test check 

This process makes me think about what type of data I'm handling, how to process and transform it and this is also a good way to structure your code before you write it. Preparing coding sessions, tutorials and seminars, this has come in handy as well.

> If you have no clue what I'm talking about when I mention gambits or unit behaviour rules, think of characters in a strategy / role playing game that need to decide what to do next based on rules. The enemy might attack the player or use a healing spell when his hitpoints are below a certain level... that sort of thing.

What do we need for this example? I spell it out and write it down. This is a step I often do on paper or my smartphone way before I write the first line of code.

### 1) Paper Prototype
- the player can create and adjust gambits that will control his units
- gambits will be checked one after another until a condition is met
  - get all targets matching the selection
  - filter them down based on the condition
  - see if the skill/attack is available
  - return the action and target or check the next gambit 

Our next step is to put this general idea into functions we could implement.

### 2) Empty Functions
```javascript
/* processBehaviourChain()
 * - check all gambits in the behaviour chain for a match
 */
 
/* getAvailableTargets()
 * - get all targets matching the target type
 */

/* getMatchingTargets()
 * - return only targets matching the selection criteria
 */
 
/* isActionAvailable()
 * - check if the action is possible 
 * - (enough resources, material and so on)
 */
 
/* compareTargetHP()
 * - check if the current target matches the hitpoint condition
 */
 
/* compareTargetStatus()
 * - check if the current target matches the status condition
 */
```

As you can see, from reading this rough sketch alone you already get a pretty good idea of what will happen when and why. All functions have fitting names (some may change during implementation anyway) and you also get a good idea about what these functions will need to do their job.

### 3) Typescript Types
```typescript
// We need units to compare and some stats and status to check. 
// I will skill the imported 'Effect' and 'RaceName' types here
type UnitBase = {
    hp: {
        current: number;
        max: number;
    };
    status: Effect[];
};

export type Unit = UnitBase & {
    name: string;
    id: string;
    race: RaceName;
};


// possible selections for targeets and derived types
const targets = [
    "self",
    "currentAlly",
    "currentEnemy",
    "leader",
    "leaderCurrentEnemy",
    "leaderCurrentAlly"
] as const
type Target = typeof targets[number];

const conditionTargets = [
    ...targets,
    "ally",
    "enemy",
] as const
type ConditionTarget = typeof conditionTargets[number];

/*
 * skipped a few more types here including a selection of 
 * valid absolute numbers and percentage values as well as 
 * our previously used Effect type
 */

// comparison operators for numerical values like hitpoints
const comparisonOperators = ["lt", "lte", "eq", "gte", "gt"] as const
type ComparisonOperator = typeof comparisonOperators[number];

type PercentageComparison = [ComparisonOperator, Percentage];
type AbsoluteComparison = [ComparisonOperator, AbsoluteValue];
type StatusComparison = [boolean, Effect];

// last thing to do here is a simple mapping of valid combinations
type TargetComparison =
    | ["hp", AbsoluteComparison]
    | ["hp%", PercentageComparison]
    | ["mp", AbsoluteComparison]
    | ["mp%", PercentageComparison]
    | ["status", StatusComparison]
    
type TriggerCondition = [ConditionTarget, TargetComparison];

type Gambit = {
  target: ConditionTarget;
  comparison: TargetComparison;
  action: Spell | Skill | Item;
}
```

Let's put those types to good use by creating our functions.

```javascript
const processBehaviourChain = (units: Unit[], gambits: Gambit[]): void => {
  // check all gambits in the behaviour chain for a match
}
 
const getAvailableTargets =(units: Unit[], targetType:ConditionTarget): Unit[] | null => {
  // get all targets matching the target type
} 

const getMatchingTarget = (targets: Unit[], comparison: TargetComparison): Unit | null => {
  // return only targets matching the selection criteria
}
 
const isActionAvailable = (actor: Unit, action: Action): boolean => {
  // check if the action is possible 
  // (enough resources, material and so on)
} 
 
const compareTargetHP = (target: Unit, comparison: TargetComparison): boolean => {
  // check if the current target matches the hitpoint condition
} 
 
const compareTargetStatus = (target: Unit, comparison: TargetComparison): boolean => {
  // check if the current target matches the status condition
} 
```

As you can see, we have already built a lot of confidence in our code just by defining types. What do we want from the tests now? If we produce an output that does not match our types, the typescript compiler will already catch this. We don't write tests to reach a certain level of code coverage, we will write tests to check in those nooks and corners where typescript can't look. We will check for validation issues with user input, application of filters (like getAvailableTargets()) and complexity levels inside our functions including whether or not certain subfunctions have been called and make sure that edge cases and such are caught correctly. 