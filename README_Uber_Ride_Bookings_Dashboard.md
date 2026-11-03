# Uber Ride Bookings Analysis Dashboard

## 1. Project overview

This project analyses NCR ride-booking data in **Microsoft Power BI**. The work covers the complete BI workflow: importing the CSV source, preparing the data in **Power Query Editor**, creating a model-level measure in **Data view / Model view**, and building a multi-page interactive dashboard.

The report is designed to help a viewer understand booking performance, vehicle usage, revenue-related patterns, cancellations, and customer/driver ratings. It is an analytical report based on the supplied dataset; it is not a live connection to Uber's operational systems.

## 2. Dataset details

| Item | Details |
|---|---|
| Source file | `ncr_ride_bookings.csv` |
| Rows | 150,000 |
| Columns | 21 |
| Date coverage | 1 January 2024 – 30 December 2024 |
| Main record | A ride-booking record |
| BI tool | Microsoft Power BI Desktop |
| Data preparation | Power Query Editor |
| Calculations | DAX measure(s) in the semantic model |

### Fields in the source

| Field | Meaning |
|---|---|
| Date, Time | Date and time associated with the booking |
| Booking ID | Unique booking reference |
| Booking Status | Outcome/status of the booking |
| Customer ID | Customer reference |
| Vehicle Type | Type of vehicle selected |
| Pickup Location, Drop Location | Start and destination locations |
| Avg VTAT | Average vehicle time to arrive |
| Avg CTAT | Average customer time to arrive |
| Cancelled Rides by Customer | Customer-cancellation indicator/count field |
| Reason for cancelling by Customer | Reason recorded for customer cancellation |
| Cancelled Rides by Driver | Driver-cancellation indicator/count field |
| Driver Cancellation Reason | Reason recorded for driver cancellation |
| Incomplete Rides | Incomplete-ride indicator/count field |
| Incomplete Rides Reason | Reason recorded for an incomplete ride |
| Booking Value | Recorded booking amount |
| Ride Distance | Recorded ride distance |
| Driver Ratings, Customer Rating | Rating fields |
| Payment Method | Payment method recorded for the booking |

The source contains blank values in several fields. Many of these are expected because cancellation reasons, incomplete-ride reasons, ratings, payment information, and trip metrics may only be available for particular booking outcomes. A blank should not automatically be treated as zero.

## 3. End-to-end workflow

### Step 1 — Import the CSV file

1. Open **Power BI Desktop**.
2. Select **Home → Get data → Text/CSV**.
3. Browse to `ncr_ride_bookings.csv`.
4. Preview the data and select **Transform Data** rather than loading it immediately.
5. Power BI opens the Power Query Editor, where the source can be checked and prepared before it enters the data model.

**Why this matters:** importing the source into Power Query first makes it possible to correct data types, inspect missing values, standardise fields, and avoid building visuals on top of inconsistent data.

### Step 2 — Clean and prepare the data in Power Query Editor

Power Query is the ETL/preparation layer of this project:

- **Extract:** connect to the CSV file.
- **Transform:** clean and shape the rows and columns.
- **Load:** apply the query changes and load the prepared table into the Power BI model.

Recommended checks and transformations for this dataset:

1. **Promote and verify headers.** Confirm that the first row is being interpreted as the column header row and that all 21 field names are correct.
2. **Set data types.** Set `Date` to Date, `Time` to Time, ID and descriptive fields to Text, and numeric fields such as `Booking Value`, `Ride Distance`, `Avg VTAT`, `Avg CTAT`, and ratings to suitable numeric types. Correct types are essential for date filtering, aggregation, and calculations.
3. **Check identifiers.** Keep `Booking ID` and `Customer ID` as text. IDs are labels, not quantities; summing or averaging them would be meaningless. Check for leading/trailing spaces and inconsistent formatting.
4. **Clean text columns.** Use **Transform → Format → Trim** and, where appropriate, **Clean** on text columns. This helps prevent values that look identical from being treated as separate categories because of invisible spaces or characters.
5. **Review missing values.** Inspect blanks in cancellation reasons, incomplete-ride reasons, ratings, payment method, booking value, and ride distance. Preserve meaningful blanks where a field is not applicable. Replace a blank with zero only when the business definition says that zero is the correct value.
6. **Check category consistency.** Review `Booking Status`, `Vehicle Type`, `Payment Method`, and reason columns for spelling or case variations. Standardise only genuine duplicates; do not merge categories that have different meanings.
7. **Validate numeric ranges.** Check for negative or implausible booking values, distances, waiting/travel times, and ratings. Investigate unexpected values rather than deleting them automatically.
8. **Check duplicate records.** Use `Booking ID` to investigate potential duplicates. Remove a row only when it is confirmed to be an accidental duplicate, not merely because two rides share similar values.
9. **Rename the query clearly.** For example, use `ncr_ride_bookings` as the model table name, consistent with the report.
10. **Apply changes.** Select **Home → Close & Apply** to load the transformed table into the model.

> Documentation note: the supplied PBIX confirms that the report uses the `ncr_ride_bookings` table and the supplied CSV has 150,000 rows and 21 columns. The exact sequence of individual Power Query steps is stored inside the PBIX query definition; this README describes the cleaning checks and transformations that should be documented, rather than claiming that every suggested check was necessarily applied.

### Step 3 — Data modelling and DAX measures

After loading the prepared table:

1. Open **Data view** or **Model view**.
2. Select the `ncr_ride_bookings` table.
3. Choose **Table tools / Modeling → New measure**.
4. Enter the DAX expression in the formula bar, give the measure a meaningful name, and press Enter.
5. Set the measure's format (for example, percentage, whole number, decimal, or currency) to match the result.
6. Use the measure in a Card or other visual where a calculated KPI is required.

The supplied report contains a model measure named **`CanceledPercentage`**, used on the Cancellation page. The report also uses aggregations such as booking counts and sums of booking value, ride distance, and rating fields. Those aggregations appear in the visual definitions and may be implicit visual aggregations rather than separate named measures.

**Important:** the precise DAX formula for `CanceledPercentage` should be copied from the measure's formula bar in Power BI before publishing this README as final technical documentation. The measure name alone does not establish whether its denominator is all bookings, completed plus cancelled bookings, or another defined population. Document the numerator, denominator, and treatment of `No Driver Found` and `Incomplete` records explicitly so that the percentage has an unambiguous business meaning.

Example of the general DAX pattern (illustrative only — use the actual business definition in your PBIX):

```DAX
Cancellation % =
DIVIDE(
    [Cancelled Bookings],
    [Total Bookings],
    0
)
```

`DIVIDE` is generally preferable to `/` because it lets you specify a result when the denominator is zero. Do not replace the existing `CanceledPercentage` formula with this example unless it matches your intended definition.

### Step 4 — Build the report pages

The supplied PBIX contains six report pages. The first is a navigation/landing page, followed by five analytical pages.

#### 4.1 Home Page

The landing page is the entry point to the report. It uses navigation/action buttons to direct the viewer to the analysis pages. This gives the report a dashboard-style experience instead of requiring the viewer to search through report tabs.

**How to use it:** select the relevant navigation button to open the corresponding analysis section.

#### 4.2 Overall

This page provides a high-level view of booking activity.

- **Bookings over time:** a line chart uses the booking count against the Date hierarchy. It supports a time-based view of booking volume, helping reveal peaks, dips, and changes across the reporting period.
- **Booking status distribution:** a pie chart groups bookings by `Booking Status` and displays the count for each status. This helps compare completed, cancelled, no-driver-found, and incomplete outcomes.
- **Date slicer:** filters the page to a selected date or date range.
- **KPI card:** provides a booking identifier-based card in the supplied report. Confirm the card's intended label and aggregation in Power BI; an ID minimum is not a total-bookings KPI.

**Questions this page can help answer:** How does booking volume change over time? What share of records falls into each booking status? Does the status mix change when a date range is selected?

#### 4.3 Vehicle Type

This page focuses on how the different vehicle categories contribute to booking value and ride distance.

- The report contains repeated KPI cards based on **Sum of Booking Value**, arranged for vehicle-category analysis.
- It also contains repeated KPI cards based on **Sum of Ride Distance**.
- A Date slicer provides time filtering.
- The page layout is intended to make vehicle categories easy to compare.

**How to interpret it:** compare booking value and distance across vehicle types. A high booking value does not necessarily mean a high number of rides; value, ride count, and distance are different measures. Check each card's category label and filter context before drawing a conclusion.

#### 4.4 Revenue

This page explores recorded booking value and its relationship with payment method and ride distance.

- **Booking value by payment method:** a column chart uses the sum of `Booking Value`, grouped by `Payment Method`. This compares the total recorded value associated with each payment category.
- **Ride distance over time:** a column chart uses the Date hierarchy and summed `Ride Distance`, allowing the viewer to examine distance patterns across the date hierarchy.
- **Customer booking value table:** a table groups `Customer ID` with the sum of `Booking Value`, making it possible to inspect customer-level recorded value.
- **Date slicer:** filters the revenue analysis to a selected period.

**Interpretation note:** the visual shows the sum of the `Booking Value` field, not necessarily recognised accounting revenue. The source contains missing booking values, so totals should be interpreted in the context of which records have a value. A customer-level sum is not the same as customer profitability or lifetime value.

#### 4.5 Cancellation

This page examines cancellation patterns and reasons.

- **Customer cancellation reasons:** a pie chart combines `Reason for cancelling by Customer` with the sum of `Cancelled Rides by Customer`.
- **Driver cancellation reasons:** a second pie chart combines `Driver Cancellation Reason` with the sum of `Cancelled Rides by Driver`.
- **Cancellation KPI cards:** cards provide cancellation-related summary indicators, including the `CanceledPercentage` measure.
- **Date slicer:** allows cancellation analysis for a selected period.

**Questions this page can help answer:** Which customer cancellation reasons occur most often? Which driver cancellation reasons are most common? Does the cancellation percentage change over time?

**Interpretation note:** cancellation reason fields are blank for many records because a reason is not applicable to every booking. Do not interpret blank reasons as a specific cancellation cause. Confirm the definition of `CanceledPercentage` in its DAX formula before quoting it in a presentation.

#### 4.6 Ratings

This page examines customer and driver rating fields across the vehicle categories.

- The page contains cards based on **Sum of Customer Rating** and **Sum of Driver Ratings**, organised for vehicle-type comparison.
- The cards make it possible to compare the rating totals shown for the different vehicle categories.

**Important analytical caution:** a sum of ratings is strongly affected by how many rating records exist in a category. For service-quality comparisons, an **average rating** is usually more interpretable than a sum, and the number of ratings should be shown alongside it. Also account for blanks: not every booking has a customer or driver rating. If the intention is to show average experience, consider measures such as `AVERAGE(ncr_ride_bookings[Customer Rating])` and `AVERAGE(ncr_ride_bookings[Driver Ratings])`, with suitable labels and blank handling.

## 5. Dataset snapshot and validation

The supplied CSV contains 150,000 rows, covering 1 January through 30 December 2024. The raw booking-status distribution is:

| Booking status | Rows | Share of all rows |
|---|---:|---:|
| Completed | 93,000 | 62.0% |
| Cancelled by Driver | 27,000 | 18.0% |
| No Driver Found | 10,500 | 7.0% |
| Cancelled by Customer | 10,500 | 7.0% |
| Incomplete | 9,000 | 6.0% |
| **Total** | **150,000** | **100%** |

Vehicle categories in the source are Auto, Go Mini, Go Sedan, Bike, Premier Sedan, eBike, and Uber XL.

These are source-data counts, not necessarily the values currently displayed by every report visual. Page slicers, cross-filtering, visual-level filters, and report interactions can change the displayed totals.

## 6. Interactivity and report usage

1. Open the Home Page.
2. Navigate to the required analysis page using the page buttons.
3. Use the Date slicer to select a date or date range.
4. Select a chart segment, bar, or category to cross-filter or highlight related visuals where interactions are enabled.
5. Clear selections or reset slicers to return to the broader view.
6. Hover over data points to inspect tooltip values.
7. Compare related metrics carefully: booking count, booking value, ride distance, cancellation count, and rating are not interchangeable.

## 7. Key insights the dashboard is designed to support

- Booking status analysis can show the balance between completed and unsuccessful bookings.
- A booking trend can reveal when demand is higher or lower during the year.
- Vehicle-category comparisons can highlight which categories contribute more recorded booking value or ride distance.
- Payment-method analysis can show how recorded booking value is distributed across payment types.
- Cancellation-reason visuals can help identify recurring operational or customer-side issues.
- Rating analysis can support service-quality monitoring, provided averages and rating counts are used for fair comparisons.

The dashboard is an exploratory analysis tool. It can identify patterns and questions for further investigation, but it does not by itself prove why a pattern occurred.

## 8. Data quality and modelling recommendations

For a stronger production-ready version:

- Create a dedicated Date table, mark it as the Date table, and relate it to the booking table. Use the Date table for consistent time intelligence.
- Confirm the relationship cardinality and filter direction if additional dimension tables are introduced.
- Create explicit measures for Total Bookings, Completed Bookings, Customer Cancellations, Driver Cancellations, Incomplete Bookings, No Driver Found, Total Booking Value, Total Ride Distance, and Average Ratings.
- Define cancellation-rate logic in writing and use the same definition throughout the report.
- Use average ratings rather than rating sums when the goal is to compare service quality.
- Display rating counts and the number of records contributing to each KPI.
- Format Booking Value as currency only after confirming the currency used by the source. Do not assume a currency symbol without validation.
- Add clear visual titles, units, date context, and explanatory tooltips.
- Test slicers and cross-highlighting on every page.
- Reconcile important totals against the source data after applying Power Query transformations.

## 9. Tools and technologies

- **Microsoft Power BI Desktop:** report creation, data modelling, DAX calculations, and visualisation.
- **Power Query Editor:** data import, cleaning, transformation, and loading.
- **DAX (Data Analysis Expressions):** measures and model calculations.
- **CSV:** source data format.

## 10. Conclusion

The Uber Ride Bookings Analysis Dashboard turns a 150,000-row booking dataset into a multi-page Power BI report. Power Query provides the data-preparation stage, the semantic model supports calculations such as `CanceledPercentage`, and the report pages present booking trends, vehicle-type comparisons, revenue-related patterns, cancellation reasons, and rating information. Together, these stages demonstrate an end-to-end business intelligence workflow: prepare reliable data, define meaningful calculations, and communicate the results through interactive visuals.

---

## Before publishing this README

Please verify these details in Power BI Desktop so the documentation exactly matches the final PBIX:

1. Copy the exact DAX expression for `CanceledPercentage` from the formula bar and add it to Section 3.
2. Confirm the exact Power Query **Applied Steps** list and replace/extend the generic cleaning checklist in Section 2 with the steps you actually performed.
3. Confirm the currency and distance units, if they are known.
4. Check whether the rating cards should display sums or averages; label them accurately.
5. Check the card on the Overall page that currently references the minimum Booking ID, and confirm its intended meaning and displayed title.
