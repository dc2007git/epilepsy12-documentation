---
title: Reporting
reviewers: Dr Simon Chapman
---

Reporting is the principle reason for the Epilepsy12 Audit - to provide patients, their families and the teams caring for them meaningful information about the standard of their care. A secondary aim of the audit is to provide clinicians and hospital manager teams with real time information, filtered to their organisation, on their performance against the indicators, as well as other descriptors of their clinic - for example demographic information and case mix. Whilst it is not a clinical tool, it is meant to be used and updated in a clinical setting, and allow multidisciplinary teams to view the dashboard together at intervals and reflect on their service.

## Filtering

To be able to count KPIs, children from a subset of organisations first need to be filtered. The number of organisations in each group depends on what geographical region is being summarised. The degree of focus is referred to in the Epilepsy12 platform as a 'level of abstraction'. From smallest to largest these are organisation, trust/local health board, NHS region, OPEN UK network, country and national (England and Wales). The filtering functions are found in ```epilepsy12/common_view_functions/report_queries.py```.

### Active/registered cases in a given organisation expressed at different levels of abstraction

```python
def all_registered_cases_for_cohort_and_abstraction_level(
    organisation_instance,
    cohort,
    case_complete=True,
    abstraction_level: Literal[
        "organisation", "trust", "icb", "nhs_region", "open_uk", "country", "national"
    ] = "organisation",
):
    """
    Returns all Cases that have been registered for a given cohort at a given abstration level.
    It can return cases that are only registered in E12 for a given cohort but have not yet completed the return, or
    cases that are both registered and have also completed all required fields in the return.
    Parameters accepted:
    Organisation instance
    Cohort - this is an integer
    case_complete: a boolean flag denoting that the case is registered in Epilepsy12 and a fully completed audit return is available
    abstraction level: string, one of ['organisation', 'trust', 'icb', 'nhs_region', 'open_uk', 'country', 'national']

    Note that all these children will by definition have epilepsy, since a cohort cannot be allocated without the clinician confirming
    1. the child has epilepsy
    2. the first paediatric assessment falls within the the dates for that cohort
    3. a lead E12 site has been allocated
    """

    if case_complete:
        all_cases_for_cohort = Case.objects.filter(
            Q(registration__isnull=False)
            & Q(registration__audit_progress__registration_complete=True)
            & Q(registration__audit_progress__first_paediatric_assessment_complete=True)
            & Q(registration__audit_progress__assessment_complete=True)
            & Q(registration__audit_progress__epilepsy_context_complete=True)
            & Q(registration__audit_progress__multiaxial_diagnosis_complete=True)
            & Q(registration__audit_progress__investigations_complete=True)
            & Q(registration__audit_progress__management_complete=True)
            & Q(registration__cohort=cohort)
        ).all()
    else:
        all_cases_for_cohort = Case.objects.filter(
            Q(registration__isnull=False) & Q(registration__cohort=cohort)
        ).all()

    if abstraction_level == "organisation":
        q_filter = (
            Q(site__organisation__pk=organisation_instance.pk)
            & Q(site__site_is_actively_involved_in_epilepsy_care=True)
            & Q(site__site_is_primary_centre_of_epilepsy_care=True)
        )
    elif abstraction_level == "trust":
        q_filter = (
            Q(
                site__organisation__ParentOrganisation_ODSCode=organisation_instance.ParentOrganisation_ODSCode
            )
            & Q(site__site_is_actively_involved_in_epilepsy_care=True)
            & Q(site__site_is_primary_centre_of_epilepsy_care=True)
        )
    elif abstraction_level == "icb":
        if organisation_instance.integrated_care_board is not None:
            """
            Wales has no ICBs
            """
            q_filter = (
                Q(
                    site__organisation__integrated_care_board__ODS_ICB_Code=organisation_instance.integrated_care_board.ODS_ICB_Code
                )
                & Q(site__site_is_actively_involved_in_epilepsy_care=True)
                & Q(site__site_is_primary_centre_of_epilepsy_care=True)
            )
        else:
            return all_cases_for_cohort
    elif abstraction_level == "nhs_region":
        """
        Wales has no NHS regions
        """
        if organisation_instance.nhs_region is not None:
            q_filter = (
                Q(
                    site__organisation__nhs_region__NHS_Region_Code=organisation_instance.nhs_region.NHS_Region_Code
                )
                & Q(
                    site__organisation__ons_region__ons_country__Country_ONS_Name=organisation_instance.ons_region.ons_country.Country_ONS_Name
                )
                & Q(site__site_is_actively_involved_in_epilepsy_care=True)
                & Q(site__site_is_primary_centre_of_epilepsy_care=True)
            )
        else:
            return all_cases_for_cohort
    elif abstraction_level == "open_uk":
        q_filter = (
            Q(
                site__organisation__openuk_network__OPEN_UK_Network_Code=organisation_instance.openuk_network.OPEN_UK_Network_Code
            )
            & Q(site__site_is_actively_involved_in_epilepsy_care=True)
            & Q(site__site_is_primary_centre_of_epilepsy_care=True)
        )
    elif abstraction_level == "country":
        q_filter = (
            Q(
                site__organisation__ons_region__ons_country=organisation_instance.ons_region.ons_country
            )
            & Q(site__site_is_actively_involved_in_epilepsy_care=True)
            & Q(site__site_is_primary_centre_of_epilepsy_care=True)
        )
    elif abstraction_level == "national":
        q_filter = Q(site__site_is_actively_involved_in_epilepsy_care=True) & Q(
            site__site_is_primary_centre_of_epilepsy_care=True
        )
    else:
        raise ValueError(
            f"Incorrect or invalid abstraction error f{abstraction_level} supplied."
        )

    return all_cases_for_cohort.filter(q_filter)
```

This function accepts the following parameters, and returns a queryset of cases:

- ```organisation_instance```: organisation instance, to identify the related group within which it sits.
- ```cohort```: this is an integer, referring to which cohort is requested
- ```case_complete```: this is a flag to allow the user to select whether all cases requested have been registered and have completed the audit, or just all cases known to that organisation in that cohort (that have registered).
- ```abstraction_level```: this is a string Literal defining level of abstraction (as defined above). It must be one of ```["organisation", "trust", "icb", "nhs_region", "open_uk", "country", "national"]``` and defaults to ```organisation```.

An organisation is defined as an epilepsy service that is both actively involved in the care of the child's epilepsy (ie no historical centres are included), and is the primary centre of care for the purposes of the Epilepsy12 audit.

### Selection of different levels of abstraction

There are several select queries that select regions by level of abstraction, rather than by case. These include:

- ```get_all_countries()```
- ```get_all_nhs_regions()```
- ```get_all_open_uk_regions()```
- ```get_all_icbs()```
- ```get_all_trusts()```
- ```get_all_organisations()```

## Aggregation

Aggregation is the process of counting, summing or averaging one or more fields for all rows in a queryset, optionally based on some criteria. It is a django tool and very quick. Functions that make use of this are found in ```epilepsy12/common_view_functions/aggregate_by.py```

### aggregate_all_eligible_kpi_fields

```python
def aggregate_all_eligible_kpi_fields(filtered_cases, kpi_measure=None):
    """
    Returns a dictionary of all KPI fields with aggregation for each measure ready to persist in KPIAggregations.
    It accepts a list of cases filtered by a given level of abstraction (all cases in an organisation, trust, icb etc)
    If an individual measure is passed in, only that measure will be aggregated.
    Returned fields include sum of all eligible KPI measures (identified as having an individual score of 1 or 0)
    for that registration as well as average score of the same and total number KPIs.
    A KPI score of 2 is excluded as not eligible for that measure.
    """

    all_kpi_measures = [
        "paediatrician_with_expertise_in_epilepsies",
        "epilepsy_specialist_nurse",
        "tertiary_input",
        "epilepsy_surgery_referral",
        "ecg",
        "mri",
        "assessment_of_mental_health_issues",
        "mental_health_support",
        "sodium_valproate",
        "comprehensive_care_planning_agreement",
        "patient_held_individualised_epilepsy_document",
        "patient_carer_parent_agreement_to_the_care_planning",
        "care_planning_has_been_updated_when_necessary",
        "comprehensive_care_planning_content",
        "parental_prolonged_seizures_care_plan",
        "water_safety",
        "first_aid",
        "general_participation_and_risk",
        "service_contact_details",
        "sudep",
        "school_individual_healthcare_plan",
    ]

    aggregation_fields = {}

    if kpi_measure:
        # a single measure selected for aggregation

        q_objects = Q(**{f"registration__kpi__{kpi_measure}__lt": 2}) & Q(
            **{f"registration__kpi__{kpi_measure}__isnull": False}
        )
        f_objects = F(f"registration__kpi__{kpi_measure}")

        # sum this measure
        aggregation_fields[f"{kpi_measure}"] = Sum(
            DJANGO_CASE(When(q_objects, then=f_objects), default=None)
        )
        # average of the sum of this measure
        aggregation_fields[f"{kpi_measure}_average"] = Avg(
            DJANGO_CASE(When(q_objects, then=f_objects), default=None)
        )

        # total cases scored for this measure
        aggregation_fields["total_number_of_cases"] = Count(
            DJANGO_CASE(When(q_objects, then=f_objects), default=None)
        )
    else:
        # aggregate all measures

        for measure in all_kpi_measures:
            # filter cases for all kpi with a score < 2
            q_objects = Q(**{f"registration__kpi__{measure}__lt": 2}) & Q(
                **{f"registration__kpi__{measure}__isnull": False}
            )  # & Q(**{f'registration__kpi__{measure}__isnull': False})
            f_objects = F(f"registration__kpi__{measure}")

            # sum this measure
            aggregation_fields[f"{measure}"] = Sum(
                DJANGO_CASE(When(q_objects, then=f_objects), default=0)
            )
            # average of the sum of this measure
            aggregation_fields[f"{measure}_average"] = Avg(
                DJANGO_CASE(When(q_objects, then=f_objects), default=None)
            )
            # total cases scored for this measure
            aggregation_fields[f"{measure}_total"] = Count(
                DJANGO_CASE(When(q_objects, then=f_objects), default=None)
            )
        # total_cases scored for all measures
        aggregation_fields["total_number_of_cases"] = Count(
            "registration__pk", default=None
        )

    return filtered_cases.aggregate(**aggregation_fields)
```

This function accepts a queryset of cases which have already been filtered according to a level of abstraction required. For example, using the filter query described above, all active and complete cases in a given trust might first be returned. An optional parameter is also included to aggregate a single key performance indicator only, but the default is to aggregate all of them.

If all are selected, the function iterates each KPI measure and creates a query parameter selecting only those cases where a score of 0,1 or None has been allocated to that child for that measure. Scores of 2 are excluded as these measures are ineligible.

New columns in the filter queryset are created in the aggregation process with the name of the KPI measure prefixing either ```_average``` or ```_total```. Total number of cases are also calculated. These are then passed back to be stored in the ```KPIAggregation``` table. Given that this process of aggregation is called every time a user lands on the organisation summary page, it was considered these queries might add a significant load to the main thread, so the results instead were stored in an asynchronous way to be pulled out in the view. It also allowed the queries to be throttled, with any public facing pages to draw results from older results, allowing some time for clinicians first to sense check the results before releasing them for public consumption.

The calling function for these queries is found in ```tasks.py``` in the function:

```python
def aggregate_kpis_for_each_level_of_abstraction_by_organisation_asynchronously(
    organisation_id: str, kpi_measure=None, open_access: bool = False
)
```

### return_all_aggregated_kpis_for_cohort_and_abstraction_level_annotated_by_sublevel

This function uses the previous function to return an aggregation of all KPIs at different levels of abstraction for a given cohort, with 'sublevel' within that level of abstraction labelled. For example, if the ```abstraction_level``` parameter is set to ```icb```, all children will be organised by Integrated Care Board and KPIs aggregated and labelled by ICB.

This returns therefore aggregated KPIs for the whole of England and Wales, organised by a given level of abstraction.

```python
def return_all_aggregated_kpis_for_cohort_and_abstraction_level_annotated_by_sublevel(
    cohort,
    abstraction_level: Literal[
        "organisation", "trust", "icb", "nhs_region", "open_uk", "country", "national"
    ] = "organisation",
    kpi_measure=None,
):
    """
    Returns aggregated KPIS for given cohort annotated by sublevel of abstraction (eg kpis in each NHS England region, labelled by region)
    """

    if abstraction_level == "organisation":
        abstraction_sublevels = get_all_organisations()

    if abstraction_level == "trust":
        abstraction_sublevels = get_all_trusts()

    if abstraction_level == "icb":
        abstraction_sublevels = get_all_icbs()

    if abstraction_level == "nhs_region":
        abstraction_sublevels = get_all_nhs_regions()

    if abstraction_level == "open_uk":
        abstraction_sublevels = get_all_open_uk_regions()

    if abstraction_level == "country":
        abstraction_sublevels = get_all_countries()

    # if abstraction_level == 'national':
    #     abstraction_sublevels = get_all_countries()
    #     abstraction_sublevel_Q = Q(site__organisation__CountryONSCode=abstraction_sublevel[0])
    # NOT NEEDED AS COVERED BY  ALL ORGANISATIONS

    final_object = []
    for abstraction_sublevel in abstraction_sublevels:
        if abstraction_level == "organisation":
            abstraction_sublevel_Q = Q(
                site__organisation__ODSCode=abstraction_sublevel.ODSCode
            )
            label = abstraction_sublevel.ODSCode
        if abstraction_level == "trust":
            abstraction_sublevel_Q = Q(
                site__organisation__ParentOrganisation_ODSCode=abstraction_sublevel.ParentOrganisation_ODSCode
            )
            label = abstraction_sublevel.ParentOrganisation_OrganisationName
        if abstraction_level == "icb":
            abstraction_sublevel_Q = Q(
                site__organisation__integrated_care_board__ODS_ICB_Code=abstraction_sublevel.ODS_ICB_Code
            )
            label = abstraction_sublevel.ICB_Name
        if abstraction_level == "nhs_region":
            abstraction_sublevel_Q = Q(
                site__organisation__nhs_region__NHS_Region_Code=abstraction_sublevel.NHS_Region_Code
            )
            label = abstraction_sublevel.NHS_Region
        if abstraction_level == "open_uk":
            abstraction_sublevel_Q = Q(
                site__organisation__openuk_network__OPEN_UK_Network_Code=abstraction_sublevel.OPEN_UK_Network_Code
            )
            label = abstraction_sublevel.OPEN_UK_Network_Name
        if abstraction_level == "country":
            abstraction_sublevel_Q = Q(
                site__organisation__ons_region__ons_country__Country_ONS_Code=abstraction_sublevel.Country_ONS_Code
            )
            label = abstraction_sublevel.Country_ONS_Name

        filtered_cases = Case.objects.filter(
            Q(site__site_is_actively_involved_in_epilepsy_care=True)
            & Q(site__site_is_primary_centre_of_epilepsy_care=True)
            & abstraction_sublevel_Q
            & Q(registration__cohort=cohort)
        )
        aggregated_kpis = aggregate_all_eligible_kpi_fields(
            filtered_cases, kpi_measure=kpi_measure
        )
        final_object.append(
            {
                "region": label,
                "aggregated_kpis": aggregated_kpis,
                "color": "#808080"
                if aggregated_kpis[kpi_measure] is None
                else "#000000",
            }
        )

    return final_object
```