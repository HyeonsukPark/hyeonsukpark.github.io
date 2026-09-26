---
title: LLM Evaluation - Evaluating LLM-Generated Travel Itineraries 
date: 2026-09-26
categories: [Blog Post, Project]
tags: [llm evaluation, project]     # TAG names should always be lowercase
---

# A Rule-Based Database Grounding Framework for Evaluating LLM-Generated Travel Itineraries

![img-description](/assets/images/project_logo.png)

## Introduction 

Large Language Models (LLMs) are rapidly transforming personalized travel planning by making it possible to generate complete travel itineraries in seconds. While these models can produce creative and detailed recommendations, they often struggle with factual accuracy and real-world reasoning. Common issues include hallucinated information, unrealistic travel schedules, ignored transit times, and recommendations that overlook business operating hours or weekly closures. 

Tranditionally, these limitations have been identified through manual evaluation, where human reviewers inspect LLM-generated itineraries and compare them against real-world constraints. This human-in-the-loop approach often provides more reliable assessments than using another LLM as a judge, as people can better detect subtle inconsistencies, factual errors, and practical issues. However, manual evaluation is both time-consuming and inherently subjective, with results varying depending on the evaluator's experience and perspective. 

To address these challenges, this project explores a lightweight and automated evaluation framework for validating LLM-generated travel itineraries. The project concept was inspired by Google Research's ["Optimizing LLM-based trip planning"](https://research.google/blog/optimizing-llm-based-trip-planning/) , which highlights the need for grounded, real-time data to fix feasibility issues. Instead of relying solely on human judgement, the framework combines a relational Ground Truth Database with a Python-based structures to verify the physical consistency of generated travel plans. The evaluation focuses on practical constraints such as travel distances, estimated transit times, opening hours, and scheduled closing days, enabling a more objective and reproducible assessment of itinerary quality. 

One challenge encountered during this work was the lack of publicly available datasets designed specifically for evaluating travel itineraries against real-word constraints. As this project was developed for personal learning, I created a synthetic dataset centered on planning trips to Paris. The dataset includes a variety of virtual itineraries designed to simulate both realistic and problematic scenarios, providing a controlled environment for testing and validating the proposed evaluation framework. 

**The project steps are:** 

1.  Create a persistent grounding database containing locations, relationships, and constraints.
2.  Prepare representative LLM-generated itinerary responses.
3.  Evaluate itinerary quality using rule-based checks. 

### [Project Repository](https://github.com/HyeonsukPark/LLM_Evaluation_Project/tree/main/itinerary_evaluation)

## System Architecture & Database Design

The system transitions from highly volatile in-memory storage to a persistent, file-based **SQLite** database (`travel_grounding.db`). This ensures that reference data is easily queryable and persistent across separate evaluation runs. 

The schema comprises three relational tables: 

**`Locations`** **Table:** 

-   Serves as the central entity registry storing place names (`name`) and metadata. 
-   Entity registry and taxonomy. 
-   `id`, `name`, `type`, `district`, `required_time`
-   `required_time` represents the recommended visit duration in hours.

---
  
**`Constraints` Table**: 

-   Operating rules
-   `location_id`, `closed_day`, `open_time`, `close_time`, `require_reservation`
-   `closed_day` uses `0–6` for Monday–Sunday and `7` for open every day.
-   `require_reservation` is stored as a Boolean integer: `1` for required and `0` for not required.

---
**`Relation_data` Table**:

-   Spatial Ontology
-    Models the spatial relationships between location pairs, storing transit distance (`distance_km`) and transit duration (`transit_min`).
-   `origin_id`, `destination_id`, `distance_km`, `transit_min`
-   Stores distance and estimated public-transit time between pairs of locations.

## Evaluation Metrics & Validation Logic

The framework automatically assesses LLM responses against two main spatiotemporal dimensions. 

### Hallucination and Execessive-Duration Checks 

The framework runs a baseline quality check using check\_quality function with three parameters: response, data, and max\_daily\_hours.  

- **Hallucination Detection** : The system parses the itinerary to verify whether the LLM hallucinated fictitious locations or relied on landmarks absent from the reference database. If an entity link fails to match the database registry, a grounding error is immediately flagged. 

- **Duration Constraints** : To simulate realistic traveler fatigue and feasible scheduling, a strict threshold of 8 hours of total daily visiting time max\_daily\_hours is enforced. If the cumulative duration of the suggested visits spent\_time exceeds this 8-hour limit, the framework returns an explicit alert message, identifying the itinerary as over-scheduled. 

### Temporal & Operational Constraint Validation - Weekly Closing Day Check

The function clsoing\_day\_check converts the planned itinerary data into a weekday index. The system then queries the constraints table via an inner join to verify if the targeted location is closed on that specific day. when an error detected, a message "\[location\] is closed on the planned weeday \[day\]" will be returned. 

### Geospatial & Transit Efficiency Validation - Route Evaluation   

The function check\_route\_evaluation and get\_route sequences the itinerary chronologically and queries the relation\_data table for consecutive location pairs to extract distance and transit times.  

 Location pairs 

[![](https://blogger.googleusercontent.com/img/a/AVvXsEjRqNKg8DDXCE6UpvABlgO0c8E1EBvfyWW4U93nU9teB5g_zSTxSwjJC_sioF0sraM4f9tbBh4saDr7_tE8G6vWh-CToUKt8AOwGO2wFyb9F94nZwsoGquZzcAJnTwlbsfIBVlPY6GUG1BHlxR5Cof_M_62cTO4ulidopaLLW0DVq789RcUBx5H9-c9DWQ=w160-h24)](https://blogger.googleusercontent.com/img/a/AVvXsEjRqNKg8DDXCE6UpvABlgO0c8E1EBvfyWW4U93nU9teB5g_zSTxSwjJC_sioF0sraM4f9tbBh4saDr7_tE8G6vWh-CToUKt8AOwGO2wFyb9F94nZwsoGquZzcAJnTwlbsfIBVlPY6GUG1BHlxR5Cof_M_62cTO4ulidopaLLW0DVq789RcUBx5H9-c9DWQ)

A critical guard clause _len(itinerary) <= 1_ was integrated. If an itinerary contains only a single spot, the system safely bypasses route validation to prevent variables from being unbound. 

An alert message occurs if a single transit segment exceeds 45 minutes or if the comulative daily travel exceeds 120 minutes. 

## Conclusion 

The project establishes a simple but reliable pipeline to ground LLM-generated outputs against a persistent SQL databases. By incorporating strict string nomalization and edge-case guard clauses, the verification script is now robust to run unattended. 

As a localized personal project designed to demonstrate a proof-of-concept evaluation architecture, the current implementation intentionally relies on a compact dataset consisting of 7 distinct landmarks and 8 synthetic LLM itineraries. While this small-scale deployment successfully validates the rule engine's logic, a larger volume of data is required to unlock deeper analytical insights.

The project can broaden the relational schema to evaluate dynamic constraints. This includes integrating real-time and seasonal operating variations, weather forecasts, and specific criterias such as specific dietary requirements for restaurants.






