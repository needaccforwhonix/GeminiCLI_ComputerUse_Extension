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
