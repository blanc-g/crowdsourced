# Crowdsourced genealogies

This release accompanies:

> Guillaume Blanc (2024), “Demographic transitions, rural flight, and intergenerational persistence: evidence from crowdsourced genealogies,” manuscript dated April 10, 2024.

## Release contents

- `data/europe.dta`: 9,426,965 observations and 119 variables, saved in Stata format 118 (Stata 14 or later).

The unit of observation is an anonymized individual profile. The file contains individuals for whom the mapped location of birth, first linked child's birth, or death is in Europe. It is not a random sample of the historical population; it is a crowdsourced genealogy sample whose representativeness is evaluated in the accompanying paper.

## European coverage and inclusion rule

The European sample covers Austria, Belgium, Bulgaria, Croatia, Czechia, Denmark, Estonia, Finland, France, Germany, Greece, Hungary, Iceland, Ireland, Italy, Latvia, Lithuania, Luxembourg, North Macedonia, the Netherlands, Norway, Poland, Portugal, Romania, Slovakia, Slovenia, Spain, Sweden, Switzerland, and the United Kingdom. In the string country variables, North Macedonia and the United Kingdom are recorded as `Macedonia` and `UK`.

Within the candidate-profile universe, an individual is included in `europe.dta` when at least one of three mapped events is in Europe: the individual's birth, the birth of the first linked child, or the individual's death. Birth in Europe is not required. Once an individual qualifies through a European event, every available event location is retained regardless of where that other event occurred. A person can therefore have been born outside Europe or later have died outside Europe and still appear in this file. Conversely, a profile with no mapped European event is not included. `europe==1` for every observation in this release.

The raw processing first limits the working universe using broad birth/death coordinate screens, so the three-event inclusion rule operates within that candidate universe. This intermediate screen is not a second population released with the data.

Non-European life events are not discarded for people selected into `europe.dta`. When such an event maps to the United States or Canada, its coordinates and place fields remain in the observation. `america` and the event-specific `*_america` variables describe those retained event locations; they do not define an additional North American sample. Historical urbanization and 1820 European population weights are generally unavailable for events outside Europe.

## Dataset construction

The data originate from the public FamiLinx genealogies assembled by Kaplanis et al. (2018) from public Geni.com profiles. The source is reported to contain 86,124,645 individual profiles plus direct parent-child links. The released data were constructed in four stages.

### 1. Profiles, dates, and plausibility screens

- Profiles are imported in 2,000,000-row blocks. The configured import reaches source row 86,000,000; see the corresponding limitation below.
- Sex, event years and months, and geographic coordinates are parsed for birth, death, baptism, and burial. Day-level fields, textual date descriptions, circa indicators, and original place-name fields are not retained.
- A missing or invalid month is set to January for constructing the Stata monthly date, while the reported `*_m` field itself is left missing or invalid. Missing birth components are subsequently filled from baptism components, and missing death components from burial components.
- A profile is removed if any nonmissing birth, death, baptism, or burial year is earlier than 1400 or is 2018 or later. Duplicate profile identifiers are reduced to the first imported record.
- When the necessary monthly dates are available, the individual is removed if lifespan is below 15 or above 125 years, first-child age is below 15, first-child age exceeds 65 for a male or 45 for a female, or the first child is born after the individual's death. The sex-specific upper limit is not applied when sex is missing.

### 2. Family links and fertility

`fert` is the number of retained parent-child link rows for an individual; it is an observed linked-child count, not a verified count of all biological children. Parent-child links for a parent with more than 50 rows are discarded rather than top-coded. If more than two parents are linked to a child, only the first two after numeric-ID ordering enter the parent and co-parent structure. A one-parent record receives a synthetic missing-co-parent placeholder during construction; the released co-parent identifier is missing.

The variables named `spouse`, `couple_id`, and `n_spouses` are constructed from shared children and should be interpreted as co-parent measures, not verified marriages:

- A candidate couple is a pair of recorded co-parents of at least one linked child.
- `couple_fert` is the number of linked children shared by that exact pair.
- `spouse` is the co-parent in the pair sharing the greatest number of linked children with the individual; an internal record-order tie-break is used when pairs tie.
- `spouse_fert` is that selected co-parent's total retained linked-child count.
- `n_spouses` counts candidate co-parent-pair records and can include a missing-co-parent pair. If it exceeds 10, the individual's derived fertility, co-parent, and child-summary fields are not retained, although the individual profile can remain in the main file.

First- and last-child years, monthly dates, months, and locations are derived from linked children whose profiles could be matched. Relation counts can therefore be observed even when dated child profiles are not. The fertility sample (`s_fert==1`) additionally requires the individual's own fertility to be observed and at least one of the 30 pedigree positions in the four preceding generations to have more than one linked child. The 30 positions are 2 parents, 4 grandparents, 8 great-grandparents, and 16 great-great-grandparents. Pedigree collapse can cause the same ancestor to occupy more than one position.

### 3. Geographic crosswalk and historical urban status

European coordinate keys are matched using 2018 LAU geography and 2013 NUTS 3, NUTS 2, and country geography. For non-European events retained with a qualifying individual, the available mapping covers 2019 US Census counties and states and 2016 Canadian census divisions and provinces. These modern units are applied to historical event coordinates. The released `*_dep`, `*_region`, and `*_country` values are dataset-specific integer categories with Stata value labels generated from area names; they are not official geographic codes. For purposes of matching historical city populations, selected region-level consolidations are applied for London, Brussels, Antwerp, Liège, Dublin, and Belfast. A Copenhagen/Hovedstaden rule is also attempted through text matching and may not match the source label.

The broad European groups in `*_group` are:

- `France`: France.
- `British_Isles`: Ireland and the United Kingdom.
- `Northern_Europe`: Denmark, Estonia, Finland, Iceland, Latvia, Lithuania, Norway, and Sweden.
- `Southern_Europe`: Croatia, Greece, Italy, Macedonia, Portugal, Slovenia, and Spain.
- `Eastern_Europe`: Bulgaria, Czechia, Poland, Romania, and Slovakia.
- `Central_Low_Countries`: Austria, Belgium, Germany, Hungary, Luxembourg, the Netherlands, and Switzerland.

Historical urban status is based on Bairoch, Batou, and Chèvre (1988) municipality populations for 1500, 1600, 1700, 1750, 1800, and 1850. Population values are expressed in thousands, and a mapped place is urban when population is at least 5 (5,000 inhabitants). Selected missing intermediate populations are geometrically interpolated between surrounding benchmarks. Once a place is observed as urban, a missing status at a later benchmark is carried forward as urban; other missing statuses are coded zero in covered countries. Helsinki is manually added as an urban place in 1850. Urban fields are set missing for Estonia, Iceland, Latvia, Lithuania, and Macedonia, as well as for events outside Europe.

For all three mapped events, the applicable urban benchmark is selected using the individual's year of birth. Thus `first_child_urban` describes the first-child location at the parent's birth-year benchmark, and `death_urban` describes the death location at the deceased person's birth-year benchmark. These are the measures used in the paper to classify origin and destination places relative to urban status at the individual's birth.

### 4. Derived variables, parents, and weights

Great-circle birth-to-death distance is generated in kilometres. Ages at first child, last child, and death are differences between Stata monthly dates divided by 12.

Parent 1 and parent 2 are ordered by numeric identifier, not by sex. Parent identifiers and linked-child counts can be present even when the parent's own profile is absent from the final individual file. Parent sex, event dates, mapped places, ages, and distance are populated only when that linked parent is also found in the retained intermediate individual sample.

Event weights use estimated country population in 1820 from the Maddison Project Database 2020. Missing 1820 values are manually supplied for Croatia, Estonia, Iceland, Latvia, Lithuania, Luxembourg, Macedonia, Slovakia, and Slovenia, using the assumptions recorded in the construction. For each European country and event, the weight equals 1820 population in thousands divided by the number of genealogy observations in the event-specific denominator window: `[1675,1900)` for birth, `[1700,1925)` for first child, and `[1750,1975)` for death. Weights are not normalized to a mean of one. The same country-event factor is attached to observations outside its denominator window, so it should normally be used only with the corresponding recommended window.

## Principal variables

Variable names are case-sensitive.

### Individual and sample variables

- `id`: anonymized FamiLinx individual identifier.
- `male`: male indicator (1=male, 0=female).
- `europe`: indicates that at least one selection event maps to Europe. It is one for every observation in this file.
- `america`: auxiliary indicator equal to one when at least one retained event maps to the United States or Canada; it does not determine inclusion in this European file.
- `s_main`: main-sample indicator. It is one for every observation in this file.
- `s_fert`: fertility-sample indicator. The European fertility sample contains 759,824 individuals with `s_fert==1`.

### Fertility and family links

- `fert`: observed number of retained linked-child rows. Fertility analyses should use observations with `s_fert==1`.
- `spouse`: identifier of the selected co-parent with whom the individual shares the most linked children.
- `spouse_fert`: observed number of retained linked-child rows of the selected co-parent.
- `couple_id`: identifier for the individual-selected-co-parent pair.
- `couple_fert`: number of linked children shared by that pair.
- `n_spouses`: number of candidate co-parent-pair records before the main co-parent is selected. Values over 10 cause the fertility-family fields, but not necessarily the profile, to be excluded.
- `fert_parents`: mean observed fertility of recorded parents.
- `s_fert_parents`: equals one when at least one linked parent is itself present in the retained individual sample with `s_fert==1`; it is missing otherwise.

The fertility sample retains an individual when fertility is observed and at least one of the 30 pedigree positions in the four preceding generations is recorded with more than one linked child. Childless individuals cannot be identified reliably. The measure is completed gross linked fertility; child mortality is under-recorded outside direct vertical lineages.

### Parent variables

Parent 1 and parent 2 are ordered by numeric FamiLinx identifier, not by sex. Use `parent1_male` and `parent2_male`; do not interpret parent 1 as the father or parent 2 as the mother.

For each parent, the file can include:

- Identifier, sex, and observed fertility: `parent#_id`, `parent#_male`, and `parent#_fert`.
- Age at first child, last child, and death: `parent#_first_child_age`, `parent#_last_child_age`, and `parent#_death_age`.
- Birth-to-death distance: `parent#_distance`.
- Year, monthly date, place identifier, raw coordinate key, country, and Europe indicator for birth, first linked child's birth, and death. These profile characteristics require the parent to be present in the retained intermediate individual sample.

### Dates and ages

For event prefix `birth`, `first_child`, `last_child`, or `death`:

- `*_y`: event year.
- `*_m`: reported month or a baptism/burial fallback month, where applicable. It can be missing or contain an invalid source code.
- `*_date`: Stata monthly date, formatted `%tm`. January is substituted when the month available at initial date construction is missing or invalid. Later baptism/burial substitutions do not reliably update this field.
- `first_child_age`, `last_child_age`, and `death_age`: monthly-date differences divided by 12.

The same date-construction caveats apply to parent event fields. The first- and last-child year, month, and date are calculated separately and can refer to different linked children. `last_child_m` is the minimum reported month among children in the latest observed child-birth year, rather than the maximum. When the chronologically first child's location is missing, the earliest dated linked child with a nonmissing location can be substituted. A first-child location therefore need not correspond exactly to `first_child_date`.

### Geography

For event prefix `birth`, `first_child`, or `death`:

- `*_latlon_ID`: raw latitude-longitude key constructed from FamiLinx coordinates.
- `*_id`: mapped LAU identifier in Europe or source coordinate-key identifier in the US/Canada.
- `*_dep`: dataset-specific integer category with value labels for NUTS 3 names in Europe, county names in the US, or census-division names in Canada; not an official code.
- `*_region`: dataset-specific integer category with value labels for NUTS 2 names in Europe, state names in the US, or province names in Canada; not an official code.
- `*_country`: dataset-specific integer country category with Stata value labels; not an official country code.
- `*_COUNTRY`: the same country information as a string.
- `*_group`: broad European country group; missing outside Europe.
- `*_latitude`, `*_longitude`: coordinates of the mapped place. European values are LAU representative points rather than the original FamiLinx coordinates.
- `*_europe`, `*_america`: event-location continent indicators.

No other last-child geography fields are included.

### Urban status, migration, and weights

- `birth_urban`, `first_child_urban`, and `death_urban` indicate that the corresponding municipality had at least 5,000 inhabitants at the latest available Bairoch benchmark at or before the individual's year of birth, where historical urban data are covered. `death_urban` therefore describes the death place's urban status at the individual's birth benchmark, not at the year of death.
- `distance` is the great-circle distance in kilometres between mapped birth and death locations, generated with Stata's `geodist` command.
- `birth_weight`, `first_child_weight`, and `death_weight` are event-country reweighting factors for the European source countries. The numerator is the country's estimated population in 1820, measured in thousands. The denominator is the number of genealogical observations for that event and country in the corresponding half-open main window: `[1675,1900)` for birth, `[1700,1925)` for first child, and `[1750,1975)` for death. They are generally missing for events outside Europe and are not normalized to a mean of one.

Use the weight corresponding to the event-time outcome being analyzed.

## Recommended analysis windows

The paper focuses on periods where comparisons with representative sources suggest selection is most limited:

- Urbanization and migration by year of birth: 1675-1900.
- Fertility by year of birth of first child: 1700-1925.
- Longevity by year of death: 1750-1975.

The file retains observations outside these windows for supplementary use.

## Complete 119-variable codebook

The dataset has the following schema.

| # | Variable | Definition |
|---:|---|---|
| 1 | `id` | Anonymized FamiLinx individual identifier. |
| 2 | `male` | Male indicator: 1=male, 0=female; missing if source sex was not identified. |
| 3 | `distance` | Great-circle distance in kilometres between mapped birth and death locations. |
| 4 | `death_age` | Age at death in years, calculated from monthly dates and divided by 12. |
| 5 | `europe` | 1 if birth, first-child, or death location maps to Europe; otherwise missing. |
| 6 | `america` | Auxiliary indicator equal to 1 if any retained birth, first-child, or death location maps to the US or Canada; missing otherwise. It does not determine inclusion in `europe.dta`. |
| 7 | `fert` | Number of retained parent-child link rows; use `s_fert==1` for fertility analysis. |
| 8 | `s_fert` | Fertility-sample indicator; 1 for the analytic fertility sample, 0 or missing otherwise. |
| 9 | `s_main` | Main-sample indicator; equals 1 for every observation in this file. |
| 10 | `spouse` | Identifier of the selected co-parent sharing the most linked children with the individual; not a verified spouse. |
| 11 | `spouse_fert` | Retained linked-child count of the selected co-parent. |
| 12 | `couple_id` | Identifier for the individual-selected-co-parent pair. |
| 13 | `couple_fert` | Number of linked children shared by the individual and selected co-parent. |
| 14 | `n_spouses` | Number of candidate co-parent-pair records before selection; fertility-family fields are absent above 10. |
| 15 | `fert_parents` | Mean observed fertility of recorded parents. |
| 16 | `s_fert_parents` | 1 if at least one linked parent is itself retained with `s_fert==1`; missing otherwise. |
| 17 | `parent1_id` | Identifier of parent 1; parent order is not sex-specific. |
| 18 | `parent1_fert` | Observed number of linked children of parent 1. |
| 19 | `parent1_male` | Male indicator for parent 1; parent order is not sex-specific. |
| 20 | `parent1_first_child_age` | Age of parent 1 at birth of first linked child. |
| 21 | `parent1_last_child_age` | Age of parent 1 at birth of last linked child. |
| 22 | `parent1_death_age` | Age at death of parent 1, in years. |
| 23 | `parent1_distance` | Great-circle distance in kilometres between parent 1 birth and death locations. |
| 24 | `parent1_birth_y` | Year of parent 1 birth. |
| 25 | `parent1_birth_date` | Stata monthly date of parent 1 birth. |
| 26 | `parent1_birth_id` | Mapped LAU/place identifier for parent 1 birth. |
| 27 | `parent1_birth_latlon_ID` | Raw FamiLinx latitude-longitude key for parent 1 birth. |
| 28 | `parent1_birth_COUNTRY` | String country name for parent 1 birth. |
| 29 | `parent1_first_child_y` | Year of parent 1 first linked child's birth. |
| 30 | `parent1_first_child_date` | Stata monthly date of parent 1 first linked child's birth. |
| 31 | `parent1_first_child_id` | Mapped LAU/place identifier for parent 1 first linked child's birth. |
| 32 | `parent1_first_child_latlon_ID` | Raw FamiLinx latitude-longitude key for parent 1 first linked child's birth. |
| 33 | `parent1_first_child_COUNTRY` | String country name for parent 1 first linked child's birth. |
| 34 | `parent1_death_y` | Year of parent 1 death. |
| 35 | `parent1_death_date` | Stata monthly date of parent 1 death. |
| 36 | `parent1_death_id` | Mapped LAU/place identifier for parent 1 death. |
| 37 | `parent1_death_latlon_ID` | Raw FamiLinx latitude-longitude key for parent 1 death. |
| 38 | `parent1_death_COUNTRY` | String country name for parent 1 death. |
| 39 | `parent2_id` | Identifier of parent 2; parent order is not sex-specific. |
| 40 | `parent2_fert` | Observed number of linked children of parent 2. |
| 41 | `parent2_male` | Male indicator for parent 2; parent order is not sex-specific. |
| 42 | `parent2_first_child_age` | Age of parent 2 at birth of first linked child. |
| 43 | `parent2_last_child_age` | Age of parent 2 at birth of last linked child. |
| 44 | `parent2_death_age` | Age at death of parent 2, in years. |
| 45 | `parent2_distance` | Great-circle distance in kilometres between parent 2 birth and death locations. |
| 46 | `parent2_birth_y` | Year of parent 2 birth. |
| 47 | `parent2_birth_date` | Stata monthly date of parent 2 birth. |
| 48 | `parent2_birth_id` | Mapped LAU/place identifier for parent 2 birth. |
| 49 | `parent2_birth_latlon_ID` | Raw FamiLinx latitude-longitude key for parent 2 birth. |
| 50 | `parent2_birth_COUNTRY` | String country name for parent 2 birth. |
| 51 | `parent2_first_child_y` | Year of parent 2 first linked child's birth. |
| 52 | `parent2_first_child_date` | Stata monthly date of parent 2 first linked child's birth. |
| 53 | `parent2_first_child_id` | Mapped LAU/place identifier for parent 2 first linked child's birth. |
| 54 | `parent2_first_child_latlon_ID` | Raw FamiLinx latitude-longitude key for parent 2 first linked child's birth. |
| 55 | `parent2_first_child_COUNTRY` | String country name for parent 2 first linked child's birth. |
| 56 | `parent2_death_y` | Year of parent 2 death. |
| 57 | `parent2_death_date` | Stata monthly date of parent 2 death. |
| 58 | `parent2_death_id` | Mapped LAU/place identifier for parent 2 death. |
| 59 | `parent2_death_latlon_ID` | Raw FamiLinx latitude-longitude key for parent 2 death. |
| 60 | `parent2_death_COUNTRY` | String country name for parent 2 death. |
| 61 | `birth_y` | Reported birth year, with baptism year substituted when birth year is missing. |
| 62 | `birth_m` | Reported birth month, with baptism month substituted when birth month is missing; may remain missing or invalid. |
| 63 | `birth_date` | Stata monthly date constructed before baptism fallback; January is used when the original birth month is missing or invalid, and the value can disagree with `birth_y` or `birth_m`. |
| 64 | `birth_id` | 2018 LAU identifier in Europe or source coordinate-key identifier in the US/Canada for the birth place. |
| 65 | `birth_dep` | Dataset-specific value-labelled category for the birth-place NUTS 3, county, or census-division name; not an official code. |
| 66 | `birth_region` | Dataset-specific value-labelled category for the birth-place NUTS 2, state, or province name; not an official code. |
| 67 | `birth_country` | Dataset-specific numeric country category with value labels for the birth location. |
| 68 | `birth_COUNTRY` | String country name for individual's birth location. |
| 69 | `birth_group` | Broad European country-group name for individual's birth location; missing outside Europe. |
| 70 | `birth_europe` | 1 if individual's birth location maps to Europe; missing otherwise. |
| 71 | `birth_america` | Birth-location indicator: 1=US/Canada, 0=Europe, missing if unmapped. |
| 72 | `birth_longitude` | Longitude of mapped birth place; a representative LAU point in Europe. |
| 73 | `birth_latitude` | Latitude of mapped birth place; a representative LAU point in Europe. |
| 74 | `birth_latlon_ID` | Raw FamiLinx latitude-longitude key for individual's birth. |
| 75 | `birth_weight` | 1820 European population in thousands divided by the country's birth-record count in `[1675,1900)`; generally missing outside Europe. |
| 76 | `birth_urban` | Urban status of the mapped birth place at the latest benchmark at or before the individual's birth year; missing where urban data are not covered. |
| 77 | `first_child_y` | Minimum nonmissing birth year among matched linked-child profiles. |
| 78 | `first_child_m` | Minimum reported or fallback month among children in the earliest observed child-birth year; may be missing or invalid and need not match `first_child_date`. |
| 79 | `first_child_date` | Minimum available Stata monthly birth date among matched linked-child profiles; it can refer to a different child from `first_child_y` or `first_child_m`. |
| 80 | `first_child_age` | Age at birth of first linked child, in years. |
| 81 | `first_child_id` | 2018 LAU identifier in Europe or source coordinate-key identifier in the US/Canada for the mapped first-child place; a fallback may use the earliest dated child with a location. |
| 82 | `first_child_dep` | Dataset-specific value-labelled category for the first-child-place NUTS 3, county, or census-division name; not an official code. |
| 83 | `first_child_region` | Dataset-specific value-labelled category for the first-child-place NUTS 2, state, or province name; not an official code. |
| 84 | `first_child_country` | Dataset-specific numeric country category with value labels for the first-child location. |
| 85 | `first_child_COUNTRY` | String country name for birth of first linked child location. |
| 86 | `first_child_group` | Broad European country-group name for birth of first linked child location; missing outside Europe. |
| 87 | `first_child_europe` | 1 if birth of first linked child location maps to Europe; missing otherwise. |
| 88 | `first_child_america` | First-child-location indicator: 1=US/Canada, 0=Europe, missing if unmapped. |
| 89 | `first_child_longitude` | Longitude of mapped first-child place; a representative LAU point in Europe. |
| 90 | `first_child_latitude` | Latitude of mapped first-child place; a representative LAU point in Europe. |
| 91 | `first_child_latlon_ID` | Raw FamiLinx latitude-longitude key for birth of first linked child. |
| 92 | `first_child_weight` | 1820 European population in thousands divided by the country's first-child-record count in `[1700,1925)`; generally missing outside Europe. |
| 93 | `first_child_urban` | Urban status of the mapped first-child place at the latest benchmark at or before the parent's birth year; missing where urban data are not covered. |
| 94 | `last_child_y` | Maximum nonmissing birth year among matched linked-child profiles. |
| 95 | `last_child_m` | Minimum reported or fallback month among children in the latest observed child-birth year; may be missing or invalid and need not match `last_child_date`. |
| 96 | `last_child_date` | Maximum available Stata monthly birth date among matched linked-child profiles; it can refer to a different child from `last_child_y` or `last_child_m`. |
| 97 | `last_child_age` | Age at birth of last linked child, in years. |
| 98 | `death_y` | Reported death year, with burial year substituted when death year is missing. |
| 99 | `death_m` | Reported death month, with burial month substituted when death month is missing; may remain missing or invalid. |
| 100 | `death_date` | Stata monthly date constructed before burial fallback; January is used when the original death month is missing or invalid, and the value can disagree with `death_y` or `death_m`. |
| 101 | `death_id` | 2018 LAU identifier in Europe or source coordinate-key identifier in the US/Canada for the death place. |
| 102 | `death_dep` | Dataset-specific value-labelled category for the death-place NUTS 3, county, or census-division name; not an official code. |
| 103 | `death_region` | Dataset-specific value-labelled category for the death-place NUTS 2, state, or province name; not an official code. |
| 104 | `death_country` | Dataset-specific numeric country category with value labels for the death location. |
| 105 | `death_COUNTRY` | String country name for individual's death location. |
| 106 | `death_group` | Broad European country-group name for individual's death location; missing outside Europe. |
| 107 | `death_europe` | 1 if individual's death location maps to Europe; missing otherwise. |
| 108 | `death_america` | Death-location indicator: 1=US/Canada, 0=Europe, missing if unmapped. |
| 109 | `death_longitude` | Longitude of mapped death place; a representative LAU point in Europe. |
| 110 | `death_latitude` | Latitude of mapped death place; a representative LAU point in Europe. |
| 111 | `death_latlon_ID` | Raw FamiLinx latitude-longitude key for individual's death. |
| 112 | `death_weight` | 1820 European population in thousands divided by the country's death-record count in `[1750,1975)`; generally missing outside Europe. |
| 113 | `death_urban` | Urban status of the mapped death place at the latest benchmark at or before the individual's birth year; missing where urban data are not covered. |
| 114 | `parent1_birth_europe` | 1 if parent 1 birth location maps to Europe; missing otherwise. |
| 115 | `parent1_first_child_europe` | 1 if parent 1 first linked child's birth location maps to Europe; missing otherwise. |
| 116 | `parent1_death_europe` | 1 if parent 1 death location maps to Europe; missing otherwise. |
| 117 | `parent2_birth_europe` | 1 if parent 2 birth location maps to Europe; missing otherwise. |
| 118 | `parent2_first_child_europe` | 1 if parent 2 first linked child's birth location maps to Europe; missing otherwise. |
| 119 | `parent2_death_europe` | 1 if parent 2 death location maps to Europe; missing otherwise. |

## Papers to cite

- Blanc, Guillaume. 2024. “Demographic transitions, rural flight, and intergenerational persistence: evidence from crowdsourced genealogies.” Manuscript dated April 10, 2024.
- Kaplanis, Joanna, et al. 2018. “Quantitative Analysis of Population-Scale Family Trees With Millions of Relatives.” *Science* 360(6385): 171-175.

## Reuse

No explicit data-license file accompanies this release. Users remain responsible for the terms attached to the upstream sources and should contact the author regarding redistribution or other uses requiring a license determination.
