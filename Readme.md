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
Note: this is just one tool among others, the author also highligts:

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

Note: you can have a high average-revs and a low degree, it means there's a lot of revisions, but only few are shared by the two modules. At the oposite, high degree and low average-revs means a strong coupling but stable modules.  

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
