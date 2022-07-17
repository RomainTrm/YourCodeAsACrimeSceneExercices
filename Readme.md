# Your Code As A Crime Scene

Here's my notes about the [book](https://pragprog.com/titles/atcrime/your-code-as-a-crime-scene/) written by Adam Tornhill.  
I am not the author of most of the content available on this repository. I've only regrouped on a single repository every resource I needed to follow the book.  

_Note_: every command prompt describe here must be executed on a bash command line.  

-----

## Chapter 1 : Welcome!

Tools needed for the examples (more details [here](https://adamtornhill.com/code/crimescenetools.htm)):

- [Code Maat](https://github.com/adamtornhill/code-maat) (requires Java >= 1.6)
- [Git](https://git-scm.com/)
- [Python](https://www.python.org/)

`scripts` folder contains python scripts for log analysis.  
`sample` folder contains various samples analysed all along the book.  

-----

## Chapter 2 : Code as a Crime Scene

Tool: [Code city](http://wettel.github.io/codecity.html)  
Each block is a package, each building a class. The height of a building is defined by the number of methods, and the base by the number of attributes.  

Tool: [MetricsTreeMap](https://github.com/adamtornhill/MetricsTreeMap)  
Size and color of blocks describe how frequently a piece of code is modified.  

By correlating those two dimensions (size of a class and how often it changes), we can identify hotspots that need our attention.  

-----

## Chapter 3 : Creating an Offender Profile

On this chapter, we run our first analysis on Code-Maat's source code.  

### Extracting data from repository

- Clone repo: `git clone https://github.com/adamtornhill/code-maat.git`
- Move to expected point in time: ``git checkout `git rev-list -n 1 --before="2013-11-01" master` ``. This command should place you on commit `d804759`.
- Getting log with stats: `git log --numstat`

### Automated mining with Code Maat  

- Mining logs:  `git log --pretty=format:'[%h] %an %ad %s' --date=short --numstat --before=2013-11-01 > maat_evo.log`
- data first inspection (evaluating number of changes): `maat.bat -l maat_evo.log -c git -a summary`
- change fequency analysis: `maat.bat -l maat_evo.log -c git -a revisions > maat_freqs.csv`

### Add the Complexity Dimension

Here, we're about to extract the number of lines of code as a complexity metric. As highlighted by the author, that metric is just as bad as any others. He chose it for now because it's langage agnostic and easy to extract.  

Tool used for counting lines of code: [Cloc](https://sourceforge.net/projects/cloc/).  
Extracting number of lines of code: `cloc ./ --by-file --csv --quiet --report-file=maat_lines.csv`

### Merge Complexity and Effort

Merging: `python scripts/merge_comp_freqs.py maat_freqs.csv maat_lines.csv`

-----

## Chapter 4 : Analyze Hotspots in Large-Scale Systems

### Clone Hibernate Repository

- Clone the repo: `git clone https://github.com/hibernate/hibernate-orm.git`
- Move to expected point in time: ``git checkout `git rev-list -n 1 --before="2013-09-05" main` ``. This command should place you on commit `a5705e011e`.

### Generate a Version-Control log

- Mining logs (note this time we also set a begining date to limit the scope we want to analyze): `git log --pretty=format:'[%h] %an %ad %s' --date=short --numstat --before=2013-09-05 --after=2012-01-01 > hib_evo.log`
- Genreate evolutions log: `maat.bat -l hib_evo.log -c git -a summary`

### Choose a Timespan for your analyses

- Between releases
- Over iterations
- Around significant events

### Mining Hibernate

Proceed as we did for Code Maat (Chapter 3), extract frequencies, the number of line of code, then merge both:  

- Change fequency analysis: `maat.bat -l hib_evo.log -c git -a revisions > hib_freqs.csv`
- Extracting number of lines of code: `cloc ./ --by-file --csv --quiet --report-file=hib_lines.csv`
- Merging: `python scripts/merge_comp_freqs.py hib_freqs.csv hib_lines.csv`  

### Explore the Visualization

The circle-packing visualization is from [D3.js](http://d3js.org), it's using the [Zoomable Circle Packing](http://bl.ocks.org/mbostock/7607535) algorithm.  
_Note_: this is just one tool among others, the author also highligts:

- Spreadsheets: an easy way to exploit CSV outputs.
- [R programming langage](http://r-project.org): a langage designed for statistical computation and data visualization.

Here we'll be looking at a prepared example; To launch hotspots visualization:

- Go into the `sample\hibernate` directory
- To avoid some security restriction issues on your browser, you have to run `python -m SimpleHTTPServer 8888` or `python -m http.server 8888` depending of your Python's version.  
- Then open `http://localhost:8888/hibzoomable.html`

To transform Code Maat's CSV output into a Json for D3.js, use: `python csv_as_enclosure_json.py -h`

-----

## Chapter 5 : Judge Hotspots with the Power of Names

For cognitive reasons, we put names on things to reduce the load and still express complex concepts: this is called chunking.  
Names can be good indicators to identify hotspots: are they descriptive (ex: `TcpListener`) or clumsy (ex: `StateManager`)?  
Combined with change frequency and number of lines of code, names can help us reducing the number of potential offenders.  
The author highlight this as an heuristic, it's not perfect and you can still have false positives.  

-----

## Chapter 6 : Calculate Complexity Trends from Your Code's Shape

Finding hotspot and acting on them may require several passes, so we need to look at the evolution of the code.  
Here we'll be using code's shape via indentation to mesure hotspots complexity and trends over time. Heavy indentation might highlight complex conditionnal flows.  

### Whitespace analysis of complexity

On the Hibernate folder used on the previous chapter:
`python scripts/complexity_analysis.py hibernate-core/src/main/java/org/hibernate/cfg/Configuration.java`  

### Analyze Compexity Trends in Hotspots

[Manny Lehman](https://en.wikipedia.org/wiki/Manny_Lehman_%28computer_scientist%29)['s laws of software evolution](https://en.wikipedia.org/wiki/Lehman%27s_laws_of_software_evolution): the more you change the code and add feature, the more code's complexity increases unless specific work is done to reduce it.  

Now, to analyze complexity trends over a period:
`python scripts/git_complexity_trend.py --start ccc087b --end 46c962e --file hibernate-core/src/main/java/org/hibernate/cfg/Configuration.java`  
Then you can use the spreadsheet of your choice to visualize trends with graphs.  

When total complexity increases, it can mean more indentation (and complexity) or more lines of code.  
Standard deviation (sd column) describes code consitency, the lower the better.  
For a high total complexity, a low standard deviation means a lot of code and a low overall complexity. Mean should return a similar trend.  

Complexity trend can be:  

- Increasing: That's a waring sign
- Decreasing: Some refactoring have been done to reduce complexity
- Stable: few modifications other time

-----

## Chapter 7 : Treat Your Code As a Cooperative Witness

Now, we're looking at high-level design of the system, we're chasing hidden dependencies and learning the concept of _temporal coupling_.  

Human brain suffers from a lot biases, the way you're asked some questions may result in different answers as the question influence memory access and associations, and even create false memories.  
That's why we need to extract real evolution of the code from the code base and our source control tools.  

You have _temporal coupling_ when modules change together, note that those modules may not have a static dependency visible through the compiler.  

-----

## Chapter 8 : Detect Architectural Decay

### Analyze Temporal Coupling

Sum of Coupling: looking at how many times a module has been coupled to anothers in a commit and sums it up.  
Mesure the Sum of Coupling: `maat -l maat_evo.log -c git -a soc`  
Mesure the Temporal Coupling: `maat -l maat_evo.log -c git -a coupling`  
The output is composed of:  

- entity: the coupled module
- coupled: the couterpart module
- degree: the percentage of shared commits where these modules are coupled  
- average-revs: a weighted number of total revisions for these modules

_Note_: you can have a high average-revs and a low degree, it means there's a lot of revisions, but only few are shared by the two modules. At the oposite, high degree and low average-revs means a strong coupling but stable modules.  

Suggested tool: [Evolution Radar](http://www.inf.usi.ch/phd/dambros/tools/evoradar.php)

Yet, Temporal Coupling suffers from limitations and biases. The code base may be maintained by several teams, including a timespan to measure coupling might be necessary (see chapter 12). Also, some important coupling occurs between commits, then you have to dig into the code. Finally, renaming module resets counters when using Code Maat, it can sounds problematic, but it's also a good signal that some refactoring happened.  

### Catch Architectural Decay

[Manny Lehman](https://en.wikipedia.org/wiki/Manny_Lehman_%28computer_scientist%29) also expressed another law: it states a program that is used _undergoes continual change or becomes progressively less used_.  

We're going to use a new repo for the analysis.  
Clone the repo: `git clone https://github.com/SirCmpwn/Craft.Net.git`  
Extract logs: `git log --pretty=format:'[%h] %an %ad %s' --date=short --numstat --before=2014-08-08 > craft_evo_complete.log`  
Mesure the Sum of Coupling: `maat -l craft_evo_complete.log -c git -a soc`  

First module observed, _MinecraftServer_ seems to have a lot of coupling. We're going to run a trend analysis on this module.  

First activity for this module is in 2012, so we extract logs for first year as a first period: `git log --pretty=format:'[%h] %an %ad %s' --date=short --numstat --before=2013-01-01 > craft_evo_130101.log`  
Then we run a temporal coupling analysis: `maat -l craft_evo_130101.log -c git -a coupling > craft_coupling_130101.csv`  

Then we repeat this process for a period over 2013 to 2014:  

- `git log --pretty=format:'[%h] %an %ad %s' --date=short --numstat --after=2013-01-01 --before=2014-08-08 > craft_evo_140808.log`  
- `maat -l craft_evo_140808.log -c git -a coupling > craft_coupling_140808.csv`  

We can now open both csv into a spreadsheet app and remove everything not coupled to _MinecraftServer_.  

### React to Structural Trends

Now we can use the enclosure diagram (see chapter 4) and compare visually results.  
Dependencies that are close to eachothers isn't an issue, we're looking for temporal coupling across (package/project) boundaries, with modules on distant parts of the system.  
You can run such an analysis as a routine on your project (each iteration for example) and spot early decays.  

-----

## Chapter 9 : Build a Safety Net for Your Architecture

### Know What's in an Architecture

Automated tests act as a safety net for the software, it's supposed to allow developers to modify the software and ensure there's no regression.  
But tests create dependencies with the tested code, so we have to choose those dependencies carefully. The author argues that automated tests should be considered like any other module of the system and designed with the same attention.  

### Analyse the Evolution on a Sytem Level

We've looked at _temporal coupling_ between invdividual modules, now we're focusing on system boundaries between production code and automated tests.  

To specify to Code Maat production and test code, we've to specify a _transformation_.  
Create a file `maat_src_test_boundaries.txt` in the repository root, then type in:  

```text
src/code_maat => Code
test/code_maat => Test
```  

_Note_: this file must use LF endline sequence instead of CRLF.  

Finally, perform the analysis: `maat -l maat_evo.log -c git -a coupling -g maat_src_test_boundaries.txt`  

We can observe a high level of coupling between code and tests, but tests regroup different kinds of tests. We must make a distinction.  

### Differentiate Between the Level of Tests

Now, put in `maat_src_test_boundaries.txt` file:  

```text
src/code_maat => Code
test/code_maat/analysis => Analysis Test
test/code_maat/dataset => Dataset Test
test/code_maat/end_to_end => End to end Test
test/code_maat/parsers => Parsers Test
```  

Then re-run the analysis.  

On this example, Analysis & Parsers tests are unit tests, they change aproximatively 40% of the time with the code. That sounds reasonable. Datastet isn't displayed has its numbers are below the default threshold.  
End to end tests also changes 40% of the time, but they're supposed to be more stable. That's a bad signal.  
In this specific case, the author explains he changed several times the output format, but didn't encapsulate it as an implementation detail. So every time he made a change, he had to modify the end to end tests.  

### Create a Safety Net for Your Automated Tests

You can monitor revisions for each boundaries: `maat -l maat_evo.log -c git -a revisions -g maat_src_test_boundaries.txt`  
By collecting several sample points, we can start to see trends. We can also observe the evolution of code/test change ratio.  
Every time the ratio evolves to a high test changes, run coupling and hotspot analysis to help you understand the problem.  

We can also spot clusters of tests that change together, it's a sign that some test refactoring is needed.  

-----

## Chapter 10 : Use Beauty as a Guiding Principle

### Learn Why Attractiveness Matters

Here, beauty should be interpreted as "the absence of ugliness".  

### Write Beautiful Code

Translated to code, the absence of ugliness means the absence of special cases.  
A beautiful code is a consistent code in terms of code style, conventions, level of expression, etc.  
Every time the code base divert, it breaks the reader's expectations and introduces cognitive cost. The result is a code base harder to understand and riskier to modify.  
This principle also applies at the architecture level, and is even more important than the local coding constructs.  

### Avoid Surprises in Your Architecture

Code Maat is build following the _Pipes and Filters_ pattern, so we should expect a low temporal coupling between filters.  

In a file `maat_pipes_filters_boundaries.txt`:  

```text
src/code_maat/parsers => Parse
src/code_maat/analysis => Analyze
src/code_maat/output => Output
src/code_maat/app => Application
```  

The perform the temporal coupling analysis: `maat -l maat_evo.log -c git -a coupling -g maat_pipes_filters_boundaries.txt`  

Result doesn't highlight any strong coupling between filters, but two of them are coupled to the _Application_ component.  
The reason is: _Application_ contains conditionnal logic to choose the parsers and the analysis to execute. It could be a problem as the software can grow with more options.  

### Analyze Layered Architectures

The transformation file doesn't have to mirror code stucture. You can ignore minor utility modules and try to focus on what can break your target architecture.  

New study case with [NopCommerce](www.nopcommerce.com) open source product. It's build with a MVC pattern.  

### Find Surpring Change Patterns

Clone the new repo: `git clone https://github.com/nopSolutions/nopCommerce.git`  
Then extract logs: `git log --pretty=format:'[%h] %an %ad %s' --date=short --numstat --after=2014-01-01 --before=2014-09-25 > nop_evo_2014.log`  

_Note_: You may encounter the following error:  

```text
warning: inexact rename detection was skipped due to too many files.
warning: you may want to set your diff.renameLimit variable to at least 1954 and retry the command.
```

Use `git config diff.renamelimit 1954` to solve it.  

Use the following transformation file `arch_boundaries.txt`:

```text
src/Presentation/Nop.Web/Administration/Models      => Admin Models
src/Presentation/Nop.Web/Administration/Views       => Admin Views
src/Presentation/Nop.Web/Administration/Controllers => Admin Controllers
src/Libraries/Nop.Services                          => Services
src/Libraries/Nop.Code                              => Core
src/Libraries/Nop.Data                              => Data Access
src/Presentation/Nop.Web/Models                     => Models
src/Presentation/Nop.Web/Views                      => Views
src/Presentation/Nop.Web/Controllers                => Controllers
```

And run coupling analysis: `maat -l nop_evo_2014.log -c git -a coupling -g arch_boundaries.txt`  

_Note_: Few commit messages contained several lines, I had to manually fix them on `nop_evo_2014.log` in order to execute the analysis without failure.  

The result highlight several coupling, a lot of them from admin modules.  

Run a Hotspots analysis: `maat -l nop_evo_2014.log -c git -a revisions -g arch_boundaries.txt`  

It shows us that the _Service_ layer is the most volatile, and temporal coupling tells us that 35% of those revisions also modify _Admin Controllers_. But with this data, we can't know if _Service_ leads to _Admin Controllers_ modifications or if it is the other way around.  

### Expand Your Analyses

You can now use _Temporal Coupling_ as a early warning system. Define the rules you want to protect, then run the analysis on a regular basis.  
If the trend evoles in an unexpected way, run a hotspots analysis to investigate.  

-----

## Chapter 11 : Norms, Groups, and False Serial Killers

Social interactions and team organization are as influent as software architecture for producing bugs and decay.  

### Learn Why the Right People Don't Speak Up

_Process loss_ is the theory that groups can't operate at 100% efficiency. There's losses by coordination and motivations.  
In software development we must accept some losses as systems are too big to be developed by a single person.  

Social biases make you influenced by others, by their attitude, how confident they look like, etc.  

### Understand Pluralistic Ignorance

_Pluralistic Ignorance_ is when everyone reject privately a rule but thinks that others support it. It can lead to teams following rules that no one wants to follow.  
An individual can also influence decisions just by repeating his opinion. Just by hearing it more often, we tend to find it more valuable.  

Social biases are hard to avoid, the best ways to do so are:  

- ask questions
- talk to people
- use data to support decisions

If you're in a leadership position, there are additionnal solutions:  

- use an outside expert to review decisions
- let subgroups work independently on the same problem
- avoid advocating a specific solution early in the discussions
- discuss worst-case scenarios and build team risk awareness
- plan a second meeting to reconsider decisions made on the first one

All those strategies are useful to avoid _groupthink_. _Groupthink_ is when a group has suppressed all internal forms of dissent, it leads groups suffering from a false sense of consensus, ignoring alternatives and risks.  

### Witness Groupthing in Action

Some workshops like brainstorming are supposed to promote high group creativity. The reality is: they generate a lot of social biases and tend to produce less creative groups than expected.  

### Discover Your Team's Modus Operandi

Every team as its own way to work. Even if it can't be observed clearly, using commit logs can provide some useful information.  

To extract only commit logs: `git log --pretty='%s'`  
You can then use tools to visualize them with clouds representations, [here](https://monkeylearn.com/blog/word-visualization/) some examples.  

The more a word is present, the more the team is doing it. You can potentially double check your discoveries with a _temporal coupling_ analysis.  

**Friendly reminder**: these kind of analysis is tools made to understand and support decisions, it doesn't replace real discussions and team interactions.  

-----

## Chapter 12 : Discover Organizational Metrics in Your Codebase

### Let's Work in the Communication Business

_"Adding manpower to a later software project makes it later"_ - [The Mythical Man-Month: Essays on Software Engineering](https://www.goodreads.com/book/show/13629.The_Mythical_Man_Month)  

Software development is an intellectual work that is hard to parallelize, adding more people to the team won't accelerate the developments. Even worst, the more people you add to a team, the more you increase coordination cost, and those tend to increase exponentially.  
Additionally, a large group suffers from _process loss_ and less responsibility for the overall goal.  

### Find the Social Problems of Scale

Adding developers isn't necessarily a bad thing as long as architecture allows them to work on separate pieces of code. Troubles start when some _hotspots_ accumulate responsibilities and force developers to edit the same code for different reasons.  

We can run an author analysis: `maat -l hib_evo.log -c git -a authors`  
The result is the modules with the number of authors and revisions.  

A study on Windows Vista's code shows that organizational structure add a huge impact on the overall code quality.  
The number of authors was one social metric, and it outperformed technical metrics such as code complexity or coverage to predict defects.  

### Mesure Temporal Coupling over Organizational Boundaries

If a module is highlighted by both _hotspot_ and _authors_ analysis, then we're facing a piece of code that is probably tricky and affects most developers: a good candidate for some rework!  

_Conway's law_: _"Any organization that designs a system will inevitably produce a design whose structure is a cop of organization's communication structure."_  
This famous law can be interpreted in two distinct ways:

- "If you have four groups working on a compiler, you'll get a 4-pass compiler"
- The reverse one: how to structure organization to match a specific architecture?

To analyze _temporal coupling_ over organizational boundaries, we need to consider commits on the same day as part of a logical change set.  
To do so: `maat -l hib_evo.log -c git -a coupling --temporal-period 1`  
With this analysis, we measure the probability of coupled modules to change within the same day.  

Next step is to identify main developers of coupled modules. We can then compare it to the formal organization and reason about communication.  

### Evaluate Communication Costs

To identify the main developer of a module, we could look at the number of lines add, but it could promote some kind of "copy-paste Cowboys".  
Code Maat propose the `refactoring-main-dev` analysis to identify the developer who removed the more lines of codes as it is probably someone who takes an active part in module maintenance and refactoring.  

To identify the main developers: `maat -l hib_evo.log -c git -a main-dev > main_devs.csv`  
We can now identify the main developer and his degree of ownership of both the hotspot and the modules it's coupled to.  
If those modules are all "owned" by the same person with a strong ownership, then it's ok from the organizational point of view. If it's "owned" by different people, we should consider several things: Are they on the same team? At the office or remotely? On the same time zone?  

To calculate individual contributions: `maat -l hib_evo.log -c git -a entity-ownership`  

_Author note_: Git "Two commiters name" can influence results, always look at logs and take times to clean it up before running an analysis.  

### Take It Step by Step

To recap:  

1. Identify parallel work
2. Compare against hotspots
3. Identity temporal coupling
4. Find the main developers
5. Check organizational distance
6. Optimize for communication

The latest step can be achieved by either changing the organizational structure or the software architecture.  

-----

## Chapter 13 : Build a Knowledge Map of Your System

### Know Your Knowledge Distribution

In previous chapter we've identified authors of a module and measured ownership metrics to identify who may old the most knowledge of this module.  
But this metric doesn't tell us if there is one main contributor or several ones who maintain overall consistency of the module.  
To do so, we have to measure individual contributions: `maat -l hib_evo.log -c git -a entity-effort`  

To improve visualization of the result, we can use some [fractal figures](https://github.com/adamtornhill/FractalFigures).  

You can then observe three different patterns:

- Single developer: Easiest pattern, the quality depends only on the expertise of the developer.
- Multiple, balanced developers: Few developers with a clear ownership of one of them. The more ownership, the fewer defects in the code and better quality.
- Collective chaos: A lot of minor contributors, this is a strong predictor of defects.

Here we've got an example of modules, we can run the same analysis at the architectural level by specifying boundaries (the `-g` parameter).  

### Grow Your Mental Maps

We can build a map with modules and associated main contributors.  
In the scala sample directory:

- Run `python -m http.server 8888`  
- Then open `http://localhost:8888/scala_knowledge.html```

We can now visualize easily whom to ask if we want to work on a specific module.  

### Investigate Knowledge in the Scala Repository

To build such a map:  

- Clone the Scala repository: `git clone https://github.com/scala/scala.git`
- Check the branch: `git status`, in my case I am on branch _2.13.x_
- Go back in time for predictable results: ``git checkout `git rev-list -n 1 --before="2013-12-31" origin/2.13.x` ``
- Extract logs: `git log --pretty=format:'[%h] %an %ad %s' --date=short --numstat --before=2013-11-01 --after=2011-12-31 > scala_evo.log`
- Extract main devs: `maat -l scala_evo.log -c git -a main-dev > scala_main_devs.csv`
- Count the number of lines: `cloc ./ --by-file --csv --quiet --report-file=scala_lines.csv`

At this point we have everything we need to build on the map. Note we've limited the period of time as knowledge decrease over time if we don't edit a module.  

We can use tools to generate good color schemes like [ColorBrewer](http://colorbrewer2.org).  
Colors should be specified by [HTML5 color names](https://www.w3schools.com/tags/ref_colornames.asp).  
We can then build an author/color mapping like the one in the sample directory.  

Finally, we can build our knowledge map: `python scripts/csv_main_dev_as_knowledge_json.py --structure scala_lines.csv --owners scala_main_devs.csv --authors scala_author_colors.csv > scala_knowledge_131231.json`  

We can now use d3.js to visualize the result (replace in the _scala_knowledge.html_ reference to the file with the one you've generated).  

### Visualize Knowledge Loss

Documentation, reviews, etc, can't replace the intricate knowledge of working on a piece of code. That's why the number of ex-developers who worked on a component is a good predictor of the number of post-release defects.  

In the scala repository, we know that Paul Phillips was a main contributor who chose to leave.  
Let's identify the abandoned code:  

- Create a new `scala_ex_programmers.csv` with content

```csv
author,color
Paul Phillips,green
```

- Generate the json visualization:  `python scripts/csv_main_dev_as_knowledge_json.py --structure scala_lines.csv --owners scala_main_devs.csv --authors scala_ex_programmers.csv > scala_knowledge_loss.json`  

Now we can visualize every "abandoned" piece of code as the result of Paul's leaving as they appear in green.  
