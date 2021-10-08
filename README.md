# DEEvalSports
take-home for data engineering role

## outputs
### 1. Domain Model

[Domain Model Capture pdf](DEEval%20-%20domain-model.pdf)

Initially started with syntactic approach, identifying the key people / places / things and modelling narratives of use. After building some understanding, most narratives were player centric as this offered comparison between match and training performance. Moved to ideating dashboards that provided insights into the test narratives utilising identified classes and attributes.

Simplified model to reduce workload. Model retains a superclass of person with specialisations for each type of person. The database implementation could have been improved, simplified to a variable of role.

Normalised out some of the variables into their own classes, e.g. events and descriptions to reduce repetition in inner tables.

Removed the need to track external; players, coaches, managers, trainers and reduced representation to variable opposition in Matches table for simplicity. This creates an issue in the PlayByPlay class where events should be tracked against both for and opposing players, however, was not needed for final dashboard.

### 2. Google Sheets

[Google Sheets Link](https://docs.google.com/spreadsheets/d/1W9SHwY958y-o6pl07wOqoSKuBvKbGLxs6s0m_6CdmDA/edit?usp=sharing)

Some variables should be calculations or aggregations of other tables, e.g. score and play by play, google sheets syntax was tricky, did not want to be importing pivot tables into bigquery. Should have completed aggregations in bigquery after import, but at the time just filled in the synthetic data.

Used ID's as keys to link across tables

### 3. Download & 4. Storage

[Process Notebook](exercise.ipynb)

Process is documented in notebook, highlights:
- able to use google sheets URL as API to download sheets using book and sheet ID's
  - this could have been used as in google cloud function to move directly from gcs to big query, however, this would likely be a cron job to at least check for changes then push on change.
- Used service key to auth local session to experiment with different methods to Google SDK, store in secrets file (not in repo)
- Function activated on create or update in allocated bucket e.g. when a new upload to bucket occurs reducing unnecessary runs and/or manual activation
- Modified requested process from ETL to ELT, extract -> add column -> load, would require memory allocation increases as the tables/files increased in size, pushing the data to big query then transforming into new table with timestamp meant that cloud function instance only pointed bigquery to correct file and location
  - alternative could be streaming the data from source files as updated, unsure of implementation
- Used autodetect schema so one function could serve all tables/files, this would allow unknown columns and data or changes in data schema causing issues downstream, could correct onsite or in biquery depending on case. As here, would likely load to ingest table with undefined schema and handle transforms and checks inside bigquery to push to table with defined schema.
- More robust workflow should append new records rather than replacing tables, however when testing tables and dashboard where changes were frequent replacing tables reduced the need to manually modify the table schema or manually remove table each change.
- Joined tables into populated tables (delivery) to use in PowerBI due to simplicity of SQL vs DAX for the transformations.

### 5. Dashboard

[Dashboard Capture pdf](deeval-dashboard-capture.pdf)

- Chose PowerBi as data links between Data Studio and Bigquery were a bit tedious, need to explore Bigquery BI engine.
- Split into Match and Player dashboards for different usecases, a detailed view of each match as this is likely usecase for match data, and then a over time view of each player's progression
- play timeline is a bit hacky using a bar chart, but served the purpose without needing custom shape formula
- visualisations are not quite aligned due to small data size, should have extracted some actuals from kaggle to test.
- stacked bar chart was not the best representation for the match performance data, however again, due to the small number of records, more appropriate plots did not display correctly
- could improve training performance by either adding units or transforming into %
- created ability to drill through to player page
