---
name: slideshow-builder
description: >
  Build and maintain a dual-mode HTML document that functions as both a scrollable
  website and a full-screen slideshow. Use this skill whenever the user asks to add
  a slideshow mode, presentation mode, or slide view to an existing HTML guide,
  report, or reference document. Also triggers when the user reports that slides
  have too much content, require scrolling, are too sparse, or need visual/structural
  fixes in the slideshow layer. The skill governs: slide decomposition rules,
  archetype agent testing, the build-test-remediate loop, and GitHub deployment.
---

# Slideshow Builder Skill

## What this skill governs

A dual-mode HTML document has two entirely independent rendering paths sharing one
source of truth. Website mode is the standard scrollable document. Slideshow mode
is a full-screen overlay built by JavaScript reading the live DOM — no content is
duplicated in the HTML.

This skill defines:
1. How content decomposes into slides (the decomposition contract)
2. How to test that decomposition before shipping
3. The agent archetypes to spawn for adversarial review
4. The remediation loop

---

## Decomposition Contract

Every content type maps to a fixed number of slides. Violating this contract produces
the two failure modes: overflow slides (too much scroll) and sparse slides (too little
content).

### Fixed mappings

| Source element | Slides produced | Content per slide |
|---|---|---|
| Skill card | 2 | Slide 1: badge + title + meta grid + rationale (1 para). Slide 2: full-screen dark terminal, prompt text only. |
| Routine card | 2 | Slide 1: index + title + frequency + connector tags + rationale (1 para). Slide 2: full-screen dark terminal, prompt text only. |
| Artifact card | 2 | Slide 1: badge + title + description + connector tags + rationale (1 para). Slide 2: full-screen dark terminal, prompt text only. |
| Workflow block | 2 | Slide 1: num + title + replaces badge + 1 body para + 1 callout max. Slide 2: full-screen dark terminal, prompt text only. |
| Prose section | 1 per ≤3 paragraphs | Section label + h2 (first slide only) + prose block. Never exceed 3 paragraphs per slide. |
| H3 sub-topic | 1 per ≤4 paragraphs | H3 heading + content. Never exceed 4 paragraphs. |
| Appendix divider | 1 | Full-screen dark: eyebrow + h2 + 1-line description only. |
| Title/cover | 1 | Eyebrow + h1 + subtitle + pills. |
| Closing | 1 | Eyebrow + h1 + subtitle + first-task block. |

### Prompt slide rule (critical)

The prompt slide is dark (`#0d0d0b` background). It contains:
- A header bar: badge | title | copy button (right-aligned)
- The prompt text in `DM Mono` 13.5px / 1.85 leading, off-white (`#f0ece4`)
- Nothing else

A prompt slide must never require more than 1.5 screens of vertical scroll on a
1080p display. If the prompt text exceeds ~600 words, split at a natural paragraph
boundary into two prompt slides, labelling them "Part 1" and "Part 2".

### Content slide rule

A context slide must fit without any vertical scroll on a 1080p display (viewport
height ~920px minus topbar 52px, progress 2px, nav 52px = ~814px usable). Budget:
- Card header: ~52px
- Meta grid (3 cells): ~72px
- Connector tags row: ~40px
- Rationale paragraph: ~80px (at 15px/1.7 leading, ~4 lines)
- Padding: ~80px

Total: ~324px. Well within budget. If rationale exceeds 4 lines, truncate in the
slide and keep the full text in website mode.

---

## Archetype Agents

Before shipping any slideshow implementation, spawn three archetype agents and
have each review the built slides. The agents run against the rendered HTML — read
the JS output, not the source.

### Agent 1: The Presenter

Persona: A corporate treasury SVP preparing to present this guide to their team.
They are using a 1920×1080 projector. They advance slides using arrow keys.

Checks to perform:
- Does every slide fit without scrolling on a 1080p display?
- Is the font size readable at 8 feet from a projector?
- Does the slide sequence make narrative sense (context before prompt)?
- Are there any slides with only a header and no body content?
- Are there any slides where the title is cut off?

Assertion: No slide requires vertical scroll. All slides have at least 3 visible
content elements (heading + at least 2 other elements).

### Agent 2: The Skeptic

Persona: A developer who does not trust AI-generated UIs. They will read the
JavaScript decomposition logic line by line and challenge every assumption.

Checks to perform:
- Does `buildSlides()` handle the case where a skill card has no `.prompt-block`?
- Does `buildSlides()` handle cards with no `.skill-meta-row`?
- Does the `goToSlide()` function correctly handle boundary conditions (slide 0, last slide)?
- Does the `proseSlides()` function correctly emit a section label and h2 only on the first chunk?
- Does the copy button inside prompt slides fire correctly after the DOM is built by JS?
- Are there duplicate function definitions in the script block?

Assertion: Each edge case is handled without a JS error. No duplicate function names.

### Agent 3: The Writing Style Judge

Persona: A senior editor applying the house style (no em dashes in prose, no emoji
in callout titles, no hype language, no bullet points in running text).

Checks to perform:
- Scan all `<p>` tags for em dashes not inside `.prompt-text` or terminal blocks.
- Scan all `.callout-title` elements for emoji characters.
- Scan all prose for: "game-changer", "revolutionary", "cutting-edge", "powerful",
  "transformative", "groundbreaking", "seamless".
- Check that no callout-title uses an emoji prefix.

Assertion: Zero em dashes in prose paragraphs. Zero emoji in callout titles.
Zero hype words.

---

## Build-Test-Remediate Loop

### Phase 1: Plan

Before writing any code:
1. List every content type present in the document (skill cards, routine cards,
   artifact cards, workflows, prose sections, appendix dividers).
2. Calculate the expected slide count using the decomposition contract.
3. Write the expected count to a planning note: "Expected: N slides."
4. Identify any content elements that may violate the prompt slide word limit.
   List them by name.

### Phase 2: Build

Implement the JS engine following the decomposition contract exactly.
After building, verify:
- The `buildSlides()` function has one definition only.
- The `fallbackCopy()` function is defined before it is called.
- The `goToSlide()` transition logic handles both forward and backward directions.
- The `type-prompt` CSS class exists and sets background to `#0d0d0b`.
- The `ss-prompt-text` element uses `white-space: pre-wrap` and `#f0ece4` color.

### Phase 3: Static Test (Skeptic Agent)

Run the Skeptic agent checks against the built JS. Specifically:

```bash
# Check for duplicate function definitions
grep -c "^function buildSlides" index.html
grep -c "^function goToSlide" index.html
grep -c "^function exitSlideshow" index.html
# Each should return 1
```

```bash
# Check for em dashes in prose
grep -n "—" index.html | grep "<p>" | grep -v "prompt-text\|ss-prompt\|\.doc\|— \[" | wc -l
# Should return 0
```

```bash
# Check for emoji in callout titles
python3 -c "
import re, sys
html = open('index.html').read()
titles = re.findall(r'callout-title[^>]*>([^<]+)', html)
emoji_pattern = re.compile('[\\U00010000-\\U0010ffff\\U00002600-\\U000027BF]', flags=re.UNICODE)
violations = [t for t in titles if emoji_pattern.search(t)]
print('Violations:', violations)
print('Count:', len(violations))
"
```

All three checks must pass (counts = 0) before proceeding to Phase 4.

### Phase 4: Presenter Agent Review

Open the rendered HTML in a headless browser simulation. For each slide in the deck:
- Measure content height: `document.querySelector('.ss-slide[data-index="N"] .ss-slide-inner').scrollHeight`
- Compare to usable viewport height (814px)
- Log any slide where `scrollHeight > 814`

If any overflow slides are found, apply remediation before Phase 5.

### Phase 5: Writing Style Agent Review

Run the style checks. If any violations are found, fix them before pushing.

### Phase 6: Ship

Only push to GitHub after all three agent checks pass with zero violations.

---

## Remediation Patterns

### Overflow on a prompt slide
Split the prompt text at a paragraph boundary. Create two prompt slides labelled
"[Title] — Part 1" and "[Title] — Part 2". Each part should be roughly equal length.

### Overflow on a context slide
The rationale paragraph exceeds 4 lines. Truncate to the first 3 sentences. Add
a `title` attribute with the full text so it appears on hover.

### Sparse slide (only header, no body)
Indicates a card with no rationale paragraph. Add a one-sentence rationale to the
source card in website mode. It will propagate automatically to the slide.

### Duplicate function definition
Remove the second definition entirely. Verify the remaining definition is the
correct (newer) version by checking its logic against the decomposition contract.

### Em dash in prose
Replace with: comma (parenthetical insertions), semicolon (independent clauses),
colon (introducing a list or elaboration), or period (if two thoughts can stand alone).

---

## CSS Requirements Checklist

Before shipping, verify these CSS rules exist:

- [ ] `.ss-slide.type-prompt { background: #0d0d0b; }`
- [ ] `.ss-slide.type-prompt .ss-prompt-text { color: #f0ece4; white-space: pre-wrap; font-size: 13.5px; line-height: 1.85; }`
- [ ] `.ss-slide.type-content { background: var(--paper); }`
- [ ] `.ss-slide.type-section { background: var(--paper); }`
- [ ] `.ss-slide.type-title { background: var(--ink); align-items: center; }`
- [ ] `.ss-slide.type-appendix { background: #0a0a09; align-items: center; }`
- [ ] `.ss-slide { overflow-y: auto; }` — allows scroll as safety net, not default
- [ ] `.ss-progress-fill { transition: width 0.35s ease; }`
- [ ] `.ss-nav-btn:disabled { opacity: 0.2; cursor: default; }`

---

## GitHub Deployment Gate

The following must be true before `git push`:

1. Static test (Phase 3) passes: 0 duplicate functions, 0 prose em dashes, 0 emoji in callout titles.
2. Presenter check (Phase 4) passes: 0 slides with `scrollHeight > 814`.
3. Writing style check (Phase 5) passes: 0 hype words in prose.
4. `wc -l index.html` is less than 4500 lines (guard against runaway duplication).
5. `curl -s -o /dev/null -w "%{http_code}" https://shertokj.github.io/corwork_finance_guide/` returns 200 after deployment.

---

## Functional Flow Tests (Phase 3 Extension)

The static Skeptic agent checks are necessary but not sufficient. The following
functional tests must also pass. They verify the runtime behaviour of the
slideshow — the flows a user actually exercises — not just the code structure.

### Root Cause of the Slideshow Button Failure (May 2026)

The page-load prompt-block transformer ran at DOMContentLoaded and replaced every
`.prompt-block`'s innerHTML, destroying the `<p>` tags. The slide engine called
`getPromptText()` on click — after the transformer had already run. It found no
`<p>` elements and returned empty strings. Every prompt slide rendered blank.

**The fix:** stash all prompt data into `data-prompt-text` and `data-prompt-label`
attributes BEFORE the transformer mutates the DOM. The slide engine then reads from
`dataset.promptText` and `dataset.promptLabel`, which survive the transformation.

**The lesson for the skill:** static pattern checks cannot catch timing bugs.
Functional flow tests must verify the execution order and the data path.

### F1 — Execution Order

```python
stash_idx    = html.find('STEP 1: STASH PROMPT DATA')
transform_idx = html.find('STEP 2: RENDER TERMINAL UI')
engine_idx   = html.find('SLIDESHOW ENGINE')
assert stash_idx < transform_idx < engine_idx, "Execution order violated"
```

### F2 — Data Path: getPromptText reads dataset, not DOM

```python
get_fn_body = extract_function_body(html, 'getPromptText')
assert 'dataset.promptText' in get_fn_body, "Must read from dataset"
assert "querySelector('p" not in get_fn_body, "Must NOT query <p> tags (destroyed by transformer)"
assert "querySelector(\"p" not in get_fn_body, "Must NOT query <p> tags"
```

### F3 — Data Path: getPromptLabel reads dataset, not DOM

```python
get_lbl_body = extract_function_body(html, 'getPromptLabel')
assert 'dataset.promptLabel' in get_lbl_body, "Must read from dataset"
assert "querySelector('.prompt-label')" not in get_lbl_body, "Must NOT query .prompt-label (destroyed)"
```

### F4 — enterSlideshow() flow is intact

```python
enter_body = extract_function_body(html, 'enterSlideshow')
assert 'buildSlides()' in enter_body, "Must call buildSlides"
assert "classList.add('active')" in enter_body, "Must show overlay"
assert "overflow = 'hidden'" in enter_body, "Must lock page scroll"
```

### F5 — Button exists and calls enterSlideshow

```python
assert re.search(r"onclick=['\"]enterSlideshow\(\)['\"]", html), "Button must exist"
```

### F6 — Overlay elements exist exactly once

```python
assert html.count('id="slideshowOverlay"') == 1, "Exactly one overlay"
assert html.count('id="ssViewport"') == 1, "Exactly one viewport"
```

### F7 — Copy buttons carry data-text

```python
copy_btns = re.findall(r'class="prompt-copy-btn"', html)
data_btns  = re.findall(r'class="prompt-copy-btn"[^>]*data-text', html)
assert len(copy_btns) == len(data_btns), "Every copy button must have data-text"
```

### F8 — exitSlideshow restores scroll

```python
exit_body = extract_function_body(html, 'exitSlideshow')
assert "overflow = ''" in exit_body, "Must restore page scroll on exit"
```

### F9 — buildSlides has double-build guard

```python
assert 'SS.built = true' in html, "Must set built flag"
assert 'if (SS.built) return' in html, "Must guard against double build"
```

### Running functional tests

Add these checks to Phase 3 (Static Test), immediately after the duplicate
function checks. All F1–F9 tests must pass before proceeding to Phase 4.

```bash
python3 << 'PYEOF'
import re, sys
html = open('index.html').read()

def check(label, condition):
    print(f"  {label}: {'✓' if condition else 'FAIL'}")
    if not condition: sys.exit(1)

def fn_body(name):
    m = re.search(rf'function {name}\([^)]*\)\s*\{{(.*?)\}}', html, re.DOTALL)
    return m.group(1) if m else ''

check('F1 stash before transform', html.find('STEP 1') < html.find('STEP 2') < html.find('SLIDESHOW ENGINE'))
check('F2 getPromptText reads dataset', 'dataset.promptText' in fn_body('getPromptText'))
check('F2 getPromptText not reading p tag', "querySelector('p" not in fn_body('getPromptText'))
check('F3 getPromptLabel reads dataset', 'dataset.promptLabel' in fn_body('getPromptLabel'))
check('F4 enterSlideshow calls buildSlides', 'buildSlides()' in fn_body('enterSlideshow'))
check('F4 enterSlideshow shows overlay', "classList.add('active')" in fn_body('enterSlideshow'))
check('F4 enterSlideshow locks scroll', "overflow = 'hidden'" in fn_body('enterSlideshow'))
check('F5 button calls enterSlideshow', bool(re.search(r"onclick=['\"]enterSlideshow\(\)['\"]", html)))
check('F6 one overlay', html.count('id="slideshowOverlay"') == 1)
check('F6 one viewport', html.count('id="ssViewport"') == 1)
check('F8 exitSlideshow restores scroll', "overflow = ''" in fn_body('exitSlideshow'))
check('F9 built guard', 'SS.built = true' in html and 'if (SS.built) return' in html)
print('All functional tests passed.')
PYEOF
```

---

## Updated Deployment Gate (8 + 9 checks)

The original 8 gates plus:
- [ ] F1 Execution order: stash → transform → engine
- [ ] F2 getPromptText reads `dataset.promptText`, not `<p>`
- [ ] F3 getPromptLabel reads `dataset.promptLabel`
- [ ] F4 enterSlideshow calls buildSlides, shows overlay, locks scroll
- [ ] F5 At least one button with `onclick="enterSlideshow()"`
- [ ] F6 Exactly one `#slideshowOverlay` and one `#ssViewport`
- [ ] F8 exitSlideshow sets `body.style.overflow = ''`
- [ ] F9 buildSlides has `SS.built` guard

---

## Mandatory End-to-End DOM Simulation Test

Static parse checks and pattern matching are insufficient. The following test
must be run before every push. It simulates an actual button click in a headless
DOM, verifying the full user flow: script execution → enterSlideshow() → overlay
visible → slides populated → navigation works.

This test caught the primary cause of the "button doesn't work" failure: a script
syntax error from orphaned top-level code that caused the browser to silently drop
the entire script block. No static check caught it; only executing the script did.

### What this test verifies

1. Script block parses without syntax errors (`new Function(scriptSrc)`)
2. Script executes without runtime errors in a mocked DOM environment
3. `enterSlideshow()` is callable (not undefined)
4. After calling `enterSlideshow()`:
   - `#slideshowOverlay` has `active` class
   - `document.body.style.overflow === 'hidden'`
   - `#ssViewport` has at least 3 child slide elements
   - `#ssTotalNum` text content equals slide count
   - First slide has `active` class
5. Calling `slideshowNav(1)` moves to slide 2 without throwing

### How to run it

Save this as `test_slideshow.js` and run `node test_slideshow.js` from the
directory containing `index.html`. All output lines must show `✓`. Any `FAIL`
or `✗` line blocks the push.

```javascript
// test_slideshow.js
const fs = require('fs');
const html = fs.readFileSync('index.html', 'utf8');
const scriptSrc = html.match(/<script>([\s\S]*?)<\/script>/)[1];

// 1. Parse check
try { new Function(scriptSrc); console.log('✓ Script parses'); }
catch(e) { console.error('✗ PARSE FAIL:', e.message); process.exit(1); }

// 2. Brace balance
let depth = 0;
for (const line of scriptSrc.split('\n')) {
  if (line.trim().startsWith('//')) continue;
  for (const ch of line) {
    if (ch === '{') depth++;
    else if (ch === '}') { depth--; if (depth < 0) { console.error('✗ Extra } found'); process.exit(1); } }
  }
}
if (depth !== 0) { console.error('✗ Brace imbalance:', depth); process.exit(1); }
console.log('✓ Brace balance: 0');

// 3. DOM simulation (minimal mock — enough to exercise the button click path)
function el(tag, opts) {
  opts = opts || {};
  const node = {
    tagName: tag.toUpperCase(), id: opts.id||'', className: opts.className||'',
    textContent:'', innerHTML:'', style:{}, children:[], parentNode:null,
    scrollTop:0, disabled:false, dataset:{},
    appendChild(c){ c.parentNode=this; this.children.push(c); return c; },
    addEventListener(){},
    cloneNode(deep){ const c=el(this.tagName,{id:this.id,className:this.className}); c.dataset=Object.assign({},this.dataset); c.textContent=this.textContent; if(deep) this.children.forEach(ch=>{const cc=ch.cloneNode(true);cc.parentNode=c;c.children.push(cc);}); return c; },
    querySelectorAll(sel){ const r=[]; const walk=(n)=>{ if(n!==this&&sel.split(', ').some(s=>this._matches(n,s.trim()))) r.push(n); n.children.forEach(walk); }; walk(this); return r; },
    querySelector(sel){ return this.querySelectorAll(sel)[0]||null; },
    closest(sel){ let n=this; while(n){ if(this._matches(n,sel)) return n; n=n.parentNode; } return null; },
    _matches(n,sel){ if(!n||!n.tagName) return false; if(sel.startsWith('.')) return n.className&&n.className.split(' ').includes(sel.slice(1)); if(sel.startsWith('#')) return n.id===sel.slice(1); return n.tagName===sel.toUpperCase(); },
  };
  node.classList={
    add(c){ if(!node.className.split(' ').includes(c)) node.className=(node.className+' '+c).trim(); },
    remove(c){ node.className=node.className.split(' ').filter(x=>x!==c).join(' '); },
    contains(c){ return node.className.split(' ').includes(c); },
    toggle(c,f){ (f===undefined?this.contains(c):!f)?this.remove(c):this.add(c); },
  };
  return node;
}

// Build minimal DOM
const overlay=el('div',{id:'slideshowOverlay'});
const viewport=el('div',{id:'ssViewport'});
const thumbStrip=el('div',{id:'ssThumbStrip'});
const progress=el('div',{id:'ssProgressFill'}); progress.style={width:''};
const curNum=el('span',{id:'ssCurrentNum'}); curNum.textContent='1';
const totalNum=el('span',{id:'ssTotalNum'});
const prevBtn=el('button',{id:'ssPrevBtn'});
const nextBtn=el('button',{id:'ssNextBtn'});
const byId={slideshowOverlay:overlay,ssViewport:viewport,ssThumbStrip:thumbStrip,ssProgressFill:progress,ssCurrentNum:curNum,ssTotalNum:totalNum,ssPrevBtn:prevBtn,ssNextBtn:nextBtn};

// Parse prompt blocks from HTML so dataset stash has data
const blocks=[];
for(const m of html.matchAll(/<div class="prompt-block">([\s\S]*?)<\/div>\s*\n?\s*<\/div>\s*\n?\s*<\/div>/g)){
  const pb=el('div',{className:'prompt-block'});
  const lm=m[1].match(/class="prompt-label"[^>]*>(.*?)<\/div>/);
  const pm=m[1].match(/<p>([\s\S]*?)<\/p>/);
  pb.dataset.promptLabel=lm?lm[1].replace(/<[^>]+>/g,'').trim():'Prompt';
  pb.dataset.promptText=pm?pm[1].replace(/<[^>]+>/g,'').trim():'';
  blocks.push(pb);
}

// Minimal section for buildSlides traversal
const bodyWrap=el('div',{className:'body-wrap'});
const sec=el('section',{id:'overview'});
const ph=el('div',{className:'part-header'});
const plEl=el('span',{className:'part-label'}); plEl.textContent='Part One';
const h2El=el('h2'); h2El.textContent='Overview';
ph.appendChild(plEl); ph.appendChild(h2El);
const p1=el('p'); p1.textContent='Test prose.';
sec.appendChild(ph); sec.appendChild(p1);
bodyWrap.appendChild(sec);

const mockDoc={
  body:{style:{}},
  createElement:(tag)=>el(tag),
  getElementById:(id)=>byId[id]||null,
  addEventListener:()=>{},
  querySelectorAll:(sel)=>{
    if(sel==='.prompt-block') return blocks;
    if(sel==='.body-wrap > section') return [sec];
    if(sel==='.appendix-divider') return [];
    if(sel==='.ss-slide') return viewport.children;
    if(sel==='.ss-thumb') return thumbStrip.children;
    return [];
  },
};

// 4. Execute script and simulate click
const ref={};
try {
  new Function('document','window','navigator','requestAnimationFrame','setTimeout','console','__r',
    scriptSrc+'\n__r.enter=enterSlideshow;__r.nav=slideshowNav;'
  )(mockDoc,{},{clipboard:null},(cb)=>cb(),(fn)=>fn(),console,ref);
  console.log('✓ Script executes');
} catch(e) { console.error('✗ RUNTIME ERROR:', e.message); process.exit(1); }

ref.enter();
const ok = (label, cond) => console.log((cond?'✓ ':'✗ ')+label) || (!cond && process.exit(1));
ok('overlay active', overlay.classList.contains('active'));
ok('body overflow hidden', mockDoc.body.style.overflow==='hidden');
ok('slides built', viewport.children.length >= 3);
ok('total counter set', totalNum.textContent.length > 0);
ok('first slide active', viewport.children[0]&&viewport.children[0].classList.contains('active'));
ref.nav(1);
ok('navigation moves to slide 2', curNum.textContent === '2');
console.log('✓ ALL TESTS PASSED');
```

### When to run this test

- Before every `git push`
- After any edit to the `<script>` block
- After any str_replace or python-based edit to the HTML file
- After any "cleanup" of duplicate functions

