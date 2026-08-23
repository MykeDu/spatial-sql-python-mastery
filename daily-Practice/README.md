# Daily Practice Workflow

Each day in this folder documents one focused Spatial SQL or Python practice session. The goal is to move from reading code, to explaining the spatial logic, to writing and verifying a working solution.

## Daily folder structure

Day-XX/
└── MMDD/
 ├── 1_overview.txt # Daily objective and the spatial pattern being practiced
 ├── 2_lesson.txt # Micro-lesson notes and key functions
 ├── 3_practice.txt # Practice problem, attempts, results, and debugging notes
 ├── 3a_read.txt # Read the code and identify what each part does
 ├── 3b_explain.txt # Explain the query in plain English
 ├── 3c_write.txt # Write or adapt the solution independently
 ├── 4_PromptDrill.txt # AI-pairing prompt and notes on the response
 └── day-xx-...log.md # Polished daily practice 

## How I use this workflow

1. Overview: Define the spatial problem and the target pattern.
2. Lesson: Capture the function, syntax, and decision points.
3. Practice: Work through the implementation with small, testable data.
4. Read: Identify tables, geometry columns, predicates, joins, and output fields.
5. Explain: Translate the code into a plain-English spatial process.
6. Write: Rebuild or adapt the query without copying it line for line.
7. Prompt drill: Use AI to draft, debug, or explain code, then verify the result.
8. Daily log: Publish the useful pattern, code, and reflection as a concise Markdown entry.

## Practice standard

I treat AI as a coding partner, not an answer generator. The work is complete when I can explain the spatial logic, verify the CRS and units, run the code, and inspect the output.

The daily log is the portfolio-facing summary. The supporting files preserve the implementation trail: the lesson, the reasoning, the prompt, and the working notes behind it.