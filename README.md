# Datasets and Scripts to find likely pro-European towns

The idea is to use opinion polls conducted from years 2022 and 2023, as well as the results of the 2019 European Elections, and fit them to city-level social and economic high-level variables as documented by INSEE.

This way we try to obtain the likelihood of a given city (or town) to vote for a pro-EU party or candidate.

For practical reasons, it is often also of interest to have regional and sub-regional ("département") level estimates. We naturally aggregate each city's vote numbers (likelihood * voters) to reach the regional or sub-regional totals.

## Datasets

Local data is extracted topic by topic from the [INSEE Local Statistics Database](https://statistiques-locales.insee.fr/). We use city-level ("commune") data when available, and when not we resort to square-km breakdowns which we then map back to each city (136 cities out of 37k could not be mapped).

Pollsters we took into account were:
* [Elabe](https://elabe.fr/), for example their [January 2022 poll](https://elabe.fr/francais-pfue/) on opinions surrounding the french presidency of the EU ("PFUE" in french)
* [IFOP](https://ifop.com), notably their [end of March 2022 poll](https://www.ifop.com/publication/quelle-france-et-quelle-europe-pour-demain/) about french people's opinion of the EU and their desire for the future of the EU
* [CEVIPOF with OpinionWay](https://www.opinion-way.com/) as part of [SciencePo's "Barometer of political confidence"](https://data.sciencespo.fr/dataset.xhtml?persistentId=doi:10.21410/7E4/9K3VGR) ("Baromètre de la confiance politique") which gives a breakdown along multiple variables of french people's answers to up to 150 questions
* [CEVIPOF with IFOP](https://ifop.com) as part of their recent [series of polls on the 2024 European Elections](https://www.ipsos.com/fr-fr/europeennes-2024-le-rassemblement-national-sinstalle-largement-en-tete-des-intentions-de-vote) which allows us to estimate how interested french people are in the 2024 European Elections

## Accounting for differences in methodology

Most french pollsters use the same main social and economic variables as a way to obtain samples representative of the french population. However, they might report ages and conditional answers based on binned values which can differ from pollster to pollster.

These age bins can also differ from those used by the [french national statistical institute](https://www.insee.fr/) (INSEE).

When this happens we have to estimate how one bin can be re-balanced into different intervals (e.g. the 50-80 age bin can be either cut at the retirement age into 50-64 & 65+ or cut decade-by-decade into 50-69 and 70+).

When we are mapping between two national-level datasets (poll results), it is fine to use [INSEE's age distribution data](https://www.insee.fr/fr/statistiques/2381472#graphique-figure1) to re-balance answer probabilities.

When we have to interpolate from city-level to national-level data (or conversely), we leverage [bayesian methods](https://www.pymc.io/) to infer a more fine-grained conditional distribution.

When a model is trained for this purpose, it will be reported and the parameters and methodology will be made available.

### Within-interval job type distribution

Most pollsters as well as INSEE use the french concept of "CSP" ("Classe Socio-Professionelle") which translates to "Social and Professional Class". This roughly translates to a "job type", in the sense that in breaks down workers in terms of how educated they are, whether they are salaried, whether they are blue or white collar, etc.

One issue is that [INSEE's CSP breakdown given age](https://www.insee.fr/fr/statistiques/2489546#tableau-figure1_radio2) uses a fairly large bin of 25-49 (anyone from 25 to 49 years old). We can use the [age distribution data](https://www.insee.fr/fr/statistiques/2381472#graphique-figure1) to reverse the probabilities with Bayes' Rule, giving the likelihood of [belonging to an age bin given job-type](./data/fr-socio-prof-per-age-binned-conditional-probs.csv).

What you cannot readily do, however, is to further break down the 25-49 bin into the more widely used 25-34, 35-49 bins. However, we have the [town-level age and CSP breakdown](https://statistiques-locales.insee.fr/index.php#c=indicator&f=3&i=rp_cs1_8.pop15p&s=2020&t=A01) together with sub-region level small age bins which we can use to estimate aggregate values via [Multi-Level Regression](https://en.wikipedia.org/wiki/Multilevel_model).

## High-level approach

The idea is to model **two probability distributions**

* the **likelihood someone exhibits pro-european beliefs**, given high-level social and economic factors (age, income, job type)
* the likelihood someone **goes out to vote for the european elections** given those same high-level variables

Additionally, feeding additional data into the model based on input-output bayesian networks trained on [2019 European Election results](https://www.data.gouv.fr/fr/datasets/resultats-des-elections-europeennes-2019/) will be considered.

## Additional comments

We are currently compiling multiple polls into a large CSV file, merging all the sources listed above into a single database. This file will be uploaded when complete.
