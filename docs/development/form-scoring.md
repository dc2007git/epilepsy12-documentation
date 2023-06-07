---
title: Scoring the audit forms
reviewers: Dr Simon Chapman
---

This is the process of tracking user progress through scoring all the elements of the child's journey through the epilepsy12 audit. Progress is tracked in the ```AuditProgress``` model which has the following fields:

- ```registration_complete```
- ```registration_total_expected_fields```
- ```registration_total_completed_fields```
- ```first_paediatric_assessment_complete```
- ```first_paediatric_assessment_total_expected_fields```
- ```first_paediatric_assessment_total_completed_fields```
- ```assessment_complete```
- ```assessment_total_expected_fields```
- ```assessment_total_completed_fields```
- ```epilepsy_context_complete```
- ```epilepsy_context_total_expected_fields```
- ```epilepsy_context_total_completed_fields```
- ```multiaxial_diagnosis_complete```
- ```multiaxial_diagnosis_total_expected_fields```
- ```multiaxial_diagnosis_total_completed_fields```
- ```investigations_complete```
- ```investigations_total_expected_fields```
- ```investigations_total_completed_fields```
- ```management_complete```
- ```management_total_expected_fields```
- ```management_total_completed_fields```

For each form, a boolean flag tracks if the form is complete, how many fields in the form have been completed so far as an integer, and how many are expected, also as an integer. This is because the denominator is dynamic - the minimum number of fields expected to be scored to complete the form changes based on the user choices. For example, in ```MultiaxialDiagnosis```, if the user selects 'yes' to 'Is there an identifiable epilepsy syndrome?', they are invited to add a syndrome and the date of diagnosis: this increases the number of expected fields therefore by 2.

The final step, is to update the ```AuditProgress``` with these results and call the ```calculate_kpis()``` function.

## ```total_expected_fields``` vs ```total_completed_fields```

The logic calculating user progress can be summarise by these two fields and is found in ```common_view_functions/recalculate_form_generate_response.py```.

### ```total_completed_fields```

This is calculated from two functions:

- ```completed_fields()```
- ```number_of_completed_fields_in_related_models()```

Both functions accept a registration instance, from which all models can be accessed.

```python
def completed_fields(model_instance):
    """
    Test for all completed fields
    Returns an integer number of completed fields for a given model instance.
    """
    fields = model_instance._meta.get_fields()
    counter = 0
    fields_to_avoid = avoid_fields(model_instance)

    for field in fields:
        if field.name not in fields_to_avoid:
            if getattr(model_instance, field.name, ()) is not None:
                if (
                    field.name == "epilepsy_cause_categories"
                    or field.name == "description"
                ):
                    if len(getattr(model_instance, field.name)) > 0:
                        counter += 1
                else:
                    if field.name in [
                        "focal_onset_atonic",
                        "focal_onset_clonic",
                        "focal_onset_epileptic_spasms",
                        "focal_onset_hyperkinetic",
                        "focal_onset_myoclonic",
                        "focal_onset_tonic",
                        "focal_onset_focal_to_bilateral_tonic_clonic",
                        "focal_onset_automatisms",
                        "focal_onset_impaired_awareness",
                        "focal_onset_gelastic",
                        "focal_onset_autonomic",
                        "focal_onset_behavioural_arrest",
                        "focal_onset_cognitive",
                        "focal_onset_emotional",
                        "focal_onset_sensory",
                        "focal_onset_centrotemporal",
                        "focal_onset_temporal",
                        "focal_onset_frontal",
                        "focal_onset_parietal",
                        "focal_onset_occipital",
                        "focal_onset_right",
                        "focal_onset_left",
                    ]:
                        if getattr(model_instance, field.name, ()) == True:
                            # only count the true values in the radio buttons in focal epilepsy to do with focality
                            if field.name in ["focal_onset_right", "focal_onset_left"]:
                                counter += 1
                    else:
                        counter += 1
    return counter
```

This function loops through all fields in a given model instance counting up all the fields that are not None (since all fields are None until scored). Several fields in each model have to be excluded, since they are not storing information about the audit. These include fields such as the primary key. This stripping of fields occurs in ```fields_to_avoid()```.

Some other fields are not included - ```epilepsy_cause_categories``` and ```description``` in multiaxial diagnosis are not included. The boolean fields describing features present or absent in a focal epilepsy are exlcuded - only focality (left or right) count to the total.

```number_of_completed_fields_in_related_models``` performs this same process for all fields in related models. For example, a single multiaxial description of a child's epilepsy comprises multiple episodes, all of which are different. 

```python
def number_of_completed_fields_in_related_models(model_instance):
    """
    Counts completed fields in models related to modelinstance passed in as parameter.
    Returns an integer number of completed fields
    If there are no related models, zero is returned.
    """
    cumulative_score = 0
    if model_instance.__class__.__name__ == "MultiaxialDiagnosis":
        # also need to count associated records in Episode, Syndrome and Comorbidity
        episodes = Episode.objects.filter(multiaxial_diagnosis=model_instance).all()
        syndromes = Syndrome.objects.filter(multiaxial_diagnosis=model_instance).all()
        comorbidities = Comorbidity.objects.filter(
            multiaxial_diagnosis=model_instance
        ).all()
        if episodes.count() > 0:
            for episode in episodes:
                calculated_score = completed_fields(episode)
                # save the episode calculated score in the model instance
                episode.calculated_score = calculated_score
                episode.save()
                cumulative_score += calculated_score
        if syndromes.count() > 0:
            for syndrome in syndromes:
                cumulative_score += completed_fields(syndrome)
        if comorbidities.count() > 0:
            for comorbidity in comorbidities:
                cumulative_score += completed_fields(comorbidity)
    elif model_instance.__class__.__name__ == "Assessment":
        # also need to count associated records in Site
        sites = Site.objects.filter(
            case=model_instance.registration.case,
            site_is_actively_involved_in_epilepsy_care=True,
        ).all()
        if sites:
            for site in sites:
                if site.site_is_childrens_epilepsy_surgery_centre:
                    cumulative_score += 1
                if site.site_is_general_paediatric_centre:
                    cumulative_score += 1
                if site.site_is_paediatric_neurology_centre:
                    cumulative_score += 1
    elif model_instance.__class__.__name__ == "Management":
        # also need to count associated records in AntiepilepsyMedicines
        if model_instance.has_an_aed_been_given:
            # antiepilepsy drugs
            medicines = AntiEpilepsyMedicine.objects.filter(
                management=model_instance, is_rescue_medicine=False
            ).all()
            if medicines.count() > 0:
                for medicine in medicines:
                    # essential fields are:
                    # medicineentity_medicine_name', 'antiepilepsy_medicine_start_date',
                    # 'antiepilepsy_medicine_risk_discussed', and if valproate prescribed
                    # in a girl > 12y, 'is_a_pregnancy_prevention_programme_in_place'
                    # 'has_a_valproate_annual_risk_acknowledgement_form_been_completed'
                    cumulative_score += completed_fields(medicine)

        if model_instance.has_rescue_medication_been_prescribed:
            # rescue drugs
            medicines = AntiEpilepsyMedicine.objects.filter(
                management=model_instance, is_rescue_medicine=True
            ).all()
            if medicines.count() > 0:
                for medicine in medicines:
                    # essential fields are:
                    # medicine_name', 'antiepilepsy_medicine_start_date',
                    # 'antiepilepsy_medicine_risk_discussed'
                    cumulative_score += completed_fields(medicine)

    elif model_instance.__class__.__name__ == "Registration":
        # also need to count associate record in Site
        if Site.objects.filter(
            case=model_instance.case,
            site_is_primary_centre_of_epilepsy_care=True,
            site_is_actively_involved_in_epilepsy_care=True,
        ).exists():
            cumulative_score += 1

    return cumulative_score
```

Related models are only found in the ```MultiaxialDiagnosis``` model (```Episode```, ```Syndrome``` and ```Comorbidity```), ```Assessment``` (```Site```), ```Management``` (```AntiepilepsyMedicine```) and ```Registration``` (```Site```). If there are related models for each of these, this function loops through each, adding up the completed fields.


### ```total_expected_fields```

This is arguable more complex as Epilepsy12 has a large number of permutations. If a child has not been referred to a paediatric neurologist, they cannot be scored for the date they were referred on or seen.

```python
def total_fields_expected(model_instance):
    """
    returns as expected fields for a given model instance, based on user selections
    """

    model_class_name = model_instance.__class__.__name__
    # get the minimum number of fields for this model
    cumulative_score = scoreable_fields_for_model_class_name(
        model_class_name=model_class_name
    )

    if model_instance.__class__.__name__ == "MultiaxialDiagnosis":
        # count episodes - note
        # at least one episode must be epileptic

        episodes = Episode.objects.filter(multiaxial_diagnosis=model_instance).all()

        # loop through all episodes and count the fields
        # if there are none, return the minimum score for an epileptic seizure
        cumulative_score += count_episode_fields(episodes)

        # syndromes are optional but if present add essential fields
        if model_instance.syndrome_present:
            if Syndrome.objects.filter(multiaxial_diagnosis=model_instance).exists():
                # there are syndromes - increase total to include essential fields per syndrome
                number_of_syndromes = Syndrome.objects.filter(
                    multiaxial_diagnosis=model_instance
                ).count()
                cumulative_score += (
                    scoreable_fields_for_model_class_name("Syndrome")
                    * number_of_syndromes
                )
            else:
                # no syndromes yet but user indicated present - add essential fields for syndromes
                cumulative_score += scoreable_fields_for_model_class_name("Syndrome")

        # if a cause for the epilepsy is know other essential fields must be included
        if model_instance.epilepsy_cause_known:
            # essential fields include
            # epilepsy_cause
            # epilepsy_cause_categories - this is an array, length must be greater than one
            cumulative_score += 2

        if model_instance.relevant_impairments_behavioural_educational:
            # there are comorbidities - add essential comorbidities
            number_of_comorbidities = Comorbidity.objects.filter(
                multiaxial_diagnosis=model_instance
            ).count()
            essential_fields_per_comorbidity = scoreable_fields_for_model_class_name(
                "Comorbidity"
            )
            if number_of_comorbidities < 1:
                # comorbidities not yet scored but user has indicated there are some present
                # increase the total by minimum number required
                cumulative_score += essential_fields_per_comorbidity
            else:
                cumulative_score += (
                    essential_fields_per_comorbidity * number_of_comorbidities
                )

        if model_instance.mental_health_issue_identified:
            # essential fields increase to include
            # mental_health_issue
            cumulative_score += 1

        if model_instance.global_developmental_delay_or_learning_difficulties:
            # essential fields increase to include
            # global_developmental_delay_or_learning_difficulties_severity
            cumulative_score += 1

    elif model_class_name == "Assessment":
        if model_instance.consultant_paediatrician_referral_made:
            # add essential fields: date referred, date seen, centre
            cumulative_score += 3
        if model_instance.paediatric_neurologist_referral_made:
            # add essential fields: date referred, date seen, centre
            cumulative_score += 3
        if model_instance.childrens_epilepsy_surgical_service_referral_made:
            # add essential fields: date referred, date seen, centre
            cumulative_score += 3
        if model_instance.epilepsy_specialist_nurse_referral_made:
            # add essential fields: date referred, date seen
            cumulative_score += 2

    elif model_class_name == "Investigations":
        if model_instance.eeg_indicated:
            # add essential fields: request date, performed_date
            cumulative_score += 2
        if model_instance.mri_indicated:
            # add essential fields: request date, performed_date
            cumulative_score += 2

    elif model_class_name == "Management":
        # also need to count associated records in AntiepilepsyMedicines
        if model_instance.has_an_aed_been_given:
            # antiepilepsy drugs
            medicines = AntiEpilepsyMedicine.objects.filter(
                management=model_instance, is_rescue_medicine=False
            ).all()
            if medicines.count() > 0:
                for medicine in medicines:
                    # essential fields are:
                    # medicine_name', 'antiepilepsy_medicine_start_date',
                    # 'antiepilepsy_medicine_risk_discussed'
                    # NOTE 'antiepilepsy_medicine_stop_date' is not an essential field

                    cumulative_score += 3

                    today = date.today()
                    calculated_age = relativedelta.relativedelta(
                        today, model_instance.registration.case.date_of_birth
                    )

                    if (
                        hasattr(medicine, "medicine_entity")
                        and model_instance.registration.case.sex == 2
                        and calculated_age.years >= 12
                    ):
                        if medicine.medicine_entity is not None:
                            if (
                                medicine.medicine_entity.medicine_name
                                == "Sodium valproate"
                            ):
                                # essential fields are:
                                # 'is_a_pregnancy_prevention_programme_needed' - this is not scored
                                if medicine.is_a_pregnancy_prevention_programme_needed:
                                    # essential fields are:
                                    # 'is_a_pregnancy_prevention_programme_in_place, 'has_a_valproate_annual_risk_acknowledgement_form_been_completed'
                                    cumulative_score += 2
            else:
                # user has said AED given but not scored yet
                cumulative_score += scoreable_fields_for_model_class_name(
                    "AntiEpilepsyMedicine"
                )

        if model_instance.has_rescue_medication_been_prescribed:
            # rescue drugs
            medicines = AntiEpilepsyMedicine.objects.filter(
                management=model_instance, is_rescue_medicine=True
            ).all()
            if medicines.count() > 0:
                for medicine in medicines:
                    # essential fields are:
                    # medicine_name', 'antiepilepsy_medicine_start_date',
                    # 'antiepilepsy_medicine_risk_discussed'
                    cumulative_score += 3
            else:
                # user has said AED given but not scored yet
                cumulative_score += scoreable_fields_for_model_class_name(
                    "AntiEpilepsyMedicine"
                )

        if model_instance.individualised_care_plan_in_place:
            # add essential fields:
            # individualised_care_plan_date, individualised_care_plan_has_parent_carer_child_agreement,
            # individualised_care_plan_includes_service_contact_details, individualised_care_plan_include_first_aid,
            # individualised_care_plan_parental_prolonged_seizure_care, individualised_care_plan_includes_general_participation_risk,
            # individualised_care_plan_addresses_water_safety, individualised_care_plan_addresses_sudep,
            # individualised_care_plan_includes_ehcp, has_individualised_care_plan_been_updated_in_the_last_year
            cumulative_score += 10

    elif model_class_name == "Registration":
        # further essential field is from Site - primary site of care
        cumulative_score += 1

    return cumulative_score
```

This function steps through the audit questions one by one, calculating the minimum number of fields that can be expected to be completed, based on the selections the user has already made. As with the other functions it accepts an instance of the ```Registration``` model that relates to the child in question. It draws on two other helper functions:

- ```scoreable_fields_for_model_class_name()```
- ```count_episode_fields()```

These both do the same for related models if they exist. The ```Episode``` model is particularly complicated as it details all the different epilepsy types each of which have different decision trees that contribute to the scores.