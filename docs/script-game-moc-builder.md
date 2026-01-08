---
ctime: "2026-01-08T19:59:40+08:00"
mtime: "2026-01-08T19:59:40+08:00"
---

# script-game-moc-builder

```js
function getSeeAlsoPart(fm) {
	if (!fm['url-wikipedia']) {
		return "";
	} else {
		return `\n> see: [${fm['title-wikipedia']}](${fm['url-wikipedia']})\n`;
	}
}
function getImageElem(wikilink) {
	const linktext = convertWikiLinkToLinkText(wikilink);
	const linkfilepath = app.metadataCache.getFirstLinkpathDest(linktext).path;
	return `<img src="${linkfilepath}" width=200>`;
}
function getImagePropTablePart(fm) {
	const rows = ['icon', 'cover', 'image'].map(key=>{
		if (!fm[key]) {
			return null;
		}
		const linkPropKey = key+'-url';
		const linkDescription = fm[linkPropKey]?`<br>${fm[linkPropKey]}`:"";
		return `| ${key} | ${getImageElem(fm[key])}${linkDescription} |`
	}).filter(row=>row).join("\n");
	return `\n| | |\n| --- | --- |\n${rows}`;
}
function getGameItemSection(file) {
	const sectionHeaderText = /^game-item-(.*)/.exec(file.basename)[1];

	const fileCache = app.metadataCache.getFileCache(file);
	const fm = fileCache.frontmatter;
	return `### ${sectionHeaderText}\n${getSeeAlsoPart(fm)}\n${fm['description-wikipedia']||fm['description']}\n${getImagePropTablePart(fm)}`;
}

function convertWikiLinkToLinkText(wikilink) {
	return /^\[\[([^\[\]\|]+)(\\?\|)?/.exec(wikilink)[1]	
}

function getGameCategorySection(file) {
	const fileCache = app.metadataCache.getFileCache(file);
	const subSections = fileCache.frontmatter.subpages.map(convertWikiLinkToLinkText).sort().map(linktext=>app.metadataCache.getFirstLinkpathDest(linktext)).map(l=>getGameItemSection(l)).join("\n\n");
	
	return `## ${file.basename.replace(/^game-category-/,"")}\n\n${subSections}\n`
}
console.log(app.vault.getMarkdownFiles()
	.filter(f=>f.path.startsWith("game-categories/"))
	.sort(f=>f.basename)
	.map(f=>getGameCategorySection(f))
	.join("\n"));

```
