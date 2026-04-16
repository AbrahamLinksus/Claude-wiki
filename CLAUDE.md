
# LLM WIKI: 
- A personal knowledge base maintained by Claude Code
- Based on Andrej Kaparthy's LLM wiki pattern.

# Purpose : 
- This wiki is a structured, interlinked knowledge base for research, coding knowledge, academics and related stuff
- Claude maintains the wiki. The human curates sources, asks questions and guides the analysis.

# Folder Structure : 
```
- raw/ -- source documents (immutable -- never modify these)
    
- templates/ -- templates on how do do certain research when called for 
     
- wiki/ -- markdown pages maintained by Claude
    
- wiki/index.md -- table of contents for the entire wiki
    
- wiki/log.md -- append-only record of all operations

```

# Templates : 
will update when required, skip for now 

# Ingestion Workflow ; 
- when the user adds a new file into the raw/ folder and asks you to ingest it : 
- 
	1) Read the full source document and get the context of the file

	2) Discuss key takeaways with the user before writing anything 

	3) Create a summary page in the 'wiki/' named after a crisp title of the source

	4) Create or update concept pages for each major idea or entity

	5) Add wiki-links ( [[page-name]] ) to connect related pages
	
	6) Update `wiki/index.md` with a link to connect main summary page to the index file, to have a proper structure
	
	7) Append an entry to `wiki/log.md` with the date, source name, and what changed
	
	8) A single source may touch 10-15 wiki pages. That is normal.
	
	9) I will give a tag for each raw file that has to be processed into a new wiki web, i want to you add it to every file that is generated for this current wiki page generation. It will be of the format ' #text '

# Summary Page format : 
The summary page must be a little different from the other wiki pages 
 Its aim is to give an overview of the whole concept, with links to go deeper into the main topics, and to give like a roadmap to completion of this topic so generate accordingly
 ```markdown
    
- # Page Title
    

- **Summary**: One to two sentences describing this page.
    

- **Sources**: List of raw source files this page draws from.
    

- **Last updated**: Date of most recent update.
  ---
  
  All the main heading topics are listed here as wiki-links properly labeled maybe with a 1 line definition. 
  

 ```

# Wiki Page format : 
The wiki page is the main entity that hold processed information from the raw folder
 Its aim is to give a detailed approach to processing the following information
 ```markdown
    
- # Page Title
    

- **Summary**: One to two sentences describing this page.
	
  
- ** Pre-Req**: topics to have prior knowledge to understand the current properly
  if there exists any, simple and basic ones can be ignored
    

- **Sources**: List of raw source files this page draws from.
    

- **Last updated**: Date of most recent update.
  ---
  
  Main content goes here. Use clear headings and short paragraphs, use wiki-links to other main topics if they are referenced here.
  Paste diagrams too if availabled properly labeled for ease of learning
  
 ```

- ## Citation rules
    

- - Every factual claim should reference its source file
    
- - Use the format (source: filename.pdf) after the claim
    
- - If two sources disagree, note the contradiction explicitly
    
- - If a claim has no source, mark it as needing verification

## Question answering
    

- When the user asks a question:
    

- 1. Read `wiki/index.md` first to find relevant pages
    
- 2. Read those pages and synthesize an answer
    
- 3. Cite specific wiki pages in your response
    
- 4. If the answer is not in the wiki, say so clearly
    
- 5. If the answer is valuable, offer to save it as a new wiki page
    

- Good answers should be filed back into the wiki so they compound over time.


- ## Lint
    

- When the user asks you to lint or audit the wiki:
    

- - Check for contradictions between pages
    
- - Find orphan pages (no inbound links from other pages)
    
- - Identify concepts mentioned in pages that lack their own page
    
- - Flag claims that may be outdated based on newer sources
    
- - Check that all pages follow the page format above
    
- - Report findings as a numbered list with suggested fixes

- ## Rules
    

- - Never modify anything in the `raw/` folder
    
- - Always update `wiki/index.md` and `wiki/log.md` after changes
    
- - Keep page names lowercase with hyphens (e.g. `machine-learning.md`)
    
- - Write in clear, plain language
    
- - When uncertain about how to categorize something, ask the user