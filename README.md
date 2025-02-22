# Website to Markdown

## Install the bookmarklet

Add this code as the address in your bookmark
```js
javascript:(function(){function getUniqueSelector(e){if(e.id)return"%23"+CSS.escape(e.id);var t=[];while(e&&e.nodeType===Node.ELEMENT_NODE&&e!==document.body){var n=e.nodeName.toLowerCase();if(e.className){var r=e.className.trim().split(/\s+/);r[0]&&(n+="."+CSS.escape(r[0]));}var a=e,s=1;while(a=a.previousElementSibling){if(a.nodeName.toLowerCase()===e.nodeName.toLowerCase())s++;}n+=":nth-of-type("+s+")",t.unshift(n),e=e.parentElement}return t.join(" > ")}function slugifyTitle(e){return e.normalize("NFD").replace(/[\u0300-\u036f]/g,"").replace(/[^a-zA-Z0-9 ]/g,"").trim().toLowerCase().replace(/\s+/g,"-")}window.__extractionSelections=window.__extractionSelections||[];var selections=window.__extractionSelections;function moveSelection(i,s){if("up"===s&&i>0){var e=selections[i-1];selections[i-1]=selections[i],selections[i]=e}else if("down"===s&&i<selections.length-1){var e=selections[i];selections[i]=selections[i+1],selections[i+1]=e}window.__extractionSelections=selections,renderSelections()}function addSelectionItem(label,selector,index){var li=document.createElement("li");li.style.marginBottom="5px",li.dataset.selector=selector;var inp=document.createElement("input");inp.type="text",inp.value=label,inp.style.width="60%",inp.onchange=function(){var e=selections.filter(function(e){return e.selector===selector})[0];e&&(e.label=inp.value)};inp.onfocus=function(){selections.forEach(function(sel){var els=document.querySelectorAll(sel.selector);els.forEach(function(el){el.style.outlineColor=(sel.selector===selector)?"blue":"rgba(0,0,255,0.5)";})});};inp.onblur=function(){selections.forEach(function(sel){var els=document.querySelectorAll(sel.selector);els.forEach(function(el){el.style.outlineColor="blue";});});};var upBtn=document.createElement("button");upBtn.textContent="Up",upBtn.style.marginLeft="5px",upBtn.onclick=function(e){e.stopPropagation(),moveSelection(index,"up")};var downBtn=document.createElement("button");downBtn.textContent="Down",downBtn.style.marginLeft="5px",downBtn.onclick=function(e){e.stopPropagation(),moveSelection(index,"down")};var remBtn=document.createElement("button");remBtn.textContent="Remove",remBtn.style.marginLeft="5px",remBtn.onclick=function(e){e.stopPropagation();var elems=document.querySelectorAll(selector);elems.forEach(function(el){el.classList.remove("extraction-highlight")});selections=selections.filter(function(e){return e.selector!==selector}),window.__extractionSelections=selections,renderSelections()};li.appendChild(inp),li.appendChild(upBtn),li.appendChild(downBtn),li.appendChild(remBtn),document.getElementById("selection-list").appendChild(li)}function renderSelections(){var e=document.getElementById("selection-list");if(!e)return;e.innerHTML="",selections.forEach(function(t,i){addSelectionItem(t.label,t.selector,i)})}var labelCounter=1,configKey="extractionConfig_"+window.location.origin;function applySavedConfig(){var e=localStorage.getItem(configKey);if(e)try{var t=JSON.parse(e);t.forEach(function(e){if(document.querySelector(e.selector)){if(!selections.find(function(t){return t.selector===e.selector}))selections.push(e);var n=document.querySelectorAll(e.selector);n.forEach(function(e){e.classList.add("extraction-highlight")})}else console.debug("Saved selector not found in DOM:",e.selector)}),renderSelections()}catch(e){console.error("Error parsing saved config:",e)}}function clearAllHighlights(){selections.forEach(function(e){var t=document.querySelectorAll(e.selector);t.forEach(function(e){e.classList.remove("extraction-highlight"),e.style.outlineColor=""})});}var existingSidebar=document.getElementById("extraction-sidebar");if(existingSidebar){existingSidebar.style.display="block",renderSelections(),setTimeout(applySavedConfig,1500);return}var sidebar=document.createElement("div");sidebar.id="extraction-sidebar",sidebar.innerHTML=%27<div id="extraction-header" style="padding:10px;background:#f0f0f0;border-bottom:1px solid #ccc;color:#000;"><strong>Extraction UI</strong><button id="extraction-close" style="float:right;">X</button><div style="clear:both;"></div></div><div id="extraction-body" style="padding:10px;color:#000;"><p>Click elements to toggle selection/deselection.</p><ul id="selection-list" style="list-style:none;padding:0;"></ul><button id="create-markdown" style="margin-top:10px;">Create Markdown</button><button id="save-config" style="margin-top:10px;">Save Config</button><button id="clear-config" style="margin-top:10px;">Clear Saved Config</button><div id="markdown-preview-section" style="display:none;margin-top:10px;"><textarea id="markdown-preview" style="width:100%;height:200px;"></textarea><button id="download-markdown" style="margin-top:10px;">Download Markdown</button></div></div>';sidebar.style.position="fixed",sidebar.style.top="0",sidebar.style.right="0",sidebar.style.width="300px",sidebar.style.height="100%",sidebar.style.background="#fff",sidebar.style.borderLeft="1px solid #ccc",sidebar.style.zIndex="9999999",sidebar.style.overflowY="auto",sidebar.style.color="#000",document.body.appendChild(sidebar);var styleElem=document.createElement("style");styleElem.innerHTML=".extraction-highlight { outline: 1px solid blue !important; }",document.head.appendChild(styleElem),renderSelections(),setTimeout(applySavedConfig,1500);var clickHandler=function(e){if(e.composedPath().some(function(e){return e.id==="extraction-sidebar"}))return;if(e.composedPath().some(function(e){return e.getAttribute&&e.getAttribute("data-download-trigger")==="true"}))return;e.preventDefault(),e.stopPropagation();var tgt=e.target;tgt===document.documentElement||tgt===document.body||(function(){var sel=getUniqueSelector(tgt),lbl=tgt.tagName.toLowerCase()+"-"+(labelCounter++),found=selections.find(function(e){return e.selector===sel});if(found){selections=selections.filter(function(e){return e.selector!==sel});tgt.classList.remove("extraction-highlight");window.__extractionSelections=selections,renderSelections();return}tgt.classList.add("extraction-highlight"),selections.push({label:lbl,selector:sel});window.__extractionSelections=selections,renderSelections()}())};document.addEventListener("click",clickHandler,true);var keyupHandler=function(e){if(e.key==="Escape"||e.keyCode===27){var sb=document.getElementById("extraction-sidebar");if(sb){clearAllHighlights(),sb.style.display="none"};document.removeEventListener("click",clickHandler,true);document.removeEventListener("keyup",keyupHandler,true)}};document.addEventListener("keyup",keyupHandler,true);document.getElementById("extraction-close").onclick=function(e){e.stopPropagation(),clearAllHighlights(),sidebar.style.display="none",document.removeEventListener("click",clickHandler,true),document.removeEventListener("keyup",keyupHandler,true)};document.getElementById("create-markdown").onclick=function(e){e.stopPropagation();var title=document.title,url=window.location.href,d=new Date(),dateStr=(d.getMonth()+1).toString().padStart(2,"0")+"-"+d.getDate().toString().padStart(2,"0")+"-"+d.getFullYear(),md="---\ntitle: "+title+"\nurl: "+url+"\ndate: "+dateStr+"\n---\n\n";selections.forEach(function(e){var t=document.querySelector(e.selector),n=t?t.innerText.trim():"";md+="## "+e.label+"\n\n"+n+"\n\n"});var ps=document.getElementById("markdown-preview-section"),ta=document.getElementById("markdown-preview");ta.value=md,ps.style.display="block"};document.getElementById("download-markdown").onclick=function(e){e.stopPropagation();var content=document.getElementById("markdown-preview").value,d=new Date(),dateStr=(d.getMonth()+1).toString().padStart(2,"0")+"-"+d.getDate().toString().padStart(2,"0")+"-"+d.getFullYear(),slug=slugifyTitle(document.title||"document"),fileName=dateStr+"_"+slug+".md",blob=new Blob([content],{type:"text/markdown;charset=utf-8"}),a=document.createElement("a");a.href=URL.createObjectURL(blob),a.download=fileName,a.style.display="none",a.setAttribute("data-download-trigger","true"),document.body.appendChild(a),a.click(),document.body.removeChild(a)};document.getElementById("save-config").onclick=function(e){e.stopPropagation();if(!selections.length){alert("No selections to save!");return}localStorage.setItem(configKey,JSON.stringify(selections)),alert("Configuration saved!")};document.getElementById("clear-config").onclick=function(e){e.stopPropagation();localStorage.removeItem(configKey);clearAllHighlights(),selections=[],window.__extractionSelections=selections,renderSelections(),alert("Saved configuration cleared.")};window.addEventListener("beforeunload",function(){document.removeEventListener("click",clickHandler,true);document.removeEventListener("keyup",keyupHandler,true)})})();
```




## Overview
The bookmarklet is a tool that lets you interactively select and label parts of a webpage and then export them as a Markdown file. It injects a sidebar UI into the page, where you can toggle selections, rearrange their order, and generate a file that includes metadata (title, URL, and date) along with your selected text.

## Goals
- **Interactive Selection:** Click on elements to add (or remove) them from your selection. Selected elements get a blue border.
- **Labeling & Reordering:** Edit labels for each selection and use "Up" and "Down" buttons to change their order.
- **Markdown Generation:** Create a Markdown file that starts with a YAML header (including page title, URL, and date) followed by sections for each selection.
- **Configuration Persistence:** Save your extraction configuration in the browser’s localStorage so you can reuse it on other pages of the same domain.
- **Clean Deactivation:** Remove all event listeners and visual highlights when you close the sidebar or press Escape, restoring normal webpage interaction.

## Implementation
- **UI Injection:**  
  The bookmarklet injects a sidebar (using vanilla HTML and inline CSS) into the current page. The sidebar contains a list for your selections and buttons for creating Markdown, saving, and clearing configurations.

- **Event Handling:**  
  A document-level click listener computes a unique CSS selector for each clicked element (using its tag name, first class, and nth-of-type index). If an element is already selected, clicking it again will remove the selection and its highlight. A keyup listener detects the Escape key to deactivate the tool.

- **Configuration Management:**  
  Selections are stored in an array and persisted using localStorage under a key based on the page’s origin. This makes it possible to load and reuse your saved selections on any page within the same domain.

- **Markdown Generation:**  
  When you create the Markdown, the script gathers the selected elements’ text and constructs a Markdown file with a YAML header. The file is then downloaded with a name generated from the current date and a slugified version of the page title.

- **Clean-Up:**  
  When closing the sidebar (via the X button or Escape key), the tool removes all highlights from the page and unregisters its event listeners, ensuring the page returns to its normal behavior.

## Usage
1. **Installation:**  
   Create a new bookmark in your browser and set its URL to the bookmarklet code.

2. **Activation:**  
   Click the bookmarklet to open the sidebar.

3. **Selection:**  
   Click on elements to toggle their selection. Edit labels or use the Up/Down buttons in the sidebar to reorder them.

4. **Generate & Download:**  
   Click "Create Markdown" to preview your content, then "Download Markdown" to save the file.

5. **Save/Reset Configuration:**  
   Use "Save Config" to store your selections or "Clear Saved Config" to remove them completely.

6. **Deactivation:**  
   Close the sidebar by clicking the X button or pressing Escape. This removes all highlights and event listeners, restoring normal page interaction.

## Known Bugs
- CSS needs work


## License
This project is open-sourced under the MIT License.
