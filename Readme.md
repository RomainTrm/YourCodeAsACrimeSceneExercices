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
