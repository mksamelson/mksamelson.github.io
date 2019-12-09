---
title:  "Mass Shootings - the Mother Jones Data"
date: 2019-12-07
tags: [Data Science]
excerpt:  #"Mass Shootings - the Mother Jones Data"
---

## Mass Shootings Are High Profile But Should Not Be Central To The Gun Debate

Gun issues in the United States are contentious and polarizing.  Passion runs high for both proponents and opponents of gun control.  In a social climate in which the consequences for misrepresentation and egregious distortion are few, individuals on both sides speak freely on principal and not from holistic data evaluation.

Gun event data is sparse and the quality limited.  There has been no government-sponsored research (a-la CDC, etc.) for decades.  This is largely due to politics.  That said, there is crowd-sourced data on the internet.  There is no denying that gun related deaths and injuries occur daily in the United States.

Mother Jones Magazine, a liberally-oriented investigative journalism publication, has published numerous articles on "mass shootings."   There is no universally accepted definition of a Mass Shooting in the gun debate.  Generally speaking,  Mass Shootings involve a high level of victims.  Mother Jones defines a Mass Shooting as an event with 4 or more fatalities.

Mother Jones Magazine was challenged by activists in response to assertions made in several articles.  In response, the magazine made it's data publicly available.

This analysis focuses on understand and attempting to elicit statistical significance from the Mother Jones data.

<iframe id="inlineFrameExample"
    title="Inline Frame Example"
    width="300"
    height="300"
    src="https://mksamelson.cartodb.com/viz/f246477c-a4cc-11e5-a726-0e3a376473ab/public_map">
</iframe>

## Mass Shooting Events

The geography of US mass shootings between 1982 and 2015 is illustrated in the map below.

<script src="https://mksamelson.carto.com/viz/f246477c-a4cc-11e5-a726-0e3a376473ab/embed_map"></script>

The map highlights the 73 events contained comprising the Mother Jones data.  The size of the event pertains to number of total victims.

Dragging your cursor over an event will provide event summary information.

The incidence of a shooting and number of victims have no relation to geography.

The following chart's highlight the leading states in terms of events and victims:

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/incidentsvictimsbystate.jpeg" alt="">

California leads the Nation in both mass shooting incidents and victims with 12 out of 73 events and approximately 200 victims.  Florida and Washington follow distantly in terms of incidents with 6 each.  Texas and Colorado follow in terms of victims with 110 each.

## Victims:  Fatalities and Wounded

Any act of random violence is an atrocity.  However, news media and interest groups often play on details to imply trends and characteristics of shooting events that just aren't there.  While most outlets attempt to shock us with the magnitude of those wounded or killed in a particular event, big numbers are not consistent across all (or even a majority) of events.  See the figures below.

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/totalvictimslist.jpeg" alt="">

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/FatalitiesHist.jpeg" alt="">

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/WoundedHist.jpeg" alt="">

30 events - just under half of total events - had less then total victims.  Just over 50 events had between 5 and 10 fatalities while just over 40 events had 0 to 5 injured.  Overall, events with large numbers of wounded and fatalities were indeed a minority of events.  While tragic and high profile - they just didn't happen all that often.

## Perpetrator Characteristics

Perpetrator sex, age, and race are shown in the figures below.

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/AgeSexEthnicity.jpeg" alt="">

Mass shooting perpetrators are most often male, most often white, and are most often 40 to 45 years old.  It is *critically* important to realize that these characteristics are independent.  In other words, the hypothetical "typical" perpetrator (if there is such a thing) is not white and male and aged 40 to 45.

Perpetrators are distinctly male.  The figure above reveals males conduct incidents in stunning proportion.  In fact, 70 of 73 incidents were perpetrated by single male actors (one event was performed by a male/female team and two events by lone female assailants).

A sizable proportion of perpetrators are white.  46 of 73 events were perpetrated by actors who were white.  This is still a standout figure.  Whites were trailed by blacks who conducted 12 events and by Asians with 6 events.  Latinos, Native Americans and Other ethnicities each accounted for under 5 events.

Males are the actors in virtually all events.  Yet how does race and age fit into the equation?  See the figures below.

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/MaleBarAgeRace.png" alt="">

While males between the ages of 40 and 45 are the largest group of perpetrators, white males constitute only about half of the group.  The figure above illustrates that white males, by virtue of their number, are heavily represented in most age groups.

A density plot of white males conditioned on age reveals the likelihood that a male of a particular ethnicity falls in a particular age group.

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/WhiteMaleAgeDensity.jpeg" alt="">

While there are many white male perpetrators in the 40 to 45 year age range, white male perpetrators are more likely to be in their early 20s.

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/OverlappingEthnicityAgeDensity.jpeg" alt="">

Density functions by race conditioned upon age suggest that differences exist in age among perpetrators of different race. The density function for white males, illustrated earlier and shown in the figure above in pink, is readily visible with a peak for individuals in their early 20s and a secondary peak for males in their early 40s.  Black men who perpetrate these shootings are most often in their late 30s.  Other races (Latino, Native American, Asian, and "other") also tend to most often be in their late 30s.

The densities show the likelihood of a perpetrator of a particular race being of a certain age.  While it provides insight into historical data, it is not enough to assert that race and age are statistically associated.

A mosaic plot with a Chi Squared test of observed vs expected values provides a formalized, rigorous statistical assessment as to the relationship of race and age among male perpetrators.  In short, a mosaic plot enables us to visually compare categorical variables (race and age group are categorical variables - age itself is a continuous) and determine statistically if our expectations for the group combinations is in line with the counts we actually observe.

The mosaic plot below illustrates our data in slightly amended groups (grouping the data as shown was necessary for a meaningful test because of a small sample size).  Grey boxes indicate that observations are in line with expectations and that knowing age will not provide insight into whether a perpetrator is white (and vice versa).  Blue or red colored boxes indicate deviations from expectations.   Blue indicates greater than expected values and red less than expected values.   In short, a blue or red box for one or more of the groups would indicate that knowing the age of a perpetrator would provide insight into that person's race or vice versa.

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/MosaicZoom.jpg" alt="">

The grey coloring of all boxes and the insignificant Chi-Squared p-value of .3114 shown under the legend tell us that we are unable to gain insight into perpetrator race knowing age group (i.e., the variables are independent).

## Weapon Legality and Mental Illness

Perpetrators obtained weapons legally and displayed "irregular behavior" prior to incidents in many events.

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/MentalHealth.jpeg" alt="">

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/LegalWeapons.jpeg" alt="">

The preponderance of an individual to obtain legal weapons and display unusual behavior in close to 60 percent of event.   There is certainly much to debate around restrictions concerning the legal purchase of guns.  There is nothing in the data that permits meaningful elaboration on this issue.

The fact that others saw "irregular" behavior prior to the shooting in nearly 60 percent of cases is another matter.  Erratic behavior that may result in harm to others is something that can readily be acted upon.  It is usually pretty clear when individuals act in depressed or despondent manners.  Most people can tell that "something is wrong".  Still, most people fail to take action on an individual that may cause harm to themselves or others.  Part of the reluctance may be fear of being wrong or involving themselves in someone else's business.  I think the failure to act in these cases is selfish and cowardly.  There is little doubt in my mind that at least some of these incidents could have been avoided if someone who "saw something" said something to law enforcement or mental health experts.

## The Worst Events

I have purposefully avoided mentioning specific events until this point.  My intention was to have readers focus on events from a statistical and analytical standpoint as opposed to becoming blinded by sensational elements of particular events.  That said, no discussion of mass shootings would be complete without a list of the most egregious events.

<img src="{{site.url}}{{ site.baseurl }}/images/massshootings/Top5List-1024x183.png" alt="">

3 0f the 5 events are quite recent, occurring since 2009.  All events dominated national media coverage when they occurred and some continue to do so.  It is notable that these events are all unusual even for mass shootings.  Note the fatality, wounded, and victims numbers and compare with the histograms above.  It is clear that the number of casualties far exceed those of the preponderance of events.  The point is that shootings such as these are a genuine rarity.  While they highlight some of the most atrocious conduct one can imagine they are extreme events.  They are not indicative of daily gun related events that occur in the United States nor are they common for their class of events.  Media and activists focusing on these events and implying they are commonplace and central to gun control and rights issues are incorrect, uninformed and likely to be shifting emphasis on these events for their own particular objectives.

## Conclusion

Mass shooting are atrocious.  No question about it.  However, scientifically evaluating the Mother Jones Mass Shooting Data yields the following insights:

* There have been only 73 events in which 4 or more individuals have been killed since 1982.

* The most prevalent events have fewer than 10 victims and few than 10 fatalities.

* Total victims constitute an extreme minority of total gun incident victims.
Weapons in most events were obtained legally.

* Most perpetrators displayed "unusual" or "erratic" or "disturbing" behavior prior to their event.

* Mass shootings with substantial media coverage tend to be those with high victim count.  These are generally outlying events within the mass shooting data set.

In regard to perpetrators:

* Most are male.

* Most are aged between 40 and 45 years old.

* Most are white.

* We are unable to gain any statistically-based predictive insight to associate age and race within the male sex category.

Mass shootings, while terrible, constitute a small proportion of gun related events.  The worse the event, the greater the media scrutiny and the hyped the coverage.  Furthermore, most events involve legally obtained weapons and prior indications of erratic behavior.  Mass shootings should certainly an area of focus for legal and medical professionals.  However, the data simply does not constitute the incorporation of mass shooting data in the broader discussion of gun rights/gun control debate.  Political, media, and activist incorporation simply takes events out of context, "muddies the waters" and makes meaningful gun-related discussions more complicated.
