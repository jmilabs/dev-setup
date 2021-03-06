# Existing Code Write Up

**ToDo**
- [x] Overview of MIC, testing and results.
- [x] Identify (filter) which isolates should be included in this execution
- [x] Demonstrate how the generated sql enforces those filters for attributes that come from each of the possible tables:  isolates, organisms, sites, isolate_attributes:
  * (validated\_mic_results can be used in filters as well but hold off on that)
- [ ] Document how the drug/mic/edge/count that comes back are stored into `frequency_distribution.rb`, and then how they are normalized in different cases (2 lows, 2 highs) in instances of `frequency_distribution.rb`
- [ ] Document how the `isolate_drug_reactions` data fields:
  * authority
  * publication
  * delivery_mechanism
  * infection_type
  * interpretation
  * eligible_interpretations
  * footnotes
  are queried for and stored in `drug_reaction_set.rb`
- [ ] Adapt the `fd_to_csv` / `fd_to_csv_debug methods` to serialize all frequency\_distribution instances returned by an isolate_filter instance into a single nested json/hash structure that contains all known data

## Outline

* [Background](#background)
* [Overview](#overview)
	* [1. Retrieve MIC frequency count and create a frequency distribution.](#step-1-get-the-results-of-the-mic-testing-for-each-drug--organism-combination-only-where-the-isolates-of-the-test-match-the-filtering-criteria)
	* [2. Retrieve a count of organisms included in each frequency distribution](#step-2-get-a-count-of-organisms-included-in-each-frequency-distribution)

## Background 

In microbiology, **Minimum Inhibitory Concentration (MIC)** is the lowest concentration of an antimicrobial that will inhibit the visible growth of a microorganism after overnight incubation. At **JMI Labs**, we several different kinds of tests to identify the MIC for a particular drug and organism combination. These tests include:

  * **MIC Panel** 
  	 <br />
  	 <img src="images/mic_panel.png" alt="Drawing" width="400px"/>
  * **E-Test** 
    <br />
  	 <img src="images/e_test.png" alt="Drawing" width="400px"/>
  * **Agar Dilution** 
    <br />
  	 <img src="images/agar_dilution.png" alt="Drawing" width="400px"/>
  * **Disk Diffusion** 
    <br />
  	 <img src="images/disk_diffusion.png" alt="Drawing" width="400px"/>

In order to perform this testing, **JMI Labs** works with hospitals around the world to get many different samples of an organism, these are called isolates. Within our labs, we have tens of thousands of different organisms each with unique properties. 

We use this wide range of isolates to provide our clients with a normalized frequency distribution of all MIC results for a particular drug / organism combination. Such results look similar to the following table:

* Todo Add description of breakpoints and authorities

<img src="images/staphylococcus_aureus_fd.png" alt="Drawing" width="600px" /> 

## Step 1: Overview

Isolates can be filtered and grouped by any combination of the following attributes:

```
# :isolate_id
# :isolate_number
# :validation_status_code
# :bank_number
# :continent
# :country
# :genus
# :gram
# :level_1
# :level_2
# :level_3
# :level_4
# :master_group
# :objective
# :organism_code
# :organism
# :organism_group
# :viridans_group
# :site
# :species
# :us_census_region
# :year
# :category_value
# :client_category_value # for using direct published client data sets
# :client_infection_type # for using direct published client data sets
# :client_program_code # for using direct published client data sets
# :reacts_like_to
# [
#   {
#     authority: 'CLSI',
#     publication: '2015',
#     delivery_mechanism: '',
#     infection_type: '',
#     drug: 'Oxacillin',
#     interpretations: %w( NS I R )
#   }
# ]
# :dilutions_differ_by
# [
#   drug_1: "Imipenem",
#   drug_2: "ImipenemSENTRY",
#   allowed_difference: 2
#   operator: :at_most, :at_least
# ]
# :drug_dilution_criteria
# becomes AND of all conditions
# [
#   {
#     drug: "Imipenem",
#     comparator: :gte | :gt | :lte | :lt | :eq
#     dilution: 32
#   },
#   {
#     drug: "Meropenem",
#     comparator: :gte | :gt | :lte | :lt | :eq
#     dilution: 16
#   }
# ]
# :run_number
# :position_in_run
# :panel_name
# :panel_lot
# :read_date
# :reader_name
```
Each of the attributes are passed in as the value of either an `:include` or `:exclude` keyed hash.

_Examples_

```
year: { include: [ 2014 ] },
organism_code: { include: %w( SA ) },
organism_code: { excule: %w( SH ) },
client_category_value: { include: 'MSSA' },
...
```


### Create the filter

To identify the isolates based on the unique combination of attributes we need to instantiate a new IsolateFilter:

```
isolate_filter = IsolateFilter.new(name: "Example")
```

Next we have to set the database _prefix_ to select which dataset we should use in the analysis:

```
isolate_filter.setTablePrefix( 'example_' )
```

### Add attributes and values to the filter

Adding filter criteria to the isolate filter is easy

```
isolate_filter.setAttributeFilter(attribute_name: { [include|exclude]: filter_value })
```

_Example_

```
isolate_filter.setAttributeFilter( year: { include: [ 2014 ] } )
```

### Calculate Frequency Distributions

In order to calculate the frequency distribution of MIC results for all isolates matched by the `isolate_filter`, we need to indicate which 
drugs and breakpoint authorities to include in the analysis:

_Primary Drug Example_

```
primary_drugs = [
  "Ceftazidime",
  "Ceftriaxone",
  "Cefepime",
  "Oxacillin",
  "Penicillin",
  "Ampicillin",
  "AmoxClav",
  "AmpSulb",
  "PipTazo",
  "Meropenem",
  "Aztreonam",
  "Erythromycin",
  "Azithromycin",
  "Clarithromycin",
  "Clindamycin",
  "Ciprofloxacin",
  "Levofloxacin",
  "Gentamicin",
  "Amikacin",
  "TrimSulfa",
  "Teicoplanin",
  "Tetracycline",
  "Tigecycline",
  "Linezolid",
  "Vancomycin",
  "Daptomycin",
  "Colistin"
]
```

_Breakpoint Authorities Example_

```
breakpoint_authorities = [
  { authority: 'CLSI', publication: '2015' },
  { authority: 'EUCAST', publication: '2015' }
]
```

Now that we have the drugs and breakpoint_authorities defined, we can calculate the frequency distribution. 

```
isolate_filter.calculateFrequencyDistributions(
  primary_drugs,
  breakpoint_authorities
)
```

The frequency distribution results are stored within the `isolate_filter.frequency_distribution` instance variable.

### Understanding how `isolate_filter.rb` works

`isolate_filter.rb` works by building complex sql queries consisting of **inner joins** and **where clauses** to filer the selected isolate attributes.

#### Step 1. Get the results of the MIC testing for each drug / organism combination, _only_ where the isolates of the test match the filtering criteria. 

_Example SQL Query_

```
select
d.name as drug,
vmr.mic as mic,
vmr.edge as edge,
count(1) as frequency
from
ceftaroline_validated_mic_results vmr
inner join ceftaroline_isolates i on i.id = vmr.isolate_id
inner join organisms o on i.organism_code = o.organism_code
inner join drugs d on vmr.drug_id = d.id
inner join sites s on i.site_code = s.site_code
left outer join us_census_regions uscr on
s.state_province = uscr.state_abbreviation
where
  1 = 1
  and vmr.drug_id in ( 1001 /* Amikacin */,1002 /* AmoxClav */, ... )
  and i.surveillance_year in ( 2014 )
  and i.organism_code in ( 'SA' )
  and s.country in ( 'USA' )
group by
d.name,
vmr.mic,
vmr.edge
order by
d.name,
vmr.mic,
vmr.edge
```

_Example Results_

| drug | mic | edge | frequency |          
| --- | --- | :---: | --- |      
| AmoxClav | 1.000000 | -1 | 3865 |    
| AmoxClav | 2.000000| 0 | 104 |
| AmoxClav | 4.000000| 0 | 142 |
| AmoxClav | 8.000000| 0 | 685 |
| AmoxClav | 8.000000| 1 | 2656 |
| ... | ... | ... | ... |
| Vancomycin | 0.250000 | 0 | 11 |
| Vancomycin | 0.500000 | 0 | 1599 |
| Vancomycin | 1.000000 | 0 | 5779 |
| Vancomycin | 2.000000 | 0 | 64 |

<hr />

To ensure that only the correct isolates are included in the **mic frequency count**, `isolate_filter.rb` generates a where clause based on the filtered attributes.

For more information see the code docs:

* [IsolateFilter#getWhereClause](doc/IsolateFilter.html#getWhereClause-instance_method)

```
where
  1 = 1
  and vmr.drug_id in ( 1001 /* Amikacin */,1002 /* AmoxClav */, ... )
  and i.surveillance_year in ( 2014 )
  and i.organism_code in ( 'SA' )
  and s.country in ( 'USA' )
```

The only **isolate attribute filters** that are not incuded by the `#getWhereClause` method are the following:

* `:category_value` - [IsolateFilter#addCategoryValueTables](doc/IsolateFilter.html#addCategoryValueTables-instance_method)
* `:reacts_like_to` - [IsolateFilter#addReactsLikeTo](doc/IsolateFilter.html#addReactsLikeTo-instance_method)
* `:drug_dilution_criteria` - [IsolateFilter#addDrugDilutionCriteria](doc/IsolateFilter.html#addDrugDilutionCriteria-instance_method)
* `:dilutions_differ_by` - [IsolateFilter#addDilutionsDifferBy](doc/IsolateFilter.html#addDilutionsDifferBy-instance_method)

<hr />

Now that we have a the mic result frequency count for all drug / isolate combinations, we need to create a distribution frequency for each drug for this filter.

```
SQLUtil::getConnection.exec( query ).each{ | row |
      fd_key = {
        filters: @filters,
        drug:
        row[ "drug" ],
        group_by: @group_by,
        group_by_values: {}
      }
      @group_by.each{ | k |
        fd_key[ :group_by_values ][ k ] = row[ getBareColumnName( k ) ]
      }
      @frequency_distributions[ fd_key ] ||=
        FrequencyDistribution.new( fd_key )
      @frequency_distributions[ fd_key ].addUnnormalizedDilutionFrequency(
        row[ "mic" ].to_f,
        row[ "edge" ].to_i,
        row[ "frequency" ].to_i
      )
    }
```

#### Step 2. Get a count of organisms included in each frequency distribution

_Example SQL Query_

```
select
d.name as drug,
i.organism_code as organism_code,
count(1) as frequency
from
ceftaroline_validated_mic_results vmr
inner join ceftaroline_isolates i on i.id = vmr.isolate_id
inner join organisms o on i.organism_code = o.organism_code
inner join drugs d on vmr.drug_id = d.id
inner join sites s on i.site_code = s.site_code
left outer join us_census_regions uscr on
s.state_province = uscr.state_abbreviation
where
  1 = 1
  and vmr.drug_id in ( 1001 /* Amikacin */,1002 /* AmoxClav */, ...)
  and i.surveillance_year in ( 2014 )
  and i.organism_code in ( 'SA' )
  and s.country in ( 'USA' )
group by
d.name,
i.organism_code
order by
d.name,
i.organism_code
```

_Example Results_


| drug | organism_code | frequency |          
| --- | --- | --- |   
| AmoxClav | SA | 7452 |
| ... | ... | ... |
| Vancomycin | SA | 7453 |

Now store the results in the frequency distirbution by calling `FrequencyDistribution#addOrganismCodeFrequency `. 

```
SQLUtil::getConnection.exec( query ).each{ | row |
  fd_key = {
    filters: @filters,
    drug: row[ "drug" ],
    group_by: @group_by,
    group_by_values: {}
  }
  @group_by.each{ | k |
    fd_key[ :group_by_values ][ k ] = row[ getBareColumnName( k ) ]
  }
  if ( @frequency_distributions[ fd_key ].nil? )
    throw "can't find frequency distribution for #{ fd_key }"
  end
  @frequency_distributions[ fd_key ].addOrganismCodeFrequency(
    row[ "organism_code" ],
    row[ "frequency" ].to_i
  )
}
```

#### Step 3. Retrieve drug reactions (resistance) counts

This is done by calling the `IsolateFilter#calculateDrugReactions` method (details can be found [here](doc/IsolateFilter.html#calculateDrugReactions-instance_method)).

_Example SQL Query_

```
select
d.name as drug,
idr.authority as authority,
idr.publication as publication,
idr.delivery_mechanism as delivery_mechanism,
idr.infection_type as infection_type,
idr.reaction as reaction,
count(1) as frequency
from
ceftaroline_validated_mic_results vmr
inner join ceftaroline_isolates i on i.id = vmr.isolate_id
inner join organisms o on i.organism_code = o.organism_code
inner join drugs d on vmr.drug_id = d.id
inner join sites s on i.site_code = s.site_code
left outer join us_census_regions uscr on
s.state_province = uscr.state_abbreviation
inner join ceftaroline_isolate_drug_reactions idr on (
idr.validated_mic_result_id = vmr.id
)
where
  1 = 1
  and vmr.drug_id in ( 1001 /* Amikacin */,1002 /* AmoxClav */, ... )
  and (
    (
      idr.authority = 'CLSI' and
      idr.publication = '2015'
    ) or
    (
      idr.authority = 'EUCAST' and
      idr.publication = '2015'
    ) or
    0=1
  )
  and i.surveillance_year in ( 2014 )
  and i.organism_code in ( 'SA' )
  and s.country in ( 'USA' )
group by
d.name,
idr.authority,
idr.publication,
idr.delivery_mechanism,
idr.infection_type,
idr.reaction
order by
d.name,
idr.authority,
idr.publication,
idr.delivery_mechanism,
idr.infection_type,
idr.reaction
```

_Example Results_

| drug     | authority | publication | delivery_mechanism | infection_type | reactions | frequency |
| --- | --- | --- | --- | --- | :---: | --- | 
| AmoxClav | CLSI      | 2015        |                    |                | R         | 3577      |
| AmoxClav | CLSI      | 2015        |                    |                | S         | 3875      |
| AmoxClav | EUCAST      | 2015        |                    |                | R         | 3577      |
| AmoxClav | EUCAST      | 2015        |                    |                | S         | 3875      |
| ... | ...| ... | ... | ... | ... | ... |
| Vancomycin | CLSI      | 2015        |                    |                | S         | 7453      |
| Vancomycin | EUCAST      | 2015        |                    |                | S         | 7453      |

Now store the results in the frequency distirbution by calling `FrequencyDistribution#addDrugReaction`. 

_**Note:** We are not including MDR as a 'hack' right now. This will be removed in future versions._

```
SQLUtil::getConnection.exec( query ).each{ | row |
  fd_key = {
    filters: @filters,
    drug: row[ "drug" ],
    group_by: @group_by,
    group_by_values: {}
  }
  @group_by.each{ | k |
    fd_key[ :group_by_values ][ k ] = row[ getBareColumnName( k ) ]
  }
  fd = @frequency_distributions[ fd_key ]
  if ( fd.nil? )
    throw "Trying to add drug reactions to non existent frequency distribution with key #{ fd_key }"
  end
  fd.addDrugReaction(
    row[ "authority" ],
    row[ "publication" ],
    row[ "delivery_mechanism" ],
    row[ "infection_type" ],
    row[ "reaction" ],
    row[ "frequency" ]
  ) unless row[ "delivery_mechanism" ].include?( 'MDR' )
}
```


#### Step 4. Retrieve drug reactions (resistance) footnotes

This is done by calling the `IsolateFilter#calculateDrugReactions` method (details can be found [here](doc/IsolateFilter.html#calculateDrugReactions-instance_method)).

_Example SQL Query_

```
select
d.name as drug,
idr.authority as authority,
idr.publication as publication,
idr.delivery_mechanism as delivery_mechanism,
idr.infection_type as infection_type,
idr.footnote as footnote,
count(1) as frequency
from
ceftaroline_validated_mic_results vmr
inner join ceftaroline_isolates i on i.id = vmr.isolate_id
inner join organisms o on i.organism_code = o.organism_code
inner join drugs d on vmr.drug_id = d.id
inner join sites s on i.site_code = s.site_code
left outer join us_census_regions uscr on
s.state_province = uscr.state_abbreviation
inner join ceftaroline_isolate_drug_reactions idr on (
idr.validated_mic_result_id = vmr.id
)
where
  1 = 1
  and idr.footnote is not null
  and vmr.drug_id in ( 1001 /* Amikacin */,1002 /* AmoxClav */, ...)
  and (
    (
      idr.authority = 'CLSI' and
      idr.publication = '2015'
    ) or
    (
      idr.authority = 'EUCAST' and
      idr.publication = '2015'
    ) or
    0=1
  )
  and i.surveillance_year in ( 2014 )
  and i.organism_code in ( 'SA' )
  and s.country in ( 'USA' )
group by
d.name,
idr.authority,
idr.publication,
idr.delivery_mechanism,
idr.infection_type,
idr.footnote
order by
d.name,
idr.authority,
idr.publication,
idr.delivery_mechanism,
idr.infection_type,
idr.footnote
```

_Example Result_

| drug | authority | publication | delivery_mechanism | infection_type | footnote | frequency |    
| --- | --- | --- | --- | --- | --- | --- |   
| Tigecycline | CLSI | 2015 | | | Breakpoints from FDA Package Insert revised 12/2014 | 7435 |

Now store the resulting footnotes in the frequency distirbution by calling `FrequencyDistribution#addDrugReactionFootnote `. 

_**Note:** We are not including MDR as a 'hack' right now. This will be removed in future versions._


```
SQLUtil::getConnection.exec( query ).each{ | row |
  fd_key = {
    filters: @filters,
    drug: row[ "drug" ],
    group_by: @group_by,
    group_by_values: {}
  }
  @group_by.each{ | k |
    fd_key[ :group_by_values ][ k ] = row[ getBareColumnName( k ) ]
  }
  fd = @frequency_distributions[ fd_key ]
  if ( fd.nil? )
    throw "Trying to add footnote to non existent frequency distribution with key #{ fd_key }"
  end
  fd.addDrugReactionFootnote(
    row[ "authority" ],
    row[ "publication" ],
    row[ "delivery_mechanism" ],
    row[ "infection_type" ],
    row[ "footnote" ]
  ) unless row[ "delivery_mechanism" ].include?( 'MDR' )
}
```

#### Step 5. Retrieve drug reactions (resistance) eligible interpretations.

This is done by calling the `IsolateFilter#calculateDrugReactions` method (details can be found [here](doc/IsolateFilter.html#calculateDrugReactions-instance_method)).

```
# Within IsolateFilter#calculateFrequencyDistributions
calculateDrugReactionEligibleInterpretations(
  drugs,
  breakpoint_authorities,
  merge_disparate_eligible_interpretations
)
```

_Example SQL Query_

```
select
d.name as drug,
idr.authority as authority,
idr.publication as publication,
idr.delivery_mechanism as delivery_mechanism,
idr.infection_type as infection_type,
idr.eligible_interpretations as eligible_interpretations,
idr.acceptable_dilutions as acceptable_dilutions,
count(1) as frequency
from
ceftaroline_validated_mic_results vmr
inner join ceftaroline_isolates i on i.id = vmr.isolate_id
inner join organisms o on i.organism_code = o.organism_code
inner join drugs d on vmr.drug_id = d.id
inner join sites s on i.site_code = s.site_code
left outer join us_census_regions uscr on
s.state_province = uscr.state_abbreviation
inner join ceftaroline_isolate_drug_reactions idr on (
  idr.validated_mic_result_id = vmr.id
)
where
  1 = 1
  and vmr.drug_id in ( 1001 /* Amikacin */,1002 /* AmoxClav */, ... )
  and (
    (
      idr.authority = 'CLSI' and
      idr.publication = '2015'
    ) or
    (
      idr.authority = 'EUCAST' and
      idr.publication = '2015'
    ) or
    0=1
  )
  and i.surveillance_year in ( 2014 )
  and i.organism_code in ( 'SA' )
  and s.country in ( 'USA' )
group by
d.name,
idr.authority,
idr.publication,
idr.delivery_mechanism,
idr.infection_type,
idr.eligible_interpretations,
idr.acceptable_dilutions
order by
d.name,
idr.authority,
idr.publication,
idr.delivery_mechanism,
idr.infection_type,
idr.eligible_interpretations,
idr.acceptable_dilutions
```

_Example Results_

| drug | authority | publication | delivery_mechanism | infection_type | eligible_interpretations | acceptable_dilutions | frequency |   
| --- | --- | --- | --- | --- | --- | --- | --- | 
| AmoxClav | CLSI | 2015 | | | S R | | 7452 |
| AmoxClav | EUCAST | 2015 | | | S R | | 7452 |
| ... | ... | ... | ... | ... | ... | ... | ... |
| Vancomycin | EUCAST | 2015 | | | S I R | | 7453 |
| Vancomycin | EUCAST | 2015 | | | S R | | 7453 |

Now store the resulting eligible interpretations in the frequency distirbution by calling `FrequencyDistribution#addDrugReactionEligibleInterpretations`. 

_**Note:** We are not including MDR as a 'hack' right now. This will be removed in future versions._


```
SQLUtil::getConnection.exec( query ).each{ | row |
  fd_key = {
    filters: @filters,
    drug: row[ "drug" ],
    group_by: @group_by,
    group_by_values: {}
  }
  @group_by.each{ | k |
    fd_key[ :group_by_values ][ k ] = row[ getBareColumnName( k ) ]
  }
  fd = @frequency_distributions[ fd_key ]
  if ( fd.nil? )
    throw "Trying to add eligible_interpretations to non existent frequency distribution with key #{ fd_key }"
  end
  fd.addDrugReactionEligibleInterpretations(
    row[ "authority" ],
    row[ "publication" ],
    row[ "delivery_mechanism" ],
    row[ "infection_type" ],
    row[ "eligible_interpretations" ],
    row[ "acceptable_dilutions" ],
    merge_disparate_eligible_interpretations
  ) unless row[ "delivery_mechanism" ].include?( 'MDR' )
}
```

#### Step 6. Normalize and calculate the frequency distributions 

Now that we have all necessary data loaded into the frequency distribution instances, we need to call the `FrequencyDistribution#normalize_and_calculate` method. 

```
# Within IsolateFilter#calculateFrequencyDistributions
@frequency_distributions.each{ | k, fd |
  fd.normalize_and_calculate
}
```

# Frequency Distribution

The frequency distribution is a range of mic results per drug sorted by count per ordinal. The starting and stopping point of the range is determined by finding the lowest <= and > highest tested dilutions.

"How frequently did a dilution occur."

 - Finds the lowest and highest tested dilutions
 - Find the highest 'low' edge and lowest 'high' edge
 - Counts the number of edge occurrences
 - process mic results found between the highest 'low' and lowest 'high'
 - process the normalized data to produce the mic50, mic90 and range per drug and based on filters set in isolate_filter.rb (see Step 1 of isolate filters for example)
 
### Mic50 and Mic90
- The Mic50 is the dilution at which 50% of isolates are susceptable.

- The Mic90 is the dilution at which 90% of isolates are susceptable.

The Mic50 and Mic90 values are calculated at the end of the normalize_and_calculate method when calculating the cumulatives. Cumulatives are created by itterating over the unnormalized data 

 
### Normalization

normalize_and_calculate

Normalization does not occour if no low or high edge exists. Because on the physical panel there is only one "bottom" or "top" on a given panel any values above or below that edge will get rolled into the highest/lowest edge value.

For example only one bottom in any given panel so 1 and <= 1 get rolled into <= 1


#### Crosstabs

Crosstabs is a 2 dimensional representation of normalization for finding outliers in data that could represent errors in the process. It is an array of buckets sorted by drug to mic value counts for 2 drugs.

It gives us the ability to create counts of how many times "drug1 was value 4", "drug2 was 8", etc and graph it on a X,Y chart.

TODO add img of crosstabs generated output


#### Eligible interpretations

TODO

## Understanding how frequency_distribution.rb and Normalization work

TODO