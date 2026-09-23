Life Insurance Agent Performance Dashboard
Project Overview
This Power BI dashboard tracks how well our life insurance agents are performing—looking closely at their sales (APE), target hits, and how well they retain policies over time. I built it using a solid data model, some custom visuals (like Deneb and HTML Content) to get exactly the right look, and DAX to handle the messy real-world data I found in the raw files.


1. Data Model & Architecture
The Foundation
I set up a standard star schema. The dim_date, dim_agent, and dim_product tables filter the core fact tables (sales, targets, and persistency). This keeps the report fast and makes cross-filtering across pages work predictably.

Fixing the "Missing Agent" Problem
When I dug into the data, I noticed about 5% of the sales records had an agent_id that didn't exist in the agent dimension table. If I left this alone, filtering the dashboard would silently drop those sales from the grand totals.


•	How I fixed it: I pulled those orphaned agent_ids directly from the sales table, created a "Missing agents" table in Power Query, and appended them back into the main agent table. This ensures our revenue totals are always 100% accurate, and it saves those original IDs so we can investigate the data entry issue later.

Handling Multiple Dates
The persistency table has two dates: when the policy was issued, and when it's due for renewal. I linked the calendar directly to the issue date so we can easily group policies into cohorts on Page 4. For anything needing the renewal date, I handled it inside the specific DAX measures using USERELATIONSHIP.


2. DAX Highlights
Active Policy Snapshot
To get a true snapshot of active policies, I couldn't just sum things up over time. Instead, the [Active Policy Count] measure looks at the end of whatever period is selected, then counts the policies that were issued before that date and haven't lapsed yet.

Dynamic HTML Profile Cards
For the Agent Profile Card, I wrote a DAX measure that calculates the agent's KPIs (YTD APE, Rank, MoM Trend) and wraps them in HTML/CSS. I also threw in CONCATENATEX to safely handle situations where two agents have the exact same name. This feeds into the HTML visual to create a clean, responsive card that updates instantly when you drill through.

Security (RLS)
I set up a Territory Manager role using [territory] = USERPRINCIPALNAME(). This means when an agency leader logs in, the whole dashboard automatically filters down to show only their specific region and agents.


3. Visuals & Layout
•	Deneb (Vega-Lite): Native Power BI visuals can get clunky with huge data grids, so I used Deneb to build a custom Actual vs. Target performance grid. It color-codes the performance (Green/Orange/Red) so you can immediately see who is on track.

•	Cohort Matrix: I used a standard matrix paired with a calculated [Years Since Issue] column to build a classic actuarial triangle. It groups policies by the year they were sold and tracks how well they renew as time goes on.

4. What I'd Do With More Time
If I were deploying this into a full production environment, here are a few practical things I'd improve:


1.	Clean up the backend: I'd move all the DAX calculations into a dedicated, standalone measures table and group them into folders (e.g., "Sales", "Persistency"). It just makes life easier for the next developer who touches the file.

2.	Push transformations upstream: Right now, I'm handling the missing agent cleanup inside Power Query. Ideally, I'd want to push that logic back to the SQL database or Data Warehouse so the Power BI model stays as light and fast as possible.

3.	Add interactive toggles: I'd love to add a toggle switch using Power BI bookmarks so users could flip between Month-to-Date and Year-to-Date views on the same page, without needing to navigate to a completely different tab.

