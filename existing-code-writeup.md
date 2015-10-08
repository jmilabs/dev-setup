# Existing Code Write Up

**ToDo**
- [ ] Identify (filter) which isolates should be included in this execution
- [ ] Demonstrate how the generated sql enforces those filters for attributes that come from each of the possible tables:  isolates, organisms, sites, isolate_attributes:
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

## Minimum Inhibitory Concentration 

In microbiology, **minimum inhibitory concentration (MIC)** is the lowest concentration of an antimicrobial that will inhibit the visible growth of a microorganism after overnight incubation. At **JMI Labs**, we several different kinds of tests to identify the MIC for a particular drug and organism combination. These tests include:
  * MIC Panel
  * E-Test
  * Auger Panel 
  * Disk Diffusion

In order to perform this testing, **JMI Labs** works with hospitals around the world to get many different samples of an organism, these are called isolates. Within our labs, we have tens of thousands of different organisms each with unique properties. We use this wide range of isolates to provide our clients with a normalized frequency distribution of all MIC results for a particular drug / organism combination. 

** MIC Panel 
![MIC Panel](/images/mic_panel.png)

## Step 1: Determine which isolates to include in the report