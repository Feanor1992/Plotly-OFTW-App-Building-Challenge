# Plotly-OFTW-App-Building-Challenge

## 1. Import Necessary Libraries ##

This section imports all required libraries:
- `os` and `time` for system operations and delay handling.
- `pandas` for data manipulation and `plotly.express` for creating interactive visualizations.
- `datetime` for managing date and time objects.
- `Dash` components (from Dash, dcc, html, Input, Output, State, callback, no_update) for building the interactive dashboard.
- `dash_ag_grid` for rendering interactive data tables.
- `google.generativeai` for integrating with Google’s Generative AI API used to generate textual insights.

## 2. HELPER FUNCTIONS ##

Several helper functions are defined to support common tasks:
- `safe_to_datetime(series)`: Converts a Pandas Series containing date strings into datetime objects, returning NaT on errors.
- `create_empty_figure(title)`: Creates an empty Plotly figure with a specified title and applies a dark template.
- `create_kpi_box(text, box_id)`: Generates a styled Dash HTML Div to display a KPI value, with a specific ID used for later callback updates.

## 3. INITIALIZE DASH APP ##
The Dash application is instantiated using:

`app = Dash(__name__)`

This sets up the app for further layout configuration and interactivity.

## 4. DATA LOADING, CLEANING, AND PREPROCESSING ##

Data is loaded from two remote JSON sources:
- **Pledges data** is loaded from a given URL into a DataFrame (`df_pledges`).
- **Payments data** is loaded similarly into `df_payments`.

Both DataFrames have duplicates removed. In the payments DataFrame, rows with missing `pledge_id` values are dropped. Date columns (e.g., `pledge_date` and `payment_date`) are converted into datetime objects using the `safe_to_datetim`e helper function.

## 5. LOADING METADATA FILE ##

A metadata file is retrieved from a Google Sheets URL (exported as CSV) into a DataFrame (`df_metadata`). This file may provide supplemental information for the dashboard and is later shown in a table.

## 6. DATA MERGING AND FEATURE ENGINEERING ##

Pledges and payments are merged on the `pledge_id` column using an **outer join** to ensure all records are preserved.
- The code then renames columns to remove default suffixes (for example, `donor_id_x` becomes `donor_id` and `payment_platform_y` is renamed to `payment_platform_payment`), standardizing the names across datasets.
- A new feature, `payment_delay`, is computed as the difference (in days) between `payment_date` and `pledge_date`.
- Additionally, the year and month are extracted from the payment date for later use.

## 7. AG GRID SETUP ##

Two interactive AG Grid tables are set up:
- One grid displays the merged dataset using a selected set of display columns (e.g., `pledge_id`, `pledge_created_at`, `date`, `payment_platform`, `amount`, `payment_delay`, etc.).
- Another grid displays the metadata from the CSV file.

The display columns are controlled by a list (`final_display_columns`).

## 8. COMPUTE OFTW METRICS ##
This function calculates key metrics from the data:
- It computes the total money moved (summing the payment amounts) and a counterfactual metric.
- For pledges, metrics like Active ARR (Annualized Run Rate) are calculated from contribution amounts.
- Pledge attrition rate, counts of active donors and active pledges, and chapter-level run rates are also derived.
The function returns a dictionary containing these metrics.

## 9. UPDATED REAL LLM INSIGHTS FUNCTION USING NEW GENERATIVE MODEL API ##

This function integrates with **Google’s Generative AI**:
- It uses a generative model (e.g., `gemini-1.5-flash`) to process the metrics (provided in **Markdown**) and generate actionable insights.
- A status message (“Analyzing…”) is prepended to the output.

It includes a retry mechanism with exponential backoff, returning a fallback message if all attempts fail.

## 10. DASH APP LAYOUT ##

The layout of the dashboard is defined using Dash components:
- A header Markdown component describes the app.
- The layout includes filter components: a dropdown for selecting payment platforms and a date picker for the payment date range.
- KPI boxes (created via the `create_kpi_box` function) display key performance metrics.
- A Markdown component shows the overall metrics summary.
- A button triggers the LLM insights generation (with its output displayed in a loading component).
- Various Plotly graphs (histogram, time series, and pie chart) and the AG Grid tables for merged data and metadata are displayed.

## 11. CALLBACK FOR UPDATING VISUALIZATIONS, KPI, AND DATA TABLE ##

This callback function updates the graphs, KPI boxes, and data table based on user inputs (selected platforms and date range):
- The `filter_data` function is used to produce a filtered DataFrame.
- Visualizations (histogram, time series, pie chart) are updated using Plotly based on the filtered data.
- KPIs are recalculated (including handling for cases with fewer than five records, appending a warning message).
- The updated table data is then passed to the AG Grid.

## 12. CALLBACK FOR UPDATING OFTW METRICS SUMMARY ##

This callback recomputes the overall metrics (ignoring filters) whenever filter inputs change.
- It calls the `compute_of_tw_metrics` function, then formats the resulting metrics dictionary into a Markdown string using `format_metrics_as_markdown`.
- The Markdown output is then displayed on the dashboard.

## 13. CALLBACK FOR REAL LLM INSIGHTS USING NEW GENERATIVE MODEL API ##

Triggered when the “Get LLM Insights” button is clicked, this callback:
- Uses State to read the current metrics summary without triggering on every update.
- Calls the `get_llm_insights` function with the metrics summary, which returns generated actionable insights.
- The generated text is then displayed in the dashboard.

## 14. RUN THE APP ##

Finally, the script checks if it is run as the main module and starts the Dash development server
