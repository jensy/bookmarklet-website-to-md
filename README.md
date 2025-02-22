# Website to Markdown

https://github.com/user-attachments/assets/190a596e-f082-4457-ab68-53f3cbae8786

## Install the bookmarklet with Bookmarkleter

Copy this code, [then paste it into Bookmarkleter](https://chriszarate.github.io/bookmarkleter/) and drag the link to your bookmarks bar.

```js
    (function(){
      // Returns a unique CSS selector for an element based on its tag, first class (if any) and its nth-of-type index.
      function getUniqueSelector(e) {
        if(e.id) return "#" + CSS.escape(e.id);
        var selectors = [];
        while(e && e.nodeType === Node.ELEMENT_NODE && e !== document.body) {
          var selector = e.nodeName.toLowerCase();
          if(e.className){
            var classes = e.className.trim().split(/\s+/);
            if(classes[0]){
              selector += "." + CSS.escape(classes[0]);
            }
          }
          var sibling = e, nth = 1;
          while(sibling = sibling.previousElementSibling) {
            if(sibling.nodeName.toLowerCase() === e.nodeName.toLowerCase()){
              nth++;
            }
          }
          selector += ":nth-of-type(" + nth + ")";
          selectors.unshift(selector);
          e = e.parentElement;
        }
        return selectors.join(" > ");
      }
      // Slugify a title (remove diacritics, spaces → dashes)
      function slugifyTitle(e) {
        return e.normalize("NFD")
          .replace(/[\u0300-\u036f]/g, "")
          .replace(/[^a-zA-Z0-9 ]/g, "")
          .trim().toLowerCase().replace(/\s+/g, "-");
      }
      // Global storage for selections
      window.__extractionSelections = window.__extractionSelections || [];
      var selections = window.__extractionSelections;
      // Move a selection in the list up or down
      function moveSelection(index, direction) {
        if(direction === "up" && index > 0) {
          var temp = selections[index-1];
          selections[index-1] = selections[index];
          selections[index] = temp;
        } else if(direction === "down" && index < selections.length - 1) {
          var temp = selections[index];
          selections[index] = selections[index+1];
          selections[index+1] = temp;
        }
        window.__extractionSelections = selections;
        renderSelections();
      }
      // Add one selection to the sidebar list
      function addSelectionItem(label, selector, index) {
        var li = document.createElement("li");
        li.style.marginBottom = "5px";
        li.dataset.selector = selector;
        var inp = document.createElement("input");
        inp.type = "text";
        inp.value = label;
        inp.style.width = "60%";
        inp.onchange = function(){
          var sel = selections.filter(function(e){ return e.selector === selector; })[0];
          if(sel) sel.label = inp.value;
        };
        // When input gets focus, adjust outline opacity of all selected elements
        inp.onfocus = function(){
          selections.forEach(function(sel){
            var els = document.querySelectorAll(sel.selector);
            els.forEach(function(el){
              el.style.outlineColor = (sel.selector === selector) ? "blue" : "rgba(0,0,255,0.5)";
            });
          });
        };
        inp.onblur = function(){
          selections.forEach(function(sel){
            var els = document.querySelectorAll(sel.selector);
            els.forEach(function(el){ el.style.outlineColor = "blue"; });
          });
        };
        var upBtn = document.createElement("button");
        upBtn.textContent = "Up";
        upBtn.style.marginLeft = "5px";
        upBtn.onclick = function(e){
          e.stopPropagation();
          moveSelection(index, "up");
        };
        var downBtn = document.createElement("button");
        downBtn.textContent = "Down";
        downBtn.style.marginLeft = "5px";
        downBtn.onclick = function(e){
          e.stopPropagation();
          moveSelection(index, "down");
        };
        var remBtn = document.createElement("button");
        remBtn.textContent = "Remove";
        remBtn.style.marginLeft = "5px";
        remBtn.onclick = function(e){
          e.stopPropagation();
          var elems = document.querySelectorAll(selector);
          elems.forEach(function(el){ el.classList.remove("extraction-highlight"); });
          selections = selections.filter(function(e){ return e.selector !== selector; });
          window.__extractionSelections = selections;
          renderSelections();
        };
        li.appendChild(inp);
        li.appendChild(upBtn);
        li.appendChild(downBtn);
        li.appendChild(remBtn);
        document.getElementById("selection-list").appendChild(li);
      }
      // Render the full list of selections
      function renderSelections(){
        var list = document.getElementById("selection-list");
        if(!list) return;
        list.innerHTML = "";
        selections.forEach(function(sel, index){
          addSelectionItem(sel.label, sel.selector, index);
        });
      }
      var labelCounter = 1;
      var configKey = "extractionConfig_" + window.location.origin;
      function applySavedConfig(){
        var cfg = localStorage.getItem(configKey);
        if(cfg){
          try{
            var arr = JSON.parse(cfg);
            arr.forEach(function(sel){
              if(document.querySelector(sel.selector)){
                if(!selections.find(function(e){ return e.selector === sel.selector; })){
                  selections.push(sel);
                }
                var els = document.querySelectorAll(sel.selector);
                els.forEach(function(el){ el.classList.add("extraction-highlight"); });
              } else {
                console.debug("Saved selector not found in DOM:", sel.selector);
              }
            });
            renderSelections();
          } catch(e){
            console.error("Error parsing saved config:", e);
          }
        }
      }
      function clearAllHighlights(){
        selections.forEach(function(sel){
          var els = document.querySelectorAll(sel.selector);
          els.forEach(function(el){
            el.classList.remove("extraction-highlight");
            el.style.outlineColor = "";
          });
        });
      }
      // If the sidebar already exists, show it and apply saved config.
      if(document.getElementById("extraction-sidebar")){
        document.getElementById("extraction-sidebar").style.display = "block";
        renderSelections();
        setTimeout(applySavedConfig, 1500);
        return;
      }
      // Create sidebar HTML.
      var sidebar = document.createElement("div");
      sidebar.id = "extraction-sidebar";
      sidebar.innerHTML = 
        '<div id="extraction-header" style="padding:10px;background:#f0f0f0;border-bottom:1px solid #ccc;color:#000;">' +
          '<strong>Extraction UI</strong>' +
          '<button id="extraction-close" style="float:right;">X</button>' +
          '<div style="clear:both;"></div>' +
        '</div>' +
        '<div id="extraction-body" style="padding:10px;color:#000;">' +
          '<p>Click elements to toggle selection/deselection.</p>' +
          '<ul id="selection-list" style="list-style:none;padding:0;"></ul>' +
          '<button id="create-markdown" style="margin-top:10px;">Create Markdown</button>' +
          '<button id="save-config" style="margin-top:10px;">Save Config</button>' +
          '<button id="clear-config" style="margin-top:10px;">Clear Saved Config</button>' +
          '<div id="markdown-preview-section" style="display:none;margin-top:10px;">' +
            '<textarea id="markdown-preview" style="width:100%;height:200px;"></textarea>' +
            '<button id="download-markdown" style="margin-top:10px;">Download Markdown</button>' +
          '</div>' +
        '</div>';
      sidebar.style.position = "fixed";
      sidebar.style.top = "0";
      sidebar.style.right = "0";
      sidebar.style.width = "300px";
      sidebar.style.height = "100%";
      sidebar.style.background = "#fff";
      sidebar.style.borderLeft = "1px solid #ccc";
      sidebar.style.zIndex = "9999999";
      sidebar.style.overflowY = "auto";
      sidebar.style.color = "#000";
      document.body.appendChild(sidebar);
      // Add our extra style for highlights.
      var styleElem = document.createElement("style");
      styleElem.innerHTML = ".extraction-highlight { outline: 1px solid blue !important; }";
      document.head.appendChild(styleElem);
      renderSelections();
      setTimeout(applySavedConfig, 1500);
      // Event listeners for element selection.
      var clickHandler = function(e){
        if(e.composedPath().some(function(el){ return el.id === "extraction-sidebar"; })) return;
        if(e.composedPath().some(function(el){ return el.getAttribute && el.getAttribute("data-download-trigger") === "true"; })) return;
        e.preventDefault();
        e.stopPropagation();
        var tgt = e.target;
        if(tgt === document.documentElement || tgt === document.body) return;
        var sel = getUniqueSelector(tgt);
        var lbl = tgt.tagName.toLowerCase() + "-" + (labelCounter++);
        var found = selections.find(function(item){ return item.selector === sel; });
        if(found){
          selections = selections.filter(function(item){ return item.selector !== sel; });
          tgt.classList.remove("extraction-highlight");
          window.__extractionSelections = selections;
          renderSelections();
          return;
        }
        tgt.classList.add("extraction-highlight");
        selections.push({ label: lbl, selector: sel });
        window.__extractionSelections = selections;
        renderSelections();
      };
      document.addEventListener("click", clickHandler, true);
      var keyupHandler = function(e){
        if(e.key === "Escape" || e.keyCode === 27){
          var sb = document.getElementById("extraction-sidebar");
          if(sb){
            clearAllHighlights();
            sb.style.display = "none";
          }
          document.removeEventListener("click", clickHandler, true);
          document.removeEventListener("keyup", keyupHandler, true);
        }
      };
      document.addEventListener("keyup", keyupHandler, true);
      document.getElementById("extraction-close").onclick = function(e){
        e.stopPropagation();
        clearAllHighlights();
        document.getElementById("extraction-sidebar").style.display = "none";
        document.removeEventListener("click", clickHandler, true);
        document.removeEventListener("keyup", keyupHandler, true);
      };
      document.getElementById("create-markdown").onclick = function(e){
        e.stopPropagation();
        var title = document.title;
        var url = window.location.href;
        var d = new Date();
        var dateStr = (d.getMonth()+1).toString().padStart(2,"0") + "-" + d.getDate().toString().padStart(2,"0") + "-" + d.getFullYear();
        var md = "---\ntitle: " + title + "\nurl: " + url + "\ndate: " + dateStr + "\n---\n\n";
        selections.forEach(function(sel){
          var el = document.querySelector(sel.selector);
          var text = el ? el.innerText.trim() : "";
          md += "## " + sel.label + "\n\n" + text + "\n\n";
        });
        var ps = document.getElementById("markdown-preview-section");
        var ta = document.getElementById("markdown-preview");
        ta.value = md;
        ps.style.display = "block";
      };
      document.getElementById("download-markdown").onclick = function(e){
        e.stopPropagation();
        var content = document.getElementById("markdown-preview").value;
        var d = new Date();
        var dateStr = (d.getMonth()+1).toString().padStart(2,"0") + "-" + d.getDate().toString().padStart(2,"0") + "-" + d.getFullYear();
        var slug = slugifyTitle(document.title || "document");
        var fileName = dateStr + "_" + slug + ".md";
        var blob = new Blob([content], { type: "text/markdown;charset=utf-8" });
        var a = document.createElement("a");
        a.href = URL.createObjectURL(blob);
        a.download = fileName;
        a.style.display = "none";
        a.setAttribute("data-download-trigger", "true");
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
      };
      document.getElementById("save-config").onclick = function(e){
        e.stopPropagation();
        if(!selections.length){
          alert("No selections to save!");
          return;
        }
        localStorage.setItem(configKey, JSON.stringify(selections));
        alert("Configuration saved!");
      };
      document.getElementById("clear-config").onclick = function(e){
        e.stopPropagation();
        localStorage.removeItem(configKey);
        clearAllHighlights();
        selections = [];
        window.__extractionSelections = selections;
        renderSelections();
        alert("Saved configuration cleared.");
      };
      window.addEventListener("beforeunload", function(){
        document.removeEventListener("click", clickHandler, true);
        document.removeEventListener("keyup", keyupHandler, true);
      });
    })();
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
- Only tested in Safari and Chrome so far


## License
This project is open-sourced under the MIT License.
