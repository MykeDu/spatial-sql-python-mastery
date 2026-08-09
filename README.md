# Spatial SQL & Python Mastery
This repository documents my journey through the Spatial SQL & Python Mastery plan, focused on building real fluency in modern GIS workflows using SQL (DuckDB, PostGIS) and Python (GeoPandas, Rasterio).

## About Me
Hi, I’m Mike—a GIS analyst focused on mastering code-first, modern spatial workflows. My goal is to become fluent in Spatial SQL and Python for spatial analysis, and to build a public portfolio that shows my progress and real-world skills. I’m committed to daily, hands-on practice and sharing what I learn along the way.
- I’ve worked in GIS for 15+ years, mostly in the ESRI stack, and I’m now making the move to code.
- I want to work on open data projects and contribute to the modern GIS community.
- I believe in learning by doing, not just reading theory.

📅 Structure
- Daily folders for each practice session (SQL and Python)
- Capstone projects at 30, 60, 90, and 180 days
- Resources and reference materials
  
🚀 Goals
- Write working code for real spatial analysis tasks
- Build a public portfolio of SQL queries, Python scripts, and AI prompt patterns
- Practice prompt engineering with AI to accelerate learning
- Master the modern GIS stack: DuckDB, PostGIS, SedonaDB, GeoPandas, Rasterio

💾 Data Path
Note: On Windows, I store large or raw data in a short path to avoid issues
- Remember to use c:/tmp when long data path cause a problem
- use environment variables for the data path
```
# config.py
DATA_PATH = r"C:/tmp"

# main script
import os
from config import DATA_PATH
parcels = gpd.read_file(os.path.join(DATA_PATH, 'parcels.geojson'))
```

🗂️ Folder Structure

![](/_imgs/flderStruc.png)

- Each lesson, dayXX_* folder matches your daily plan: keep lesson notes, code, and prompt patterns together.
- Use the capstones folder for your 30/60/90/180-day portfolio projects.
- The resources folder is for cheat sheets, links, and anything you want to reference quickly.

🗂️ Folder Guide
day01_sql_intro/
 lesson_notes.md
 practice_query.sql
 ai_prompt_examples.md

 day02_python_gdf/ lesson_notes.md first_script.py ai_prompt_examples.md
capstones/ day30_portfolio_project/ project_overview.md code/ data/
resources/ reference_links.md cheat_sheets/

📝 How I’m Using This Repo
- Each day, I add my code, notes, and prompt patterns to the matching folder
- Capstone projects are documented in their own subfolders
- I push updates after each session to track progress and build my portfolio

## Daily Logs
All daily progress logs are in the /daily folder, named by date (e.g., 2026-08-09.md). Each log includes:
- A summary of what I practiced
- Key code snippets
- Reflections and next steps
- Notes on AI prompt patterns and troubleshooting

## Learning Philosophy
- Implementation over theory: I learn by writing real code, not just reading docs.
- AI as a coding partner: I use AI to help write, debug, and explain code.
- Portfolio-driven: Every day’s work is visible and builds toward real-world projects.
- Consistency: Daily practice is the foundation of fluency.

🔗 Useful References
- [GeoPandas Documentation](https://geopandas.org/en/stable/)
- [DuckDB Documentation](https://duckdb.org/docs/current/)
- [PostGIS Documentation](https://postgis.net/docs/)- 