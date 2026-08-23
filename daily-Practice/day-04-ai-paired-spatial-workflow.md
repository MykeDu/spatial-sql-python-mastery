## Day 4 – August 10, 2026 – Daily Practice Log
### Focus:
- Spatial join mechanics (INNER JOIN vs LEFT JOIN)
- Prompt engineering for spatial SQL queries

### Activities Completed:
- Micro-lesson: Explored the difference between INNER JOIN and LEFT JOIN in spatial SQL
- Practice problem: Wrote a spatial join query
- Prompt drill: Practiced AI-generated SQL for spatial joins

### Key Takeaways:
- INNER JOIN returns only matching records between spatial tables, while LEFT JOIN keeps all records from the left table, even if there’s no spatial match.
- When prompting AI for spatial SQL, clearly specify the spatial relationship and the expected output.

### Sample Prompt Used:
"Write a PostGIS query to join a table of schools to a table of parks, returning all schools and the name of the nearest park (if any). Both tables use EPSG:4326."

### Sample Query: 
```

SELECT s.id, s.name, p.name AS nearest_park
FROM schools s
LEFT JOIN parks p
 ON ST_DWithin(s.geom::geography, p.geom::geography, 2000)
ORDER BY s.id;

```

### Reflection:
- Practicing both the SQL and the prompt-writing side-by-side helped clarify the logic and the syntax.
- Next, I want to experiment with different spatial relationships (e.g., ST_Intersects, ST_Contains) and see how the join type affects the results.
