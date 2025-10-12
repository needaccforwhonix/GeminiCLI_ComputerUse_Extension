# GeminiCLI_ComputerUse_Extension
A Google Gemini-CLI extension than enables Gemini Computer Use from the CLI

## Installation

gemini extensions install https://github.com/automateyournetwork/GeminiCLI_ComputerUse_Extension.git

### Example Gemini-CLI prompt to VISIBLY browse a web article and produce a Markdown report
Use ComputerUse MCP headfully to VISIBLY browse like a human and produce a final Markdown report.

Goals:
- Read the article at the given URL.
- Scroll like a human (visible motion), taking a snapshot after EACH scroll step.
- Visit all in-article links (avoid mailto/tel/#). Prefer opening in NEW TABS that do not overtake the main article tab. If tab tools are unavailable, open links sequentially and return to the article each time.
- Conclude with a comprehensive Markdown summary (RAFT, retrieval-augmented fine-tuning, key takeaways), citing the pages you visited.

1) initialize_browser(
     url="https://www.automateyournetwork.ca/pyats/augmenting-network-engineering-with-raft/",
     width=1920, height=1080, headless=false)

2) Smoothly scroll the article TOP→BOTTOM with visible pauses. After EACH step, take a snapshot:
   for y in [200, 400, 600, 800, 1000]:
     - execute_action("scroll_to_percent", {"y": y})
     - capture_state(f"scroll_{y}")

3) Return to TOP visibly and snapshot:
   - execute_action("scroll_to_percent", {"y": 0})
   - capture_state("back_to_top")

4) Harvest in-article links (de-dupe, skip mailto/tel/#):
   - execute_action("execute_javascript", {
       "code": "(() => { \
         const scope = document.querySelector('article, main, .post-content, .entry-content') || document.body; \
         const links = Array.from(scope.querySelectorAll('a[href]')); \
         const hrefs = links.map(a => a.href.trim()) \
           .filter(h => h && !h.startsWith('mailto:') && !h.startsWith('tel:') && !h.includes('#')); \
         return Array.from(new Set(hrefs)); \
       })();"
     })

5) Visit each harvested link (limit to 8 to stay readable):
   Preferred (if these tools exist): open_new_tab, list_tabs, switch_to_tab
   - For each <link> (1-based index i):
       - If open_new_tab exists:
           - open_new_tab(url="<link>", focus=true)
           - capture_state(f"link{i}_load")                      # snapshot immediately after load
           - For y in [200, 400, 600, 800, 1000]:
                  execute_action("scroll_to_percent", {"y": y})
                  capture_state(f"link{i}_scroll_{y}")           # snapshot each scroll step
           - switch_to_tab(0)                                   # return to main article tab
       - Else (fallback when tab tools absent):
           - execute_action("open_web_browser", {"url": "<link>"})
           - capture_state(f"link{i}_load")
           - For y in [200, 400, 600, 800, 1000]:
                  execute_action("scroll_to_percent", {"y": y})
                  capture_state(f"link{i}_scroll_{y}")
           - execute_action("open_web_browser", {"url": "https://www.automateyournetwork.ca/pyats/augmenting-network-engineering-with-raft/"})
           - capture_state(f"link{i}_return")                   # confirm we’re back on the main article

6) After links, return to the main article (if not already there) and capture_state("final_overview").

7) Produce a comprehensive Markdown report (no extra screenshots in the report; just text). Structure:
   # RAFT & Retrieval-Augmented Fine-Tuning — Field Notes
   - **Primary article:** title + URL
   - **Other pages visited:** bullet list of titles + URLs
   - **What RAFT is:** 3–6 bullets (your own words)
   - **How RAFT differs from standard fine-tuning:** bullets
   - **Retrieval-Augmented Fine-Tuning pipeline:** concise steps (data prep, retrieval store, adapters/LoRA/full FT, eval)
   - **Cloud vs Local comparison (from the two-part series):** capabilities, privacy, cost, constraints
   - **Implementation notes spotted in pages:** tools, commands, pitfalls
   - **Key takeaways:** 5–8 bullets
   - **References (visited):** list of URLs

Important:
- Keep actions human-visible (no instant jumps).
- If a page won’t load, skip it and continue.
- Take a small pause between scroll steps so motion is obvious.

### Example Searching Wikipedia for "Computer Networking" and producing a Markdown report
Use ComputerUse MCP headfully to VISIBLY search Wikipedia for “Computer Network” and explore related articles.

Goals:
- Launch browser visibly (not headless).
- Search Wikipedia for “Computer Network” using omnibox keyboard shortcuts.
- Scroll through search results like a human, taking snapshots after each step.
- Visit up to 8 relevant links, scrolling and taking screenshots on each sub-page.
- Conclude with a Markdown summary of what was explored.

---

1) initialize_browser(
     url="https://wikipedia.org",
     width=1920, height=1080, headless=false)

2) Type into the homepage search box (already focused on wikipedia.org):
   - execute_action("type_text_at", {
       "x": 0, "y": 0,
       "text": "Computer Networking",
       "press_enter": true
     })

   # Fallback only if no results load (input wasn’t focused):
   - execute_action("execute_javascript", {
       "code": "(()=>{const el=document.querySelector('#searchInput, input[type=search], input[name=search]'); if(!el) return false; el.focus(); el.value=''; return true;})()"
     })
   - execute_action("type_text_at", {
       "x": 0, "y": 0,
       "text": "Computer Networking",
       "press_enter": true
     })

3) Scroll results page TOP→BOTTOM with visible pauses, taking a snapshot after each step:
   for y in [200, 400, 600, 800, 1000]:
     - execute_action("scroll_to_percent", {"y": y})
     - capture_state(f"results_scroll_{y}")

4) Return to top and snapshot:
   - execute_action("scroll_to_percent", {"y": 0})
   - capture_state("results_back_to_top")

5) Harvest result links (de-dupe, skip non-wiki targets):
   - execute_action("execute_javascript", {
       "code": "(() => { \
         const scope = document.querySelector('#mw-content-text') || document.body; \
         const links = Array.from(scope.querySelectorAll('a[href]')); \
         const hrefs = links.map(a => a.href.trim()) \
           .filter(h => h.includes('wikipedia.org/wiki/') && !h.includes('#')); \
         return Array.from(new Set(hrefs)).slice(0,8); \
       })();"
     })

6) Visit each harvested link (limit 8):
   Preferred (if open_new_tab exists):
     - For each <link> i:
         - open_new_tab(url="<link>", focus=true)
         - capture_state(f"link{i}_load")
         - For y in [200, 400, 600, 800, 1000]:
               execute_action("scroll_to_percent", {"y": y})
               capture_state(f"link{i}_scroll_{y}")
         - switch_to_tab(0)
   Fallback:
     - execute_action("open_web_browser", {"url": "<link>"})
     - capture_state(f"link{i}_load")
     - For y in [200, 400, 600, 800, 1000]:
           execute_action("scroll_to_percent", {"y": y})
           capture_state(f"link{i}_scroll_{y}")
     - execute_action("open_web_browser", {"url": "https://wikipedia.org/wiki/Computer_network"})
     - capture_state(f"link{i}_return")

7) After visiting links, ensure we’re back on the main results page and take a final snapshot:
   - capture_state("final_overview")

8) Produce a concise Markdown report titled:
   # Wikipedia Exploration — Computer Networks

   Include:
   - **Primary search query & URL**
   - **List of visited articles (titles + URLs)**
   - **What you learned about Computer Networks** — 5–8 bullets
   - **Key subtopics found** (e.g., protocols, topology, history, standards)
   - **Notable cross-references or linked topics**
   - **References (all URLs visited)**

---

Important:
- Keep motion visible; pause slightly between scrolls.
- Take snapshots after **every** scroll and page load.
- If a link fails, skip gracefully.
- Do not summarize until step 8.
