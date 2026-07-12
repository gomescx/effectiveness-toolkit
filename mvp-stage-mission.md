
## BAckground
The application in this repo has been running as a POC for quite a while, and it's serving the original purpose to check if running a digital version of these manual paper based tools would provide any benefit. The application was deployed in GitHub with the original intention of not requiring any storage. 

When I created this repo for the first time over a year ago, I intended to use it as an added tool for creating leads for future selling of my services as a coach after I have trained people on the 5 PEP tools of the suite. 

## What has been implemented
Since then I've been using this  mostly for myself and have found some benefits within each tool and their functionalities, which are now listed below. 

1) Memory Map action plan
Main benefit: transform a memory map into an action plan which otherwise would be a lengthy manual process error prone. This is typically the main reason why people end up not implementing memory maps: they have to translate them into a list, which they must follow up on themselves or use to lead others, and the time and effort required is just too much. 

What is:
 The memory map is where a leader dump their heads and sequences out what they will do and how long it will take.  The memory map differs from a regular mind map because it contains the elapsed time and the effort time for each node and allows sequencing of nodes, that is independent from the node hierarchy, e.g. if we have nodes A.1-3 and B.1-3 in a memory map we can define the sequence of execution is for instance A1, B2, B3, A3, A2, B1. Each one of these notes can have a different effort related to them and can also take a different elapsed time. For instance, node A1 might require two hours of effort and take one day to be concluded Waiting for another person, which means node B2 can start only one day later, even though the effort expended is just two hours.  
 To make this actionable, the memory map needs to be translated into a table, sequenced in this case with six rows—A 1, B 2, B 3, A 3, A 2, B 1—one per row, including the elapsed time and other characteristics. This table is called the action plan, which users can then print out and take for their meetings and project management. 

 this implementation has used mind-elixir (originally scoped as JSMind) with some field customisation To allow for the effort and duration. 

2) Clarity of End result (COER)
Main Benefit: Integrated with the tool so I don't have to chase where I left the COER PDF and Word template. Saves as a simple JSON instead of a bloated file. One of the reasons why people don't use the COER It's because they never know where the file template is sitting, and then they don't know where they have saved it. Having a link on a browser and then opening a form is an easy way to overcome that issue, and it also helps integrate with the other tools of the sewage when we save it as a JSON. 

What is:  a very simple form that scopes out the boundaries of any initiative. 

3) Time Management Matrix (TMM)
Benefit: Helps prioritize different initiatives or different actions inside a memory map or action plan when these actions can be worked in parallel

What is: Replicates Steve Covey's time management matrix - IMPACT vs URGENCY Allowing people to plot their initiatives in various different locations. An extension of my implementation is that the person can give a relative size to the initiative, which then makes the dot become a larger circle. 


## Not implemented 

4. Strength of Belief 

Main benefit: Allows obstacles from the COER To have positive action associated with them. This reduces procrastination, as people start to become more confident in tackling things that apparently are very difficult because they don't have control over them. 

What is: The strength of belief is a circle of concern, a circle of control, and a relationship from the work of Martin Seligman.Three concentric circles are drawn - the inner most is the circle of control, the middle one is the circle of influence plotted in dotted line and the outer circle is the circle of concern. we start this exercise plotting the different obstacles identified on the COER between the circles of control and concern. If the feeling of control of a certain obstacle is very low, the obstacle gets plotted closer to the circle of concern and vice versa. If the obstacle is fully under control, it gets as close as possible to the center of the inner circle. .   By having a visual understanding of them, the person working with this tool starts  to figure out what actions they can take to expand their influence And hopefully start encompassing more obstacles that currently are not under control. 

The outcome of this exercise is a list of candidate actions that give more confidence to the person involved in an initiative. This list of candidate actions will feed back into the memory action plan as high priority work. 


5. Impact Map 
Main benefit: create a list of candidates for action to reduce the impact of issues that cannot be solved.

What is: A mindmap that starts with the issue itself and gets expanded into the following questions:
- What are the situations impacting the initiative or the work? 
- What are the consequences of each situatio and is it positive or negative to the initiative or work? 
- What can be done to maximize the positive consequence or minimize the negative consequence? 

Typically, when this map is created visually, the person ends up realizing that one of the actions that creates positive consequences also affects other situations; As well as some of the actions that minimize negative consequences also affect other situations. The more situations a candidate action affects, the higher the benefit it creates, and therefore they become candidates for action. 

## What Has not worked 

- Typically, when I create a memory map to outline an action plan, the issue of priority arises, and I need to run the time‑management matrix to prioritize things. This means I have to re‑enter all the data into another tool and then bring it back to the memory map to sequence it.

Some initiatives that initially appear on the memory map are very large and need to be decomposed into smaller steps. This again requires both an action plan and an assessment of priority via the time‑management matrix. The resulting document may scope a number of actions or correspond one‑to‑one with each node of the memory map, depending on the size. For example, if I have five initiatives on a memory map that are plotted into the time‑management matrix and then returned to the memory map, and two of them need to be decomposed into another ten or fifteen nodes, I end up with many detailed outcomes for the larger initiatives.

Very quickly, I can have dozens of unrelated files that I need to name specifically to Make sure that they relate to each other. 

## What I need

I need an integrated tool that allows me to start a memory map, then select one of the nodes of the memory map and create a clarity of the end result for it. Then continue detailing the memory map, select a node and its children, turn it into an action plan, import the obstacles into the strength of belief, and eventually export some of them into the impact map. Finally, there should be an option to write everything into a single document markdown file I can then take it as my guide for the things I need to do and be able to return, knowing exactly the name of the initiative, to update it on a regular basis. 

The application should be able to ingest any action plan in its specific format, export it back into a memory map for further ideation

## out of scope for the MVP

- multi-user

## Mandatory
###  Review of the whole technology stack
 to ensure longevity, maintainability. e.g. JSMind may not be as sustainable as other tools - it was chosen due to the intention of keeping the app serverless - this is no longer mandatory

###  Do not reinvent the wheel 

 There's a lot of custom written code in this suite that is not needed. For instance, the repo has a whole mind map customization in JSMind whenthere are loads of mindmap applications that could be adapted or extended with custom fields to allow the sequencing, Relapsed time and invested time aka effort.
 I would love to be able to use MindNode, either the classic version or MindNode Next, or any other mind‑map application such as iThoughts that we could extend from. I have an active SetApp macos subscription which offers both of these mindmaps apps in the package.

 Other solutions could exist for resolving the Strength of Beliefs and TMM needs if we add extensions to them. I expect you to research this heavily and come with options for me. 

 I want to minimize, as much as I can, the code I create in this application. Less code created, less maintenance. 

## PO decisions — MVP review (2026-07-12)

Captured during the mission review Q&A. These bound the MVP and become exit-criteria inputs for the stage mission.

1. **Users**: Single user (Claudio) on his own Mac. No demos, no pilot coachees during MVP.
2. **Platform**: macOS-only is acceptable. Printing the action plan / markdown guide covers meetings.
3. **Deployment/storage**: A small self-hosted backend is acceptable (Docker on the Mac, or a cheap VPS). Serverless is no longer a constraint.
4. **Build vs buy**: Composition of commercial apps is welcome — SetApp apps or generous free tiers — provided the move between tools is seamless (temporary file storage in the handoff is fine). MindNode as the ideation editor is the ideal. It is accepted that no single app covers all 5 tools.
5. **Markdown guide**: Living document, two-way. Ingestion may be rigid: an LLM can be used to reformat manual edits into the strict ingestible format.
6. **Round-trip fidelity**: Lossless. The memory map and the action plan are two views of the same data — hierarchy, sequence, effort/elapsed, and cross-tool links must survive map → plan → map.
7. **MVP exit criterion**: After ~a month of real use, this is the tool reached for by default and the "dozens of unrelated files" problem is gone. No fixed initiative count; no demo-readiness bar.
8. **Repo strategy**: Fresh repository for the MVP; this repo is archived read-only (POC stays deployed on GitHub Pages as a working fallback). Surviving modules (TMM view, COER form, export logic) are ported selectively if the tech-stack review keeps them.
9. **Tech direction (decided 2026-07-12)**: **Option B — Obsidian as the hub.** The canonical store is an Obsidian vault: one folder per initiative, markdown as the native living document (decision 5 solved by construction), Canvas/Excalidraw plugins hosting TMM and Strength of Belief, a mind-map plugin (or MindNode one-way import for first-draft ideation) for the memory map, and Dataview-style queries rendering the action plan. Custom code is limited to converters/templates and at most one small plugin. Successor repo: `effectiveness-obsidian-toolkit`.

### Facts established during review (research, 2026-07-12)

- **iThoughts is dead**: toketaWare ceased trading in February 2024. Remove it from consideration.
- **MindNode Next is healthy**: actively developed (2026.3, May 2026), exports Markdown/OPML/FreeMind, and its new Shortcuts architecture exposes actions that interact with **documents, nodes and tags** — not just whole-file import/export. This is the closest thing to an extension surface it has; validating its fidelity is the first MVP spike.
- **MindNode has no plugin API and no per-node custom fields**: effort/elapsed/sequence would have to ride on tags or naming conventions and live canonically in our own store.
- **mind-elixir is actively maintained** (v5.10.0, July 2026) — the current POC stack is not a burning platform; any migration is by choice, not necessity.
- The POC implementation uses **mind-elixir, not JSMind** (doc corrected above).

 




