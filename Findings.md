### CLEANING 

1. Before 2007 the field  Aircraft category was not being filled.
There is a 99% missing value rate between 1983 and 2007 while from 2008 onwards this drops to almost 0%
Dataset would hence be skewed towards 2008-2023.

2. Handled Injury columns by dropping 54 rows that had missing values all the columns and filled in 0 for the remaining columns with null values. Dropped rows that also recorded 0 passengers 

3. Dropped missing and unknown values for Aircraft damage which constituted to about 6% of the total rows.
As for the Is.Destroyed column the airplanes that are Destroyed are marked as binary 1 and the Substantial and Minor as binary 0

4. Cleaning tasks for 'Make'
Standardize casing and strip whitespace (.upper().strip())
Remove punctuation (periods/commas) that create near-duplicates (CORP. vs CORP)
Strip common corporate suffixes (INC, CORP, CORPORATION, CO, COMPANY, LLC) so entity-name variants of the same manufacturer collapse together
Fix a few known spelling/spacing aliases (DEHAVILLAND → DE HAVILLAND)
Drop the 1 row with Make NaN (negligible)
Apply the ≥50-record threshold for the final analysis set

5. Model cleaning. Drop all the Null values, labels are not unique and have to be paired with Make to define a plane type

6. For the rest of the  relevant columns 
EngineType and WeatherCondition UNK replaced with unknown to consolidate the two. 
Rows with null or 0.0 values for number of engines immediately dropped.
Purpose of flight formatted to consolidate Air Race Show and Air Race/show

7. Dropping columns that are irrelevant to the study or that have large number of missing values. NB kept the Broad.phase.of.flight despite it having 85% missing columns because it is required as one of the metrics for analysis

### ANALYSIS

1. 