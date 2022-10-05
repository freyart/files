---
obsidianUIMode: preview
---
An extension that gives the option to select random lines from any note in your vault, letting a simple note act as a random table that you can roll on.
__

__
```js

async function getAllLinesFromNote($1) {
	$1 = $1.replaceAll(/^\"|\"$/g, "");
	
	/* Validate the note path, must be a .md file and get its content. */
	const fileSearch = app.vault.getMarkdownFiles().filter(x => x.path.toLowerCase() == $1.toLowerCase() + ".md")
	if (fileSearch.length != 1)
		return "Cant find specified file. The file __" + $1 + ".md__ was not found.\n\n";
	
	const sampleTFile = this.app.vault.getAbstractFileByPath(fileSearch[0].path);
	const file = await this.app.vault.cachedRead(sampleTFile); 
	let allLines = file.split(/\r\n|\n/); // splits the file into individual lines
	
	if(allLines.length == 0)
		return "The file __" + $1 + ".md__ is empty.\n\n";
	
	/* Strip YAML tags block if detected */
	if(allLines[0] == "---" && allLines.filter(x=> x == "---").length > 1){
		allLines = allLines.slice(allLines.indexOf("---", 1)+1);
	}
	
	/* Filter out empty lines */
	allLines = allLines.filter(x => x.trim() != "");
	if(allLines.length == 0)
		return "The file __" + $1 + ".md__ is empty.\n\n";
	
	return allLines;
}
```
__
Helper scripts

__

__
```js
	async function getRandomResultsFromNote($1, $2) {		
		/* Fetch all lines of content from Markdown .md note */
		const lines = await getAllLinesFromNote($1);
		if (!Array.isArray(lines))
			return lines; //return error message.
		
		/* Generate result(s) of the roll a number of times specified by $2 */
		let result = new Array();
		for (let i = 0; i < Math.max(1, $2); i++) {
			let randomNum = Math.floor(Math.random() * lines.length);
			result.push({"roll":randomNum+1, "result":lines[randomNum]});
		}
		
		return {result: result, total: lines.length};
	}
```
__
Helper Scripts

__
```
^linesFromNote ("[^ \t\\:*?"<>|][^\t\\:*?"<>|]*"|[^ \t\\:*?"<>|]+) ?([1-9][0-9]*|)$
```
__
```js
	/* $1 must be equal to the relative path of a markdown note */
	/* $2 (optional) the number of results to generate */
	/* Return error message as string or Array of results (at least 1) */
		
	$2 = Number($2 || 1);
	const randomResults = await getRandomResultsFromNote($1, $2);
	
	if (typeof randomResults === 'string' || randomResults instanceof String)
		return randomResults;

	let result = "🎲 ";
	let firstResult = true;
	randomResults.result.forEach((element, index) => { 
		const separator = firstResult ? '': ', ';
		firstResult = false;
		result += `${separator}${element.result}`; 
	});
	
	return result + "\n";
```
__
linesFromNote {note-path: } {number >0, default: 1} - Picks a number of random lines from the specified note.
Return the random results on a single line.
*Note*: Put {note-path} between apostrophes if there's a space in the name. "notes/like this for example".

__
```
^lst linesFromNote ("[^ \t\\:*?"<>|][^\t\\:*?"<>|]*"|[^ \t\\:*?"<>|]+) ?([1-9][0-9]*|)$
```
__
```js
	/* $1 must be equal to the relative path of a markdown note */
	/* $2 (optional) the number of results to generate */
		/* Return error message as string or Array of results (at least 1) */
	$2 = Number($2 || 1);
	const randomResults = await getRandomResultsFromNote($1, $2);
	
	if (typeof randomResults === 'string' || randomResults instanceof String)
		return randomResults;
	
	let result = `🎲: __${$2}__ Rolls from ${randomResults.total} possible results`;
	randomResults.result.forEach((element, index) => { 
		result += `\n- ${element.result} (${element.roll})`; 
	});
	
	return result + "\n";
```
__
lst linesFromNote {note-path: } {number >0, default: 1} - Picks a number of random lines from the specified note.
Return the random results as a List.
*Note*: Put {note-path} between apostrophes if there's a space in the name. "notes/like this for example".

__
```
^tbl linesFromNote ("[^ \t\\:*?"<>|][^\t\\:*?"<>|]*"|[^ \t\\:*?"<>|]+) ?([1-9][0-9]*|)$
```
__
```js
	/* $1 must be equal to the relative path of a markdown note */
	/* $2 (optional) the number of results to generate */
	/* Return error message as string or Array of results (at least 1) */
	$2 = Number($2 || 1);
	const randomResults = await getRandomResultsFromNote($1, $2);
	
	if (typeof randomResults === 'string' || randomResults instanceof String)
		return randomResults;
	
	let result = `|🎲 D${randomResults.total}| Result(s) |`;
	result += `\n|:---:|---|`
	randomResults.result.forEach((element, index) => { 
		result += `\n|${element.roll}|${element.result}|`; 
	});
	
	return result + "\n";
```
__
tbl linesFromNote {note-path: } {number >0, default: 1} - Picks a number of random lines from the specified note.
Return the random results as a Table.
*Note*: Put {note-path} between apostrophes if there's a space in the name. "notes/like this for example".