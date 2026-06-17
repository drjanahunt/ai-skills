---
name: barometer-report
description: >
  Use this skill whenever the user asks to analyse, write, or produce a report from barometer 
  survey data — including MSE (micro and small enterprise) barometers, SME surveys, entrepreneur 
  research, or any structured longitudinal survey dataset covering business performance, 
  finance, digitisation, cybersecurity, gender, or support services. Trigger this skill when 
  the user mentions barometer data, survey raw data files (.xlsx), or asks to create a 
  report from research data. Also trigger when the user asks to "analyse the data" alongside 
  an uploaded spreadsheet that appears to be survey or barometer data.
---

# Barometer Report Skill

A structured workflow for producing high-quality analytical reports from raw barometer/survey 
data — informed by a critical comparison between an AI-generated draft and a professionally 
produced 60 Decibels / Mastercard Czechia MSE Barometer report.

---

## CRITICAL: Read This Before Touching the Data

The gap between a competent first-pass analysis and a research-quality report comes down to 
five things: correct terminology, longitudinal context, segmentation depth, precise 
denominator discipline, and calibrated conclusions. Every rule below exists because one of 
these was wrong in a prior draft.

---

## Step 0 — Understand the Study Design Before Analysis

Before running any code:

1. **Identify the population and scope.** What businesses are in scope? MSEs (micro and small, 
   up to 49 employees) are NOT the same as SMEs (small and medium, which includes up to 250 
   employees). Using the wrong term invalidates the framing.

2. **Check if this is a longitudinal study.** Are there prior waves? If yes, the most important 
   findings are often *trends*, not point-in-time numbers. Collect prior wave data before 
   analysis. A number that hasn't improved across two waves is more damning than the number itself.

3. **Get the sampling design.** Was any group intentionally oversampled? (e.g. small businesses 
   oversampled for statistical reliability.) This affects how you interpret group-level findings.

4. **Read the codebook or survey script first.** Understand what each question actually measures, 
   especially: which questions are multi-select, which are only asked to subgroups, and what 
   the exact response options are.

---

## Step 1 — Data Exploration (Run These Every Time)

```python
import pandas as pd

df = pd.read_excel('file.xlsx', header=0)
# Row 0 is typically question labels, row 1+ is data
questions = df.iloc[0]   # question descriptions
data = df.iloc[1:].reset_index(drop=True)

print(f'Respondents: {len(data)}')
print(f'Columns: {list(df.columns)}')
```

### Mandatory cross-tabs to run on EVERY key variable:

The primary segmentation variables are almost always:
- **Business size**: solopreneur / micro (1-9 employees) / small (10-49 employees)
- **Gender**: male-led / female-led / jointly led
- **Region**: capital city vs. other regions
- **Business tenure**: young (<6 years) / mid (7-12 years) / established (13+ years)

Do NOT report a headline number without checking whether it varies significantly across these 
segments. If it does, the segment finding is often the real story.

---

## Step 2 — Denominator Discipline (Non-Negotiable)

Wrong denominators are the most common source of inaccurate findings. Follow these rules:

| Question type | Correct denominator |
|---|---|
| Multi-select question | Total valid responses to that question |
| Subgroup question (e.g. women only) | Only that subgroup |
| Follow-up question (e.g. "if yes...") | Only those who answered yes |
| Cross-tab | Row total for that segment |

**For multi-select questions**, disaggregate the responses and count each option independently:
```python
from collections import Counter
responses = data['Q_column'].dropna()
all_items = []
for r in responses:
    all_items.extend([x.strip() for x in str(r).split(',')])
counts = Counter(all_items)
```

---

## Step 3 — Key Metrics Checklist

These are the core metrics in a standard MSE/SME barometer. Compute all of them.

### Financial Health
- [ ] Primary source of income (% reliant solely on business)
- [ ] Annual revenue distribution (by size and gender)
- [ ] Annual profit distribution
- [ ] Revenue change last 12 months (very much increased / slightly increased / no change / 
      slightly decreased / very much decreased)
- [ ] Ability to pay bills on time (same 5-point scale)
- [ ] Financial runway: median months, % with <3 months, % with 0 reserves
- [ ] Financial worry trend

### Credit and Finance Access
- [ ] % who attempted to access credit
- [ ] Of those who tried: % successful
- [ ] Top barriers to loan application (multi-select, ranked)
- [ ] Gender gap in credit access attempts (men vs. women %)
- [ ] Size gap (solopreneurs vs. micro/small %)

### Digitisation
- [ ] % of investment budget allocated to digitisation (by gender and size)
- [ ] Digital tools currently used (multi-select, ranked)
- [ ] Barriers to technology adoption (multi-select, ranked)

### Cybersecurity
- [ ] Importance rating (very/somewhat/neither/somewhat un/very unimportant)
- [ ] % who have experienced an incident
- [ ] Cybersecurity measures currently in place (multi-select — DO NOT collapse to "has a system")
- [ ] Willingness to invest monthly (CZK bands)
- [ ] Barriers to better cybersecurity (multi-select, ranked)
- [ ] Support needs (multi-select, ranked)
- [ ] Gender differences in both concern type and investment willingness

### Support Services
- [ ] % who accessed no support services
- [ ] Support services accessed (multi-select, ranked)
- [ ] Prioritisation of support for future growth (1-5 scale, mean by category)
- [ ] Gender gap in service access

### Growth and Aspirations
- [ ] Growth expectations (rapidly / moderately / same / downsize / close temporarily / 
      close permanently)
- [ ] Growth expectations by size and gender
- [ ] Growth factor importance ratings (1-5 scale, mean)
- [ ] Business environment perception (3-year change)
- [ ] Confidence change (3-year)
- [ ] Digital tools availability change (3-year)
- [ ] Support services availability change (3-year)

### Gender-Specific
- [ ] Challenges women-run businesses face (women respondents only, multi-select)
- [ ] % who agree men have an advantage (overall AND by gender — the gender gap within this 
      is often the most striking finding)

---

## Step 4 — Cybersecurity: Common Mistakes to Avoid

This section is almost always misread. Key distinctions:

**DO NOT collapse all cybersecurity measures into "has a system."** Report individual measures:
- Antivirus (typically ~68%)
- Regular software updates (~59%)
- Two-factor authentication (~58%)
- Firewalls (~42%)
- Employee training (~34%)
- Data encryption (~9%)

The story is: most businesses have basic hygiene but not comprehensive protection. That is 
different from "only 30% have a system."

**DO report** the finding that businesses who have experienced an incident are significantly 
more willing to invest afterwards — this is a behaviour-change signal that matters for 
programme design.

**DO segment** cybersecurity concern by type: women-led businesses worry more about identity 
theft; men-led businesses worry more about malware. This shapes what support to offer whom.

---

## Step 5 — Longitudinal Framing (If Prior Waves Exist)

If this is part of a series, frame every major finding as a trend, not a snapshot:

**Template sentences:**
- "This figure has remained unchanged from [year], suggesting [interpretation]."
- "This represents a positive shift from [X]% in [year] to [X]% in [year]."
- "Despite [intervention], [metric] has not improved over [N] years — indicating 
  [structural rather than behavioural] barriers."

Trend findings that haven't moved are often more important than the numbers themselves. 
A financial vulnerability figure stuck at 1 in 5 across two years is an accountability 
finding, not just a statistic.

---

## Step 6 — Framing and Calibration Rules

These rules prevent overclaiming, which undermines credibility:

1. **Stay close to the data on recommendations.** If the data shows low cybersecurity 
   investment, recommend affordable tools and subsidies — do not specify programme structures 
   (e.g. "equivalent to UK Cyber Essentials") unless the data specifically supports that 
   level of prescription.

2. **Report the full distribution, not just the top box.** "70% consider cybersecurity 
   important" is a different statement from "33% consider it very important and 36% somewhat 
   important." Both are true; the second is more precise and honest.

3. **Distinguish aversion from inability.** Low credit uptake (e.g. 21% attempted) is NOT 
   just a structural access problem — it also reflects psychological aversion to debt. Both 
   need different interventions. Don't collapse them into one finding.

4. **Do not overstate environmental pessimism.** "41% say conditions worsened" is accurate. 
   "The business environment has deteriorated" is an editorial claim that goes beyond the data 
   (36% said no change, 22% said improved).

5. **Never use "SME" when the study is about MSEs.** They are different populations with 
   different characteristics and different support needs. The whole point of MSE-focused 
   research is to separate them from medium enterprises.

---

## Step 7 — Report Structure

Follow this sequence. The "11 Lessons" or "Key Findings" section at the front is the most 
important for senior readers — write it last, but place it first.

```
1. Cover page
2. About the programme / About the research organisation
3. Foreword / Opening remarks (if applicable)
4. Introduction — purpose and scope
5. Key Findings / Lessons (8-12 headline findings, each 2-3 sentences)
6. Sample and Methodology
   - Survey details (dates, language, screening)
   - Sample composition (size, gender, region, sector, tenure, size)
   - Sampling notes (oversampling, limitations)
   - Regional analysis approach
7. Data Analysis — structured by theme:
   a. Current state / business health
   b. Digitisation (with cybersecurity as a spotlight section)
   c. Finance and credit access
   d. Support services
   e. Business landscape perceptions
   f. Aspirations and growth outlook
8. Conclusions (one per major theme, grounded in specific data)
9. Recommendations
   - Public sector
   - Business support providers
   - Financial institutions (if applicable)
10. Expert comments / Case studies (if available)
11. Annex — survey script
```

---

## Step 8 — Writing Standards

### Terminology
- **MSE** = micro and small enterprise (up to 49 employees), includes solopreneurs
- **Solopreneur** = sole trader with no employees
- **Micro business** = 1-9 employees
- **Small business** = 10-49 employees
- Never use "SME" for this population unless the study explicitly includes medium enterprises

### Headline finding format
Each key finding should state: the finding + the number + the implication. Example:
> "Entrepreneurs want support services like technology and business training but do not 
> actively seek them out. Despite recognising the value of training, 1 in 2 entrepreneurs 
> did not access any support services in the past year."

### Avoid
- Collapsing nuanced distributions into single percentages
- Describing structural problems as behavioural ones (or vice versa)
- Recommendations that go beyond what the data can support
- Ignoring segmentation to report only headline averages
- Reporting cybersecurity as binary (has system / doesn't) when the measures data is available

---

## Step 9 — Document Production

Read `/mnt/skills/public/docx/SKILL.md` before generating the Word document.

Key formatting decisions for barometer reports:
- Use consistent colour scheme (suggest brand colours of commissioning organisation)
- Include data tables for all major metrics — don't just describe in prose
- Each thematic section: brief intro → data table or key numbers → interpretation → 
  callout box for the "so what"
- Callout boxes for particularly striking findings (the ones a busy policy reader needs 
  to see without reading the full section)
- Page breaks between major sections
- Running header: report title | year
- Footer: page numbers + confidentiality note if applicable

---

## Checklist Before Finalising

- [ ] Used "MSE" not "SME" throughout (or whichever is correct for this study)
- [ ] Every headline figure has been cross-tabbed by size, gender, and region
- [ ] Multi-select denominators are respondents to the question, not total sample
- [ ] Subgroup questions use only the relevant subgroup as denominator
- [ ] Cybersecurity measures reported individually, not collapsed
- [ ] Longitudinal trend called out wherever prior wave data exists
- [ ] Credit finding distinguishes aversion from inability
- [ ] Recommendations stay close to the data
- [ ] Gender finding includes % who agree men have advantage, broken out by gender
- [ ] Financial runway reported as median months + % with zero reserves + % with <3 months
- [ ] Key findings section written (last, placed first)
- [ ] Written in first person plural ("we") throughout
- [ ] Gender gaps attributed to structural causes, never individual behaviour
- [ ] Stagnant findings stated plainly as accountability findings, not softened
- [ ] Every recommendation names a specific actor as subject
- [ ] Document validated after creation

---

## Style Reference: "Striving to Thrive" Barometer Series

*Based on direct analysis of the 2022, 2023/24, and 2025 60 Decibels / Mastercard Czechia 
MSE Barometer reports. Apply these rules whenever writing, summarising, or presenting 
Barometer findings in any format.*

---

### What This Report Is

The **"Striving to Thrive"** Barometer is a longitudinal survey-based research series tracking 
the health, challenges, and aspirations of Czech micro and small enterprises (MSEs). Produced 
by 60 Decibels for Mastercard Strive in Czechia (a joint Mastercard / CARE Czech Republic 
programme). Each wave surveys ~600 MSEs in Czech, December–January.

It is NOT a corporate report, NOT an academic paper, and NOT a policy brief. It sits 
deliberately between all three — rigorous enough for policymakers, accessible enough for 
entrepreneurs, branded enough for a corporate funder. The best comparators are OECD or 
World Economic Forum impact research, but more human-centred and accessible.

---

### MSE Definitions — Use These Exactly, Every Time

| Term | Definition |
|---|---|
| Solopreneur / solo business | One person — no employees |
| Micro business | 1–9 employees (excluding owner) |
| Small business | 10–49 employees |
| MSE | Micro and small enterprise — the population this report studies |
| SME | Small and medium enterprise — explicitly NOT this report's population |
| MSME | Micro, small and medium enterprise — also NOT this population |

Always distinguish MSEs from SMEs and MSMEs. State explicitly that micro and small businesses 
face structurally different challenges from medium enterprises. The phrase "the small yet 
mighty" appears across all waves — use it.

---

### Target Audience

Write for all of these simultaneously. The Key Lessons section serves the time-poor reader 
who won't go further; the data chapters serve the analyst; the case studies serve the 
entrepreneur and the journalist.

- Policymakers and government ministries
- Business support organisations and NGOs
- Financial institutions
- Programme funders and corporate social impact teams
- Media and journalists
- Researchers and impact measurement practitioners
- Entrepreneurs themselves (particularly through case studies)
- Women's entrepreneurship advocates

**The implied reader** is educated and time-poor. Design the document so someone gets the 
essential message in five minutes from the lessons page, or can spend an hour in the full 
analysis. Both experiences are intentionally catered for.

---

### Fixed Report Structure

```
Cover
About the organisations (Mastercard Strive, 60 Decibels) — 2 short paragraphs each
"A note on MSEs" — terminology definition box, placed before the foreword
Foreword — from Mastercard/funder GM (reflective, proud, forward-looking)
Opening remarks — from CARE/partner (global context + explicit call to action)
Introduction
  — What goes into a Barometer
  — What to expect from this edition
  — KEY LESSONS SECTION  ← written last, placed HERE (before methods)
  — Our Methods
  — About the data / sample profile
  — Regional findings
Thematic chapters (5–6 sections, each with complete-statement subheadings)
Conclusion
What Next? (4 action themes, each addressed to a named actor)
Expert Comments (8–10 named external voices, presented verbatim)
Annex: Survey script
Thank you from 60 Decibels
```

**Critical structure rule:** The Key Lessons section is placed BEFORE the methodology and 
data detail. The time-poor reader gets the essential message without reading through methods.

---

### Key Lessons Section — Exact Format

The most important section. Written last, placed first after the introduction. Numbered 
(typically 9–11). Each lesson follows this exact pattern:

**Bold declarative headline sentence.** Supporting paragraph with "1 in X" framing, 
prior-wave comparison, and segmentation note by size/gender/region. No hedging. 
No passive voice.

Example:
> **Entrepreneurs are financially stable but have limited safety nets.** Many MSE owners 
> depend entirely on their business for income but lack the financial buffers needed to 
> weather economic shocks. In fact, 77% of MSE owners rely on their business as their 
> primary source of income. Despite this, 1 in 5 have no financial runway if their business 
> were to stop generating revenue — a trend consistent with the findings of the previous 
> Barometer in 2024.

---

### Data Chapter Paragraph Structure

Every body paragraph in a thematic chapter follows this sequence:

1. One-sentence scene-setter (context before data — never data first)
2. Key data point with "1 in X" framing
3. Longitudinal comparison ("This is the same proportion we found in last year's report...")
4. Segmentation cut by size, gender, or region
5. Implication or "so what" sentence

Never end a data paragraph without pointing somewhere — toward a recommendation, a related 
finding, or an open question.

---

### Tone of Voice: The Core Register

**Informed, accessible, purposeful.** Warm but not sentimental. Declarative not tentative. 
Always action-oriented.

**Write in first person plural throughout.** "We found," "we look to," "we believe," 
"we invite you." This creates the feeling of researchers talking to you, not an institution 
publishing at you. This is non-negotiable — without it the warmth collapses.

**Strategically honest, not strategically optimistic.** The report is NOT always optimistic. 
When data shows no improvement, say so directly. "There has been little change in the 
resilience of these MSEs." Stagnation is an accountability finding. Do not soften it to 
protect programme reputation. This intellectual honesty is what makes the report credible.

---

### What the Tone Is NOT

| Wrong register | Why it's wrong |
|---|---|
| Academic | No passive constructions, no hedging, no literature review style |
| Corporate PR | Difficult findings are stated plainly, not managed |
| Dry policy language | Warmth and human stories are present throughout |
| Journalistic | No hyperbole, no dramatic framing |
| Evangelical about technology | Barriers to adoption are reported neutrally |
| Consultant-speak | No "leverage," "synergies," "ecosystem enablement" |
| Charity copy | Empathy without sentimentality |

---

### Specific Language Patterns to Replicate

**"1 in X" construction** — use throughout the document, not just in headlines. Fractions 
land harder than percentages. Use alongside the percentage: 
"1 in 5 (18%) have no financial runway."

**"Interestingly"** — flags a finding that complicates or surprises. Use sparingly but 
consistently. Signals to the reader to pay extra attention.

**Data transition phrases — use these:**
- "Aside from this..." — moves to a related concern
- "Interestingly..." — counterintuitive or complicating finding
- "This data also aligns with broader trends..." — connects to global context
- "Despite these challenges..." — pivots toward resilience framing
- "These figures remain broadly unchanged from the previous Barometer, suggesting..." — 
  stagnation/accountability flag

**Global benchmark sentences** — contextualise Czech findings against World Bank, IFC, OECD, 
GEM data, particularly in opening remarks and introduction. Example: "This mirrors a broader 
global concern, where 50% of small businesses worldwide cannot survive more than three months 
without revenue."

**Subheadings** — complete statements or questions, never topic labels:
- ✓ "What's working and what entrepreneurs need"
- ✓ "Spotlight on cybersecurity"  
- ✗ "Digitisation"
- ✗ "Cybersecurity"

**Split colour heading treatment** (for Word/design output) — section titles and case study 
labels use two colours: "Case study:" in brand orange, remainder in black. 
"11 lessons" in orange, "from this Barometer" in black.

---

### Longitudinal Framing Rules

Every major finding must be framed as change, continuity, or stagnation — not just a 
point-in-time number. Always cite which wave.

**Templates:**
- "This represents a positive shift from [X]% in [year]..."
- "These figures remain broadly unchanged from the previous Barometer."
- "This trend has been consistent across all [N] waves of the Barometer."
- "Despite [N] years of programme activity, [metric] has not improved — indicating 
  structural rather than behavioural barriers."

---

### Gender Framing Rules — Non-Negotiable

1. Use **business leadership gender** (who runs the business) not respondent gender for 
   cross-tabs, unless specifically stated otherwise.

2. Gender gaps are always attributed to **structural and societal barriers** — sector 
   segregation, family responsibilities, cultural stereotypes, collateral requirements, 
   time poverty. Never attribute gaps to individual behaviour, risk appetite, or capability.

3. Always present the **gender-within-gender finding**: "59% of women agree men have an 
   advantage, compared to 37% of men." The perception gap within the gender gap is often 
   the most striking finding.

4. Women's challenges are always explained as structural, multi-dimensional, and not 
   reducible to financing alone.

---

### Recommendations Format

Always addressed to a named actor. Never passive or vague.

✓ "Financial institutions should develop products tailored to MSEs, incorporating lower 
   interest rates and flexible repayment terms."
✗ "Consideration should be given to improving access to finance."
✗ "It would be beneficial to explore options for MSE financing."

**Group under four categories:**
1. Enhancing financial awareness and business resilience
2. Reducing the barriers to digitalisation
3. Bridging the persistent gender gap
4. Fostering a more inclusive business environment

---

### Tone by Section

| Section | Register |
|---|---|
| Foreword (funder GM) | Reflective, proud, forward-looking. Warmer than body. |
| Opening remarks (CARE/partner) | Global context + explicit call to action. More urgent. |
| Introduction | Narrative, contextualising, inviting. Human stories before data. |
| Key Lessons | Punchy, declarative, numbered. Bold first sentence. No hedging. |
| Data chapters | Analytical, precise, comparative. Data with interpretation, never alone. |
| Case studies | Narrative, human, direct quotes. Third-person storytelling. |
| Expert comments | Each voice distinct and verbatim. No editorial framing added. |
| What Next | Directive. Named actor as subject of every sentence. |
| Thank you | Warm, humble, personal. |

---

### Case Studies

Two per report, embedded mid-report in thematic sections. Named entrepreneur (may be 
composite/fictional drawn from qualitative research). Third-person narrative with direct 
quotes in inverted commas. Always concludes with an explicit link back to the broader data 
finding it illustrates. Used to humanise specific analytical points — not as general colour.

---

### Expert Comments Section

Eight to ten short statements from named external stakeholders responding to the data. 
Placed near the end of the report. Voices should span: government officials, financial 
institutions, business support organisations, digital/tech sector, women's entrepreneurship 
advocates. Presented verbatim with name, title, organisation. No editorial paraphrasing. 
Gives the report the character of a conversation, not a monologue.

---

### Visual Design Reference

**Colour palette:** Deep orange/red as primary (chart bars, section numbers, callout boxes, 
highlighted words in headings). Dark charcoal for body text. Light grey/off-white for 
sidebar boxes.

**Chart types by question type:**

| Question type | Chart |
|---|---|
| Multi-option barriers / tools | Horizontal bar, % labels on bars |
| Likert scale (5-point) | 100% stacked bar, colour-coded by response |
| Simple binary / three-way split | Donut chart, large % in centre |
| Gender / size comparison | Side-by-side or diverging bars |
| Regional distribution | Choropleth map |

No scatter plots. No complex statistical charts. All charts readable without explanation.
Figures numbered sequentially with plain italic titles: "Figure 13: Ability to pay bills 
on time." No chart subtitles or embedded explanatory notes — context always in surrounding 
prose.

**Sidebar boxes:** Light grey background for methodology notes, glossary, pull-out insights. 
Rounded-corner shaded boxes for expert quotes, with speaker photo, name and title in brand 
orange.

**Photography:** Full-width candid images of entrepreneurs at work between sections. 
Never posed corporate photography.

---

### Podcast / Audio Adaptation Rules

- Use "1 in X" framing verbally — percentages are harder to absorb in audio
- Lead with the human story (case study character), then pull back to data
- Always name the wave and year
- Always note whether something has or hasn't changed since the previous wave
- Do not read sequences of percentages — paraphrase into narrative 
  ("nearly half," "most," "only one in ten")
- Sound conversational but expert — not influencer, not academic
- Maintain nuance: avoid flattening complex findings into simple good/bad frames

**Numbers to always express as fractions in audio:**

| Finding | Say this |
|---|---|
| ~18% no cash reserves | "1 in 5" |
| ~50% no support services | "1 in 2" |
| ~27–31% cyber incident | "nearly 1 in 3" |
| ~21% credit attempt | "only 1 in 5" |
| ~34% digitisation budget below 1% | "1 in 3" |

---

### Core Themes — Present Across All Waves

Financial resilience · access to finance · digitisation · technology adoption · 
cybersecurity · entrepreneur wellbeing · work-life balance · trust · ecosystem support · 
women entrepreneurship · inclusion · regional differences · business aspirations · 
resilience and adaptability
