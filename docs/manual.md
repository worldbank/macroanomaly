<!-- Source of truth for the Quarto book is docs/index.qmd and docs/chapters/*.qmd.
     This file is kept as the original single-file export; edit the chapter files instead. -->

Using the macroanomaly R Package and an AI-Enabled Anomaly Classifier

DRAFT

Version 0.0.5

2026-06-26

World Bank Group

Office of the Chief Statistician and Development Data Group

**[Acknowledgments]{.smallcaps}**

The document was produced by Olivier Dupriez (Deputy Chief
Statistician), Aivin V. Solatorio (Program Manager), and Luciano
Perfetti Villa (Consultant). David Newhouse participated in the
development of the macroanomaly package.

Feedback and input was received from colleagues from the World Bank
Development Data Group, including Yan Bai, Thijs Benschop, Daniel
Boller, Christoph Lakner, Hiroko Maeda, Daniel Gerszon Mahler, Joao
Miguel Falcao Pinto Da Silva, Jaime Estuardo Fernandez Romero, Evis
Rucaj, Emi Suzuki, and Matthew Welch, and from the Data Quality Working
Group in the Indicator and Statistics Unit.

**[License]{.smallcaps}**

![Downloads - Creative
Commons](images/media/image1.png){width="0.8958333333333334in"
height="0.3277777777777778in"}This document was created by the World
Bank Group. It is shared under a CC BY 4. 0 Attribution license.

**[Use of AI]{.smallcaps}**

This document was produced with the assistance of AI---specifically
Claude Sonnet 4.6 by Anthropic. All AI-generated content was reviewed
and validated by the authors before inclusion.

**[Maintenance and Tools]{.smallcaps}**

> ![GitHub - Apps on Google
> Play](images/media/image2.jpeg){width="0.5680555555555555in"
> height="0.5680555555555555in"}This document and the related tools are
> maintained on GitHub.
>
> The macroanomaly package and documentation are available at:
> <https://github.com/worldbank/macroanomaly>
>
> The AI tools for anomaly classification and explanation are available
> at: <https://github.com/worldbank/ai4data>


# Introduction

National statistical organizations and international organizations
compile and disseminate time series data on development indicators for
use by policymakers, researchers, businesses, and the general public.
Ensuring the quality of these series is essential for credible evidence
and sound decision-making.

Data quality encompasses several dimensions: relevance, accuracy,
timeliness, comparability, coherence, accessibility, and
interpretability. Quality assurance and quality control should be
applied across the entire data lifecycle --- from design and collection
through compilation, derivation, validation, documentation, and
dissemination --- with appropriate metrics, assessment methods, and
corrective actions at each stage.

This document focuses on one specific aspect of data quality control:
the detection of anomalies in time series, defined as observations ---
or short sequences of observations --- that depart markedly from
expectations. Before publishing time series, data producers and
distributors should systematically screen for anomalies. Anomaly
detection is a diagnostic input within a broader, human-governed data
quality process. Its purpose is to identify observations that warrant
review, not to determine automatically that a value is wrong or should
be changed. An anomaly flag is therefore a trigger for review, not a
correction decision. In practice, flagged observations may reflect data
errors, real-world shocks, methodological changes, structural breaks, or
temporary departures from historical patterns. Some cases may remain
uncertain even after expert review. For this reason, anomaly detection
should be implemented as part of a governed workflow that separates four
steps: statistical screening, expert review, classification, and
treatment through approved data revision procedures. The document covers
how artificial intelligence (AI) can assist in classifying and resolving
detected anomalies.

Anomaly detection is primarily the responsibility of data curators and
publishers. However, because it is not always performed consistently at
the source, data analysts are also encouraged to apply these procedures
before using time series in their own work.

The document is organized as follows. Section 2 situates anomaly
detection within the broader framework of data quality assurance.
Section 3 defines \"anomaly\" in the context of time series and
describes key distinctions --- point versus collective anomalies,
univariate versus multivariate detection, and errors versus legitimate
outliers. Section 4 sets out the scope and objectives of the document.
Section 5 introduces the R package macroanomaly. Section 6 reviews
selected anomaly detection methods. Section 7 explains how to apply
these methods using macroanomaly, including dataset preparation and
practical code examples. Section 8 presents an AI-assisted tool for
classifying detected anomalies and generating explanations. Section 9
offers recommendations for treating anomalies once identified.

This guide presents a methodological framework, not a software
requirement. Although examples may use the macroanomaly R package, the
principles described here---preprocessing, threshold calibration,
univariate and multivariate screening, expert review, and
classification---can be implemented in any suitable statistical
environment. Package-specific code should therefore be treated as
illustrative rather than normative.

This document was developed to support implementation of the World Bank
Group Development Data Quality Policy. Its production was made possible
in part by funding from a WBG Innovation Award (P514969 --- Leveraging
MCP for Responsible AI).

# Anomaly Detection in the Data Quality Assurance Context

The table below, adapted from work by Thijs Benschop, Daniel Boller, and
Matthew Welch, situates anomaly detection within the broader framework
of data quality assurance and quality control for time series. The main
point is that anomaly detection is only one of several diagnostic
approaches used to assess data quality. Some data-quality issues are
best identified through metadata, source documentation, and
production-process controls, while others are primarily statistical and
therefore well suited to anomaly detection methods.

This distinction matters because an anomalous observation is not
necessarily an error. It may reflect a genuine economic shock, a
methodological break, a reporting gap, or a processing mistake. Anomaly
detection can help identify observations or periods that merit review,
but it does not by itself determine the cause, nor does it decide
whether a value should be corrected, retained, revised, or annotated.
Those decisions belong within a broader governance process that includes
metadata review, reproducibility checks, and subject-matter expertise.

  -----------------------------------------------------------------------------------------
  **Type of data   **Typical          **Primary         **Role of anomaly **Typical
  quality issue**  manifestations**   diagnostic        detection**       governance
                                      tools**                             response**
  ---------------- ------------------ ----------------- ----------------- -----------------
  Coverage or      Missing years,     Completeness      **Limited**.      Follow up with
  completeness     delayed reporting, checks, source    Anomaly detection the source,
  issues           partial reporting, monitoring,       may identify gaps document the gap,
                   discontinuities    metadata review,  or abrupt         and distinguish
                   caused by          comparison with   absences, but it  missing data from
                   non-reporting      expected          cannot determine  true zeros or
                                      reporting         why data are      discontinued
                                      calendars         missing.          series.

  Methodological   Rebasing,          Metadata review,  **Indirect**.     Annotate the
  changes          redefinitions,     revision          Anomaly detection series, apply
                   changes in         analysis, break   may flag a        revision and
                   classification,    analysis,         structural break  versioning
                   scope, units, or   consultation with or level shift,   policies, and
                   source methodology source agencies   but it cannot     document
                                                        determine whether comparability
                                                        the change is     limitations.
                                                        legitimate or     
                                                        methodological.   

  Processing or    Coding mistakes,   Reproducibility   **Useful as an    Correct the error
  transformation   unit conversion    checks, code      alert**. Anomaly  through the
  errors           errors,            review, pipeline  detection can     governed
                   aggregation        validation,       help surface      production
                   mistakes,          input-output      suspicious values process, document
                   duplicate records, consistency       produced by the   the fix, and
                   incorrect seasonal checks            pipeline, but     strengthen
                   adjustment or                        root-cause        preventive
                   deflation                            diagnosis         controls.
                                                        requires process  
                                                        controls.         

  Unexpected       Extreme values,    Statistical       **Primary**. This Submit flagged
  statistical      sudden jumps,      diagnostics,      is the setting in cases for expert
  behavior         unusual            algorithmic       which anomaly     review, then
                   volatility,        anomaly           detection plays   retain, annotate,
                   out-of-pattern     detection,        its central role  revise, or
                   movements relative peer-series       by identifying    escalate
                   to historical      comparison,       observations or   depending on the
                   behavior           residual analysis segments that     findings.
                                                        deviate from      
                                                        expected          
                                                        patterns.         

  Real economic or Crises, conflicts, Contextual        **Supportive but  Retain the values
  social shocks    disasters, policy  analysis,         not decisive.**   if validated, and
                   changes, commodity external          Anomaly detection document the
                   shocks, pandemics  validation, event may flag the      event and its
                                      timelines, domain event, but it     implications for
                                      expertise         cannot determine  interpretation.
                                                        whether the       
                                                        observation is    
                                                        valid or          
                                                        substantively     
                                                        meaningful.       
  -----------------------------------------------------------------------------------------

# Defining Anomalies in Time Series

An anomaly in a time series is an observation --- or a short sequence of
observations --- that departs markedly from what the series\' historical
behavior would lead one to expect.

**An anomaly is not an absolute property of a data point.** Whether an
observation appears anomalous depends on the time span, comparability
spell, and contextual information used in detection. An observation that
appears unusual in a short window may appear consistent in a longer
historical series. Users should also distinguish between temporary
collective anomalies, which affect a short sequence of observations, and
structural breaks, which reflect a more persistent change in level,
trend, or data-generating process. This distinction matters because
temporary anomalies may call for case review, while structural breaks
may require a change in model, metadata, or interpretation rather than
data correction.

Anomalies can be categorized along two independent dimensions: whether
they affect one or multiple observations (point vs. collective), and
whether they are detected within a single series or across several
(univariate vs. multivariate). Regardless of type, an anomaly may
reflect either a genuine real-world event or a data quality problem.
This distinction --- legitimate outlier versus error --- has direct
implications for how the anomaly should be treated.

**Point vs. collective anomalies**

A point anomaly is a single observation that deviates sharply from the
expected level, trend, or seasonal pattern. For example, the number of
international air passengers dropped abruptly in 2020 --- a legitimate
reflection of COVID-19 travel restrictions. By contrast, a life
expectancy series showing a value of 805 years for a given country and
year is almost certainly a data entry error, likely caused by a
misplaced decimal point.

A collective anomaly is a contiguous block of observations exhibiting a
sustained and unusual change in pattern. For example, a government
revenue series might show a persistent upward level shift beginning in
2020 and continuing in subsequent years --- a pattern that could
legitimately reflect a major tax reform rather than a data problem.

**Univariate vs. multivariate anomalies**

A univariate anomaly is one detected within a single indicator series,
assessed on its own terms relative to its own history.

A multivariate anomaly arises from an unusual combination of values or
co-movements across indicators or geographies, even when each individual
series appears unremarkable in isolation. For example, if a country\'s
employment rate rises sharply while GDP contracts and electricity
consumption remains flat in the same year, no single series may look
anomalous on its own --- but the combination is internally inconsistent
and warrants closer scrutiny.

Multivariate methods are most useful when the main concern is not an
isolated spike but an inconsistency across related variables. In this
guide, 'multivariate' refers to methods that evaluate an observation
using more than one feature, such as multiple indicators, lagged values,
or derived contextual variables. A multivariate anomaly may arise when a
value appears plausible on its own but implausible in context---for
example, when poverty declines sharply during a period of economic
contraction. In such cases, the flag should be interpreted as a signal
of possible incoherence requiring review, not as proof of error.

**Legitimate outliers vs. errors**

Legitimate outliers are anomalies that reflect real events: economic
shocks, natural disasters, policy changes, statistical
reclassifications, series rebasings, or definitional revisions. Because
they carry genuine information, legitimate outliers should be retained
in the data. They should, however, be clearly explained and documented
in metadata to support correct interpretation by users and analytical
systems alike.

Errors are anomalies caused by mistakes introduced at any stage of the
data production process --- data entry, derivation, or modeling. Errors
should be corrected wherever possible. When correction is not feasible,
the affected values should be suppressed, with a clear justification
recorded.

# Scope and Objective

This document presents a practical set of statistical and machine
learning methods for automated, unsupervised detection of both point and
collective anomalies in time series. The methods are designed with
development statistics in mind: series in this domain are often short,
may exhibit trends and seasonality, and frequently contain gaps. A
deliberate design goal is that these methods can be applied without
country-specific or subject matter expertise, making them suitable for
systematic, large-scale quality screening.

The document does not cover rule-based checks or indicator-specific
consistency tests --- such as range checks, accounting identities, or
cross-series plausibility checks. These are valuable and complementary
tools for quality control, and data producers are strongly encouraged to
implement them. However, because they depend on domain-defined rules and
expert judgment, they fall outside the scope of what this document
addresses.

Within these boundaries, the document pursues two practical objectives.
The first is to demonstrate how to implement anomaly detection using
macroanomaly, a dedicated R package (R Core Team 2025). The second is to
show how detected anomalies can be classified as errors or legitimate
outliers, and how explanations for those classifications can be
generated using an AI-assisted workflow.

Improving the accuracy, interpretability, and trustworthiness of time
series data goes beyond internal quality-control procedures. It also
requires a systematic feedback loop to data producers, and sustained
collaboration with national statistical offices and other partner
organizations. Because many World Bank series originate from external
agencies, the quality of the Bank\'s data is ultimately inseparable from
the quality of those upstream sources. The tools and methods described
in this document are openly accessible to support this broader agenda:
they can be used for internal quality screening, to inform corrections
at the source, and to promote the wider adoption of sound anomaly
detection practices across partner systems.

# The macroanomaly R Package 

macroanomaly is an R package designed to support the detection of
anomalies in social and economic time series, with a focus on the
indicators central to the World Bank Group\'s development data work. It
was developed and is maintained by the Office of the Chief Statistician
and the Development Data Group at the World Bank Group (WBG), as part of
the broader implementation of the WBG Development Data Quality Policy.
The package is available on GitHub under an MIT License with an IGO
rider (see <https://github.com/worldbank/macroanomaly>).

Rather than introducing entirely new algorithms, macroanomaly draws on
functions from well-established R packages and organizes them into a
coherent, accessible workflow tailored to the characteristics of
development data. Many existing anomaly detection tools are designed for
contexts --- such as industrial process monitoring or biomedical signal
analysis --- where data are abundant, regularly spaced, and free of
seasonal distortions. Social and economic time series rarely meet these
conditions: annual series are often short (sometimes only a few dozen
observations), may contain gaps from missing values, and frequently
exhibit trends or seasonal patterns that must be accounted for before
anomalies can be reliably identified. macroanomaly is built to handle
these conditions directly.

The package implements a three-step workflow. First, it prepares the
data by imputing missing values in a statistically neutral manner and
transforming each series to achieve stationarity --- that is, removing
trends and seasonal components so that anomalies can be detected against
a stable baseline. Second, it applies a selection of statistical and
machine learning methods to flag potential anomalies. Third, it produces
outputs to support review and reporting: a CSV file in which flagged
observations are annotated with scores or metrics for each detection
method applied, and a set of plots to help users visualize the results.
The package takes as input a dataset in long format, making it
straightforward to integrate into standard data pipelines.

This document describes the anomaly detection methods included in
Version 1.1.0. The package is under active development, and future
versions may introduce additional methods. The GitHub repository always
holds the latest release, a complete changelog, and a vignette
documenting each function. Users are encouraged to submit feedback,
suggestions, and issue reports through the repository\'s Issues section.

# Anomaly Detection Methods 

This section describes the statistical and machine learning methods
included in macroanomaly. Each method was selected for its suitability
to the characteristics of social and economic time series: short series
with limited observations, potential gaps from missing values, and the
presence of trends or seasonal patterns. The methods vary in their
underlying assumptions and sensitivity to different types of anomalies;
using them in combination, as the package is designed to facilitate,
produces more robust results than relying on any single approach. The
following subsections explain how each method works and what kinds of
anomalies it is best equipped to detect.

## Z-score

The Z-score is a straightforward and widely used method for detecting
univariate point anomalies. It works by standardizing each observation
relative to the series\' mean and standard deviation, producing a
measure of how far that observation departs from the average. The
formula is:

$$z = \frac{(x - \mu)}{\sigma}$$

where *x* is the observed value, *μ* is the series mean, and *σ* is the
standard deviation. The resulting score expresses the distance of each
observation from the mean in units of standard deviations. A positive
Z-score indicates a value above the mean; a negative Z-score indicates a
value below it.

The method assumes that the values in the series --- or in a relevant
segment of it --- are approximately normally distributed. Under this
assumption, observations whose absolute Z-score exceeds a predefined
threshold are flagged as anomalies. The choice of threshold directly
controls the method\'s sensitivity: a lower threshold flags more
observations but risks more false positives, while a higher threshold
flags only the most extreme deviations.

Three threshold values are commonly used, grounded in the properties of
the normal distribution:

- **3 (strict)**: Flags only the most extreme outliers. Under a normal
  distribution, approximately 99.7% of observations fall within three
  standard deviations of the mean, so only very rare values are flagged.

- **2.5 (moderate)**: A widely used intermediate setting that balances
  sensitivity and specificity.

- **2 (sensitive)**: Flags a broader set of potential anomalies, since
  approximately 95% of observations in a normal distribution fall within
  two standard deviations.

In practice, macroanomaly computes a Z-score for each observation and
ranks them by their absolute value, allowing users to identify and
prioritize the most significant deviations.

Detection thresholds should be calibrated, not treated as fixed
defaults. Calibration should consider at least three factors: the type
of indicator, the length and comparability of the series, and the team's
review capacity. Lower thresholds increase sensitivity but also increase
the number of false positives and the burden of manual review. Higher
thresholds reduce review burden but increase the risk of missing
substantive problems. Users should begin with a documented default, test
how many observations are flagged under alternative settings, and choose
a threshold that produces a manageable review workload while still
surfacing meaningful cases. For short or fragmented series, automated
flags should be treated primarily as a prioritization tool for manual
review rather than as strong evidence on their own.

**A robust alternative: the modified Z-score**

A well-known limitation of the classical Z-score is that both the mean
and standard deviation are sensitive to extreme values --- precisely the
values the method is trying to detect. A single outlier can inflate the
standard deviation and shift the mean toward it, making other anomalies
harder to identify.

The modified Z-score addresses this by replacing the mean with the
median and the standard deviation with the Median Absolute Deviation
(MAD). The MAD is defined as:

$$\text{MAD} = \text{median}(\left| x_{i} - \text{median}(x) \right|)$$

The modified Z-score is then computed as:

$$\text{Robust}\ z \approx 0.6745 \times (x_{i} - \text{median}(x))/\text{MAD}$$

The constant 0.6745 is a scaling factor that makes the MAD comparable to
the standard deviation for normally distributed data, so that modified
Z-scores remain interpretable on the same scale as classical ones.
Because the median and MAD are resistant to the influence of extreme
values, the modified Z-score provides a more reliable measure of
deviation in datasets that already contain outliers.

**Implementation of Z-score in macroanomaly**

macroanomaly implements the z-score method internally when applying the
normalize() function.

*Parameter:*

- **Threshold**: the absolute Z-score above which an observation is
  flagged as an anomaly. The default value is 3, but users can adjust
  this based on the series characteristics or the desired level of
  sensitivity.

*Output fields:*

- **Flag**: a binary indicator marking whether the observation is
  flagged as an anomaly.

- **zscore** and **abs_zscore**: the signed and absolute Z-score for
  each observation. A higher absolute value indicates a greater
  deviation from expectations; the sign indicates the direction
  (positive means higher than expected, negative means lower).

**Limitations**

Both the classical and modified Z-score methods perform best on
stationary series that are approximately normally distributed. If a
series exhibits a clear trend or seasonal pattern, it should be
transformed to stationarity before these methods are applied --- a step
that macroanomaly handles automatically in its data preparation stage.
Additionally, both methods become less reliable on very short series,
where estimates of central tendency and dispersion (whether mean and
standard deviation, or median and MAD) are inherently noisy.

## Collective and Point Anomalies (CAPA)

CAPA is a univariate method capable of detecting both point and
collective anomalies within the same framework --- a distinctive feature
that sets it apart from methods designed for only one type. The
algorithm approaches detection as an optimization problem: it searches
over all possible configurations of anomalous segments and isolated
points, selecting the combination that best improves the fit to the data
relative to the cost of introducing additional anomalies. This is
achieved through penalized likelihood optimization, solved efficiently
using dynamic programming with pruning, which gives the method
near-linear computational performance even on long series.

By default, macroanomaly applies robust standardization --- using the
median and MAD rather than the mean and standard deviation --- when
estimating baseline behavior. This reduces the influence of extreme
values on the reference distribution, making the method more reliable in
series that already contain anomalies.

**Implementation in macroanomaly**

macroanomaly implements the capa method from the anomaly package.

*Parameters:*

- **Minimum segment length**: the shortest sequence of consecutive
  observations that can be classified as a collective anomaly. The
  default value is 2, meaning at least two consecutive observations must
  be involved for a segment to be flagged as collective.

- **Anomaly type**: the type of change that CAPA is configured to
  detect. The default is \"meanvar\", which flags collective anomalies
  characterized by simultaneous shifts in both the mean and variance of
  the series. Alternative options are \"mean\", which detects shifts in
  the mean only, and \"robustmean\", which focuses on mean shifts while
  explicitly accounting for the presence of point outliers in the
  series.

*Output fields:*

- **Flag**: a binary indicator marking whether an observation has been
  flagged as an anomaly, either as a point anomaly or as part of a
  collective anomaly segment.

- **capa_strength**: the magnitude of the anomaly, expressed as the
  absolute deviation from the local robust mean in Z-score units. This
  field is populated only for flagged point anomalies; it is NA for
  non-anomalous observations and for observations that are part of a
  collective segment but do not independently qualify as point
  anomalies.

**Limitations**

CAPA operates on a single series at a time and is therefore a univariate
method. Like the Z-score methods, it performs best on stationary series;
macroanomaly handles the necessary preprocessing automatically. The
choice of penalty parameters and minimum segment length can influence
results meaningfully, particularly on short series, and users are
encouraged to review outputs across a range of settings when working
with unusual or sparse data.

## Time Series Outlier (TSOutlier)

TSOutlier is a univariate method for detecting point anomalies based on
the interquartile range (IQR). Rather than applying the IQR to raw
values, the method first decomposes the series into its trend, seasonal,
and remainder components, then applies outlier detection to the
remainder alone. This ensures that observations are assessed relative to
the local behavior of the series, stripped of systematic patterns that
would otherwise obscure genuine anomalies. The implementation follows
the approach provided in the forecast R package.

The detection logic proceeds as follows. For a given window of
observations (or the full series), the first and third quartiles --- Q1
and Q3 --- are computed, and the interquartile range is defined as IQR =
Q3 − Q1. Lower and upper bounds are then set at Q1 − k × IQR and Q3 + k
× IQR respectively. Any observation falling outside these bounds is
flagged as an anomaly.

A key advantage of this approach is that the IQR is inherently robust:
unlike the mean and standard deviation, quartiles are not distorted by
the extreme values they are meant to detect. The method is also
nonparametric, requiring no assumptions about the underlying
distribution of the data.

**Implementation in macroanomaly**

macroanomaly implements the tsoutlier algorithm from the forecast
package natively.

*Parameters:*

- **Multiplier k (**.threshold**)**: determines how far an observation
  must fall from the interquartile range to be flagged. A value of 1.5
  is conventional for detecting mild outliers; a value of 3.0 flags only
  more extreme deviations. The default in macroanomaly is 3.0.

*Output fields:*

- **Flag**: a binary indicator marking whether the observation has been
  flagged as an anomaly.

**Limitations**

The sensitivity of TSOutlier depends heavily on the choice of window
length and multiplier k, and users should consider adjusting these
parameters when working with series of unusual length or volatility.
Because the method operates on the remainder after trend and seasonal
decomposition --- a step handled automatically by macroanomaly --- it is
less prone to false positives than applying IQR-based detection directly
to raw values. However, in very short series, decomposition may be
unreliable, and results should be interpreted with caution.

## Hampel Filter 

The Hampel filter is a univariate method for detecting point anomalies
using a sliding window approach. For each observation in the series, it
computes a robust Z-score based on the local median and MAD within a
surrounding window, then flags observations whose deviation from the
local center exceeds a defined threshold. More formally, for each time
point *t*:

1.  A window of neighboring observations centered on *x_t* is selected.

2.  The window median m_t and the median absolute deviation MAD_t =
    median(\|x_i − m_t\|) are computed.

3.  A robust Z-score is calculated as *z_t = \|x_t − m_t\|* / (1.4826 ×
    MAD_t), where the constant 1.4826 makes the MAD comparable to the
    standard deviation under a normal distribution.

4.  The observation is flagged as an anomaly if *z_t* exceeds a
    threshold *t0*, commonly set to 3.

Because detection relies on the median and MAD rather than the mean and
standard deviation, the method is inherently robust: extreme values in
the window have little influence on the reference statistics used to
assess them. The sliding window design also makes detection local,
allowing the method to adapt to series where the typical level or
variability changes over time.

**Controlling false positives**

A limitation of the standard Hampel filter is its tendency to produce
false positives, particularly in series with moderate variability. As
shown in Figure 1, where red dots mark flagged anomalies, values that
deviate only slightly from the trend, such as the 1990 observation, may
be incorrectly flagged.

![- False positives produced by the standard Hampel
filter](images/media/image3.jpg){width="6.502047244094488in"
height="3.707416885389326in"}

![- Result after applying the secondary
filter](images/media/image4.jpg){width="6.5735586176727905in"
height="3.748193350831146in"}

**Implementation in macroanomaly**

macroanomaly implements (and adapts) the hampel method from the pracma
package.

*Parameters:*

- **Half-window size**: the number of observations on each side of *x_t*
  included in the local window. A larger window captures broader context
  but may dilute local sensitivity; a smaller window is more responsive
  to sharp local changes. The default is 5.

- **Threshold *t0***: the robust Z-score above which an observation is
  flagged. Higher values reduce sensitivity and false positives; lower
  values flag more observations. The default is 5, reflecting a
  conservative setting appropriate for most development data series.

- **Secondary filter**: an on/off toggle enabling the false-positive
  reduction step described above. Off by default.

- **Secondary filter bandwidth**: the number of standard deviations from
  the trimmed mean within which flagged observations are unflagged. Only
  applies when the secondary filter is enabled. The default is 1.

- **Input column**: determines whether the filter is applied to the raw
  series values or to the Z-score column. Defaults to the Z-score
  column, consistent with macroanomaly\'s standardized preprocessing
  pipeline.

*Output fields:*

- **outlier_indicator**: a binary 0/1 flag indicating whether the
  observation has been identified as an anomaly.

- **hampel_replacements**: a corrected version of the series in which
  flagged observations are replaced by the local window median. This can
  be used as a cleaned series for downstream analysis, though users
  should exercise judgment before treating replacements as authoritative
  corrections.

**Limitations**

The Hampel filter\'s performance depends on the choice of window size
and threshold. A window that is too narrow may miss anomalies embedded
in locally volatile segments; one that is too wide may smooth over
genuine deviations. Like the other methods in this document, the filter
performs best on stationary series --- macroanomaly applies trend and
seasonal adjustment in its preprocessing stage before this method is
run. On very short series, local window estimates may be unstable, and
results should be interpreted accordingly.

## Outlier Trees

Outlier Trees is a multivariate method for detecting point anomalies.
Unlike the univariate methods described in previous sections, it
assesses each observation in the context of multiple variables
simultaneously, making it capable of identifying anomalies that arise
from unusual combinations of values rather than extreme values in any
single series.

The method works by building a decision-tree-like structure whose
objective is not prediction but the identification of rarity. At each
node, the algorithm selects splits that isolate small, low-density
regions of the feature space --- subsets of observations that are
unusually sparse given the values of the features used to define them.
The leaves of the tree correspond to narrow, well-defined segments
containing very few observations. An observation falling into such a
leaf is flagged as an outlier, and the path of splits leading to that
leaf constitutes a human-readable rule explaining why.

Internally, the method uses robust statistics --- such as median- and
quantile-based thresholds --- to identify rare ranges for numeric
features, reducing sensitivity to the very extremes it is designed to
detect. Each observation receives a score reflecting the rarity and
strength of the rule it satisfies; higher scores indicate stronger
anomalies. A notable advantage of this approach is interpretability:
unlike many anomaly detection algorithms, Outlier Trees can explain each
flagged observation in terms of concrete, auditable conditions on the
input features.

The method is well-suited to high-dimensional data and imposes no strong
distributional assumptions. However, its performance depends on the
quality and relevance of the input features: strongly correlated or
highly noisy features can distort the splitting process. The method may
also flag only a small number of anomalies in datasets with moderate
variability, and tuning of key parameters is often necessary to achieve
appropriate sensitivity.

**Implementation in macroanomaly**

macroanomaly implements the outliertree algorithm from the outliertree
package.

*Parameters:*

- **Columns used for detection**: the subset of features passed to the
  algorithm. Defaults to all available columns in the dataset.

- **Input column**: determines whether the method is applied to the raw
  series values or to the Z-score column. Defaults to the Z-score
  column, consistent with macroanomaly\'s standardized preprocessing
  pipeline.

- **Score threshold**: the minimum score above which an observation is
  flagged as an anomaly. Scores range from 0 to 1, with higher values
  indicating greater anomalousness. The default threshold is 0.5.

- **Threads**: the number of CPU threads used for parallel computation.
  The default is 2.

- Additional arguments: using \... the user can pass additional
  arguments to the outliertree function, if needed.

*Output fields:*

- **outlier_score**: a continuous score between 0 and 1 produced by the
  algorithm, where higher values indicate a greater degree of
  anomalousness. This field can be used to rank observations by severity
  rather than applying a binary cutoff.

- **outlier_indicator**: a binary 0/1 flag derived by applying the score
  threshold to outlier_score. Observations with a score above the
  threshold (default 0.5) are flagged as anomalies.

**Limitations**

Outlier Trees is a multivariate method, and its results are shaped by
which variables are included in the analysis. Including irrelevant or
redundant features can degrade performance, so users should give some
thought to feature selection before applying the method. As with the
other methods in this document, the input series should be stationary
--- macroanomaly applies the necessary preprocessing before this step.
On small datasets, the tree-based approach may produce unstable results,
and increasing the number of trees can help mitigate this.

## Isolation Trees

Isolation Forest (and Isolation trees) is a multivariate method for
detecting point anomalies. Its central insight is that anomalies are, by
nature, few and different: because they occupy sparse, atypical regions
of the feature space, they tend to be isolated from the rest of the data
more quickly and with fewer random partitions than normal observations.
The algorithm exploits this property directly, using isolation speed as
a proxy for anomalousness.

The method builds an ensemble of random binary trees --- isolation trees
--- each constructed on a random subsample of the data. Within each
tree, a feature and a split value are chosen at random, and the data is
partitioned recursively until each observation is isolated or a maximum
depth is reached. The number of splits required to isolate an
observation, averaged across all trees, forms its path length. This
average path length is then normalized into an anomaly score:
observations that are isolated quickly receive high scores, while those
that require many splits --- sitting in denser, more typical regions ---
receive lower ones.

The method scales efficiently to large, high-dimensional datasets and
operates without distributional assumptions. However, like Outlier
Trees, it may flag only a small number of anomalies in datasets with
moderate variability, and results can be sensitive to the choice of
threshold and tree configuration.

**Implementation in macroanomaly**

macroanomaly implements the isolation.forest algorithm from the isotree
package.

*Core parameters:*

- **Number of dimensions per split**: controls whether splits are made
  on a single variable at a time (1, the default) or on linear
  combinations of multiple variables (higher values). Higher values
  enable the algorithm to capture multivariate patterns that no single
  variable would reveal.

- **Number of trees**: the size of the tree ensemble. The default is 10,
  which is intentionally conservative; increasing this value improves
  the stability of anomaly scores at the cost of additional computation
  time.

- **Subsample fraction**: the proportion of the dataset used to build
  each tree. The default is 1 (the full dataset); values below 1 reduce
  computation time but may reduce accuracy.

- **Score threshold**: the minimum anomaly score above which an
  observation is flagged. Scores range from 0 to 1, with higher values
  indicating greater anomalousness. The default is 0.5.

- **Threads**: the number of CPU threads used for parallel computation.
  The default is 2.

- **Additional arguments**: using \... the user can pass additional
  arguments to the isolation.forest function, if needed (see Advanced
  parameters subsection).

*Advanced parameters:*

The following parameters are passed directly to the underlying algorithm
and allow finer control over the tree construction and flagging logic.
Default values are appropriate for most use cases, but users working
with unusual data structures may wish to adjust them.

- **max_depth**: the maximum depth of each isolation tree. The default
  is 4. Setting this to 0 causes each feature to be assessed
  independently, without conditioning on other variables.

- **min_gain**: the minimum improvement required for a split to be
  accepted, expressed as a proportion of the standard deviation or
  entropy. The default is 0.01.

- **z_norm**: the maximum Z-score still considered consistent with
  normal behavior. The default is 2.67. This parameter does not assume
  normality --- it simply defines the boundary between typical and
  unusual observations within a branch.

- **z_outlier**: the minimum Z-score required for an observation to be
  flagged as an outlier. The default is 8. The gap between z_outlier and
  z_norm controls how extreme a deviation must be relative to its local
  branch before it is flagged; reducing this gap increases the number of
  observations flagged.

- **pct_outliers**: the approximate maximum expected proportion of
  outliers per branch. The default is 0.01 (1%).

- **min_size_numeric**: the minimum number of observations required in a
  branch before outlier evaluation is performed on a numeric column. The
  default is 25; a branch must contain at least twice this number for a
  split to be evaluated.

- **min_size_categ**: the equivalent minimum branch size for categorical
  or ordinal columns. The default is 50.

- **categ_split**: the strategy used to split categorical variables.
  Options are \"binarize\" (default), \"bruteforce\", and \"separate\".

- **categ_outliers**: the strategy used to detect outliers in
  categorical columns. \"tail\" (default) is more sensitive;
  \"majority\" is stricter and better suited to large samples.

- **numeric_split**: how split points are defined for numeric variables.
  \"raw\" (default) uses the actual observed value; \"mid\" uses the
  midpoint between branches, which may be preferable for continuous
  data.

- **cols_ignore**: a vector of column names to exclude from splitting
  decisions. Excluded columns are still used as split targets but will
  not themselves drive partitioning.

- **follow_all**: whether to continue branching from every valid split.
  The default is FALSE. Setting this to TRUE can produce an
  exponentially large number of branches and a correspondingly large
  number of spurious detections; this setting is not recommended for
  routine use.

*Output fields:*

- **outlier_score**: a continuous value indicating the \"outlierness\",
  according to the documentation of the isotree package. Values close to
  1 are highly anomalous, while values close to 0.5 are \"average
  isolation depth\".

- **outlier_indicator**: a binary 0/1 flag derived by applying the score
  threshold to outlier_score. The default threshold is 0.5.

**Limitations**

Isolation Forest is a multivariate method, and the choice of which
features to include meaningfully affects its results. Noisy or
irrelevant features can obscure genuine anomalies by diluting the signal
in the partitioning process. The large number of available parameters
gives the method considerable flexibility, but also means that results
on small or unusual datasets may require careful tuning. As with all
methods in this document, macroanomaly applies trend and seasonal
adjustment before this step, ensuring that the algorithm operates on
stationary input.

# Detecting Anomalies in Practice

## Operational Considerations

The World Bank Group (WBG) curates databases spanning tens of thousands
of indicators, produced by Bank units or sourced from a large number of
external providers, and covering countries and territories worldwide.
This translates to hundreds of thousands of individual time series, many
of which are updated on a regular basis. At this scale, manual review is
not feasible: anomaly detection must be as automated and systematic as
possible, which is the primary motivation for providing an open-source,
scriptable package that can be integrated into existing data pipelines.

**Using multiple methods**

Each detection method implemented in macroanomaly has its own strengths
and blind spots --- no single method reliably identifies all anomaly
types across all series. In practice, it is strongly recommended to
apply several methods in combination. Each method produces a binary flag
(anomaly or not), and some additionally provide a continuous measure of
anomalousness, such as an absolute Z-score. Running multiple methods in
parallel allows detections to be triangulated: an observation flagged by
several methods independently warrants more attention than one flagged
by only a single method.

Prioritization is a practical necessity when detection yields a large
number of flagged observations, as it frequently will at the scale of
WBG databases. Useful criteria for prioritization include the number of
methods that flagged a given observation, the magnitude of the anomaly
score, the recency of the observation, the relative importance of the
indicator or geography in question, and the reliability of the data
source.

**Feedback loops and upstream correction**

The macroanomaly package is publicly available, and the methods and
workflows described in this document are intentionally open and
replicable. This reflects a broader principle: anomalies are ideally
detected and treated at the source --- by national statistical offices,
central banks, or regional and international organizations --- rather
than corrected downstream by data distributors. When the World Bank
detects a potential anomaly in a third-party dataset, the appropriate
response is to establish a feedback loop with the originating
institution rather than to apply unilateral corrections. The tools
described here are designed to support that two-way exchange.

**The iterative nature of anomaly detection**

An important characteristic of anomaly detection that users should keep
in mind is that it is inherently iterative and relative. If a detection
script is run, some values are corrected, and the same script is then
run again on the updated series, new anomalies will often surface ---
because removing or correcting extreme values changes the mean, standard
deviation, and other distributional properties of the series, shifting
what counts as \"normal.\" This is expected behavior, not a flaw, but it
means that anomaly detection should be treated as an ongoing process
rather than a one-time procedure, and that each successive round of
detection requires the same level of scrutiny as the first.

**Human oversight is essential**

macroanomaly is a decision-support tool, not a decision-making system.
Detected anomalies must be reviewed, classified, and assessed by
qualified experts --- potentially with the assistance of AI tools such
as those described in Section 8 --- before any action is taken. Under no
circumstances should flagged values be corrected automatically based on
detection outputs alone. All changes to published data must involve
human judgment and follow established quality control and governance
procedures. The role of the package is to surface candidates for review
efficiently; the responsibility for what happens next rests with the
people using it.

## Data Preparation

Before anomaly detection can be run, the input data must meet three
requirements: it must be structured in long format, it must contain no
missing values, and it must be normalized. The following subsections
explain how to prepare a dataset that satisfies these requirements.

### Formatting the Dataset

A single input dataset passed to macroanomaly may contain multiple
indicators and multiple geographies. However, all series in the dataset
must share the same periodicity --- annual, quarterly, or monthly. If
the same indicator is available at more than one frequency, the data
should be split by periodicity and anomaly detection run separately on
each subset.

**Long vs. wide format**

Time series data is commonly stored in one of two formats: wide or long.
macroanomaly requires data in long format, so datasets stored in wide
format must be reshaped before use. The package does not provide
reshaping functions; this step should be performed using base R
(reshape()) or a package such as tidyr (pivot_longer()).

In **wide format**, each row represents a single subject --- an
indicator-geography combination --- and each time period occupies a
separate column. This produces a compact table that is easy to read at a
glance but is not well-suited to programmatic processing.

  ------------------------------------------------------------------------
  Indicator        Geography   Y2020      Y2021      Y2022      Y2023
  ---------------- ----------- ---------- ---------- ---------- ----------
  Population       Belgium     11492641   11521238   11584008   11697557

  ------------------------------------------------------------------------

In **long format**, each row represents a single observation at a single
point in time for a single subject. The table is narrower but taller,
with separate columns identifying the indicator, geography, time period,
and observed value. This is the structure macroanomaly expects.

  ------------------------------------------------------------------------
  Indicator             Geography        Year             Value
  --------------------- ---------------- ---------------- ----------------
  Population            Belgium          2020             11492641

  Population            Belgium          2021             11521238

  Population            Belgium          2022             11584008

  Population            Belgium          2023             11697557
  ------------------------------------------------------------------------

Some indicators carry additional ***dimensions***. For example,
population data disaggregated by sex would include an extra column
identifying the subgroup. Each unique combination of indicator,
dimension(s), geography, and time period corresponds to a single row.

  ------------------------------------------------------------------------
  Indicator             Sex       Geography    Year       Value
  --------------------- --------- ------------ ---------- ----------------
  Population by Sex     Total     Belgium      2020       11492641

  Population by Sex     Total     Belgium      2021       11521238

  Population by Sex     Total     Belgium      2022       11584008

  Population by Sex     Total     Belgium      2023       11697557

  Population by Sex     Male      Belgium      2020       5660064

  Population by Sex     Male      Belgium      2021       5677211

  Population by Sex     Male      Belgium      2022       5708902

  Population by Sex     Male      Belgium      2023       5761410

  Population by Sex     Female    Belgium      2020       5832577

  Population by Sex     Female    Belgium      2021       5844027

  Population by Sex     Female    Belgium      2022       5875106

  Population by Sex     Female    Belgium      2023       5936147
  ------------------------------------------------------------------------

**Required columns**

macroanomaly requires the following columns to be present in the input
dataset. Column names are flexible --- the package does not impose
specific naming conventions --- but the content of each column must
conform to the descriptions below.

- **Indicator identifier**: a code or label uniquely identifying the
  indicator.

- **Dimensions** (if applicable): one column per dimension, such as sex
  or age group.

- **Geography**: a single column identifying the country, region, or
  territory. Each geography must be represented as a single value ---
  nested or hierarchical geographic structures should be flattened
  before input.

- **Time period**: the year, quarter, or month of the observation. All
  entries in a given dataset must share the same periodicity.

- **Observed value**: the numeric value of the indicator for the given
  combination of identifier, dimensions, geography, and time period.

The combination of indicator, dimensions, geography, and time period
must be unique for every row in the dataset. Duplicate combinations will
cause errors in the detection functions and should be resolved during
data preparation.

### Imputation of Missing Values

Time series of social and economic indicators frequently contain missing
values, and most anomaly detection methods cannot handle them directly.
macroanomaly therefore imputes missing values automatically as part of
its preprocessing pipeline, via the normalize() function (called with
impute = TRUE). The imputation is performed solely to enable anomaly
detection and has no status beyond that: imputed values should never be
saved to a database or used for any purpose other than running the
detection methods.

This point deserves emphasis. Imputation for anomaly detection is
intentionally simple and neutral --- designed to fill gaps in a way that
does not distort the series\' behavior, not to produce defensible
estimates of what the missing values actually were. If gap-filled series
are needed for dissemination or analysis, a dedicated imputation
approach is required: one that is methodologically appropriate for the
indicator in question, optimized for that context, and documented in a
transparent and reproducible manner.

Because macroanomaly may flag imputed values as anomalies during
detection, the package automatically excludes such flags from its
outputs. Only anomalies detected in originally observed values are
reported and require review or treatment.

Missing data require special care because preprocessing choices can
affect anomaly scores. When imputation is used to create a continuous
series for algorithmic purposes, users should assess whether the
imputation method changes the apparent severity or location of
anomalies. Flags on imputed observations, or on observations adjacent to
long gaps, should be treated as lower-confidence signals. In operational
use, review tables and visualizations should distinguish observed from
imputed values, and users should compare results before and after
imputation when missingness is substantial. Imputed values may support
screening, but they should not be treated as equivalent to observed data
for classification or correction decisions.

**Imputation methods**

The imputation step draws on functions from the imputeTS R package. The
following methods are available:

- **na_interpolation** (default): replaces missing values using
  interpolation. The default interpolation type is linear; spline and
  Stineman interpolation are also available.

- **na_kalman**: uses a Kalman filter on a state-space model to estimate
  missing values --- well-suited to series with trend or seasonal
  structure.

- **na_mean**: replaces missing values with the series mean, optionally
  computed over a local window.

- **na_locf**: carries the last observed value forward to fill
  subsequent gaps (Last Observation Carried Forward).

- **na_ma**: replaces missing values with a weighted moving average of
  neighboring observations.

- **na_seadec**: applies seasonal decomposition before interpolating
  residuals, making it appropriate for series with strong seasonal
  patterns.

- **na_random**: fills gaps with random draws from the observed
  distribution of the series.

The default method, na_interpolation with linear interpolation, is
appropriate for most development data series and requires no parameter
tuning. For series with pronounced trends or seasonality, na_kalman or
na_seadec may produce more neutral imputations. Full documentation for
each method, including parameter options, is available in the imputeTS
package documentation.

### Detrending and Deseasonalization

Most anomaly detection methods assume that a series is approximately
stationary --- that its mean and variance remain roughly constant over
time. Social and economic time series routinely violate this assumption:
they may exhibit long-term trends (gradual upward or downward drift) and
seasonal patterns (repeating cycles tied to the calendar). If these
components are not removed before detection, the result may be
consequential: flagging observations that are unusual only relative to a
global mean when they are perfectly normal for their position in a trend
or seasonal cycle (false positives), or missing genuine anomalies that
are absorbed into an expected pattern (false negatives).

**Available methods**

A range of approaches can be used to remove trend and seasonal
components, including differencing, regression-based decomposition,
moving average filters, STL (Seasonal-Trend decomposition using LOESS),
TBATS, Prophet, and state-space models such as the Kalman filter. These
methods vary in their complexity, assumptions, and suitability for
different series characteristics.

**Implementation in macroanomaly**

Detrending and deseasonalization are handled automatically within
macroanomaly\'s normalize() function, prior to any detection step. Users
do not need to preprocess series manually. The function applies STL
decomposition by default, which is well-suited to the short, potentially
irregular series typical of development data, and returns the remainder
component as input to the detection methods. The choice of decomposition
method can be adjusted through the function\'s parameters for users who
wish to experiment with alternatives.

## Implementation Examples

We provide two examples of scripts, one with annual data, the other one
with monthly data. They can be replicated (only need to change the
default folder name). Make sure the required packages are installed (if
necessary, install them). Note that the results may vary as datasets are
updated regularly.

### Example 1: Annual Series of Education Indicators

This example uses a subset of the World Development Indicators (WDI).
The dataset is filtered to education-related indicators --- those whose
identifier begins with \"SE.\" --- and restricted to observations from
1980 onwards. As of 18 April 2026, this selection comprises 1,015,910
observations across 147 indicators and 256 geographies.

```{r}
#| eval: false

#----------------------------------------------------------------------
# Anomaly detection in WDI dataset - Standard Education (SE) indicators
#----------------------------------------------------------------------

# Load the required R packages
library(macroanomaly)
library(collapse)
library(tidyr)
library(dplyr)

# Set the default folder >>> (enter your own) <<<
setwd("C:/.../")

# Download the WDI dataset if not previously done, read, and filter (267 Mb)
wdi_url <- "https://databank.worldbank.org/data/download/WDI_CSV.zip"
if (!file.exists("WDICSV.CSV")) {
  download.file(wdi_url, destfile = "WDI.zip")
  unzip("WDI.zip")
}

df_wdi_SE <- read.csv("WDICSV.csv")
df_wdi_SE <- df_wdi_SE[startsWith(df_wdi_SE$Indicator.Code, "SE."), ]

# Reshape the dataset from wide to long
df_wdi_SE <- df_wdi_SE %>%
  pivot_longer(cols = starts_with("X"),
               names_to = "Year", values_to = "Value") %>%
  mutate(Year = as.integer(sub("X", "", Year)))

# Drop years < 1980 and series with less than 10 non-missing values
df_wdi_SE <- df_wdi_SE[df_wdi_SE$Year >= 1980, ]
min_obs <- 10
df_wdi_SE <- df_wdi_SE %>%
  group_by(Country.Code, Indicator.Code) %>%
  filter(sum(!is.na(Value)) >= min_obs) %>%
  ungroup()

# Normalize the dataset (detrend, impute missing values)
df_wdi_SE |>
  macroanomaly::normalize(
    .value_col = "Value",
    .country_col = c("Country.Code", "Country.Name"),
    .indicator_col = "Indicator.Code",
    .time_col = "Year",
    .detrend = TRUE,
    .impute = TRUE) -> wdi_SE_normalized

# Detect anomalies (6 methods)
wdi_SE_normalized |>
  macroanomaly::detect(
    .method = c("tsoutlier", "isotree", "capa", "outliertree", "zscore", "hampel"),
    .args = list(
      capa = c(.min_seg_len = 3),
      isotree = c(threshold = 0.7),
      hampel = c(.trim = TRUE, .trim_sd = 1.5)),
    .additional_cols = TRUE) -> wdi_SE_anomalies

# Sort the anomalies data frame by severity of anomalies
wdi_SE_anomalies <- wdi_SE_anomalies[
  order(-wdi_SE_anomalies$outlier_indicator_total,
        -wdi_SE_anomalies$absZscore_zscore), ]

# Number of anomalies detected
summary(wdi_SE_anomalies)
table(wdi_SE_anomalies$outlier_indicator_total)

# Save output to CSV (all observations, with anomaly status by method)
fn <- paste0("WDI_SE_ANOMALIES", "_", format(Sys.Date(), "%Y-%m-%d"), ".CSV")
write.csv(wdi_SE_anomalies, file = fn, row.names = FALSE)

# Plot a sample of anomalies (top 20)
for (i in 1:20) {
  if (wdi_SE_anomalies$Imputed[i] == FALSE) {
    subttl <- paste0(wdi_SE_anomalies$Country.Name[i], " - ",
                     wdi_SE_anomalies$Indicator.Name[i])
    print(plot(wdi_SE_anomalies,
               country = wdi_SE_anomalies$Country.Code[i],
               indicator = wdi_SE_anomalies$Indicator.Code[i],
               .total_threshold = 2, x.lab = subttl))
  }
}

# To plot one selected anomaly (provide country code and indicator code)
plot(wdi_SE_anomalies,
     country = "IRL", # Ireland
     indicator = "SE.PRM.ENRR", # School enrollment, primary (% gross)
     .total_threshold = 2, x.lab = "Year")
```

Running all six detection methods on this dataset with the parameters
shown in the script produced the following distribution of flags across
observations:

  -----------------------------------------------------------------------
       Methods flagging the      Number of observations
           observation           
  ------------------------------ ----------------------------------------
         6 methods (All)         0

            5 methods            11

            4 methods            2,987

            3 methods            13,131

            2 methods            29,599

             1 method            483,699

             0 method            486,483
  -----------------------------------------------------------------------

No observation was flagged unanimously by all six methods, which is
consistent with the expectation that different methods have different
sensitivities and are designed to detect different types of anomalies.
Observations flagged by a larger number of methods should be prioritized
for review, as convergence across methods provides stronger evidence of
a genuine anomaly. Anomalies flagged by only one or two methods could in
this case probably be ignored by data curators.

**Output and visualization**

The output dataset is sorted first by the number of methods that flagged
each observation, then by the absolute Z-score, so that the most
anomalous observations appear at the top. The top 20 flagged
observations are displayed as time series plots, each showing the full
series to which the flagged observation belongs. An example plot is
shown below. Plots distinguish between flagged and non-flagged
observations, and between originally observed and imputed values,
allowing reviewers to quickly assess the context of each detection.

![- Anomaly plot - Annual time
series](images/media/image5.png){width="6.5in"
height="3.9381944444444446in"}

The full output is saved as a CSV file. This file contains every
observation in the input dataset --- flagged or not, imputed or not ---
and includes the following fields for each row: an imputation indicator,
a binary anomaly flag for each detection method, and the anomaly score
where the method produces one. This structure allows users to apply
their own prioritization criteria and to filter or rank detections in
ways suited to their specific review workflow.

### Example 2: Monthly Series of Unemployment Rate

This example applies the same workflow to a different dataset: monthly
estimates of the unemployment rate by sex and age group, sourced from
the International Labour Organisation (ILO). As of 18 April 2026, this
selection comprises 409,566 observations across 48 indicators and 68
geographies.

The primary difference from the previous example is the frequency of the
data. The time periods in the data file are provided in the format
yyyyMmm where yyyy is the year and mm is the month (01 to 12). Monthly
series have stronger and more regular seasonal patterns than annual
series, and the decomposition step must account for this. In
macroanomaly, the monthly frequency is specified via the .frequency
argument of the normalize() function, which ensures that trend and
seasonal components are removed appropriately before detection is run.
For this example, four of the six available detection methods are
applied.

```{r}
#| eval: false

#----------------------------------------------------------------------
# Anomaly detection in ILO dataset - Unemployment rate by sex and age group
#----------------------------------------------------------------------

library(macroanomaly)
library(readr)
library(collapse)
library(tidyr)
library(dplyr)

# Set default folder >>> (enter your own) <<<
setwd("C:/.../")

# Download the data from ILO website
df <- read_csv("https://rplumber.ilo.org/data/indicator/?id=UNE_DEAP_SEX_AGE_RT_M&format=.csv")

# Create a code i_code for each indicator (combine indicator, sex, age group)
df$icode <- paste0("UNE_", df$sex, "_", df$classif1)

# Drop series with insufficient number of non-missing values
min_obs <- 36
df <- df %>%
  group_by(ref_area, icode) %>%
  filter(sum(!is.na(obs_value)) >= min_obs) %>%
  ungroup()

# Normalize the dataset (detrend, impute missing values)
df |>
  macroanomaly::normalize(
    .value_col = "obs_value",
    .country_col = "ref_area",
    .indicator_col = "icode",
    .time_col = "time",
    .frequency = "monthly",
    .detrend = TRUE,
    .impute = TRUE) -> df_normalized

# Detect anomalies (4 methods)
df_normalized |>
  macroanomaly::detect(
    .method = c("tsoutlier", "capa", "zscore", "hampel"),
    .args = list(
      capa = c(.min_seg_len = 3),
      hampel = c(.trim = TRUE, .trim_sd = 1.5)),
    .additional_cols = TRUE) -> df_anomalies

df_anomalies <- df_anomalies[
  order(-df_anomalies$outlier_indicator_total,
        -df_anomalies$absZscore_zscore), ]

# Number of anomalies detected
summary(df_anomalies)
table(df_anomalies$outlier_indicator_total)

# Save output to CSV
# fn <- paste0("ILO_UNE_ANOMALIES", "_", format(Sys.Date(), "%Y-%m-%d"), ".CSV")
# write.csv(df_anomalies, file = fn, row.names = FALSE)

# Plot a sample of anomalies
for (i in 1:20) {
  if (df_anomalies$Imputed[i] == FALSE) {
    print(plot(df_anomalies,
               country = df_anomalies$ref_area[i],
               indicator = df_anomalies$icode[i],
               .total_threshold = 2, x.lab = "Month"))
  }
}
```

Running the four selected detection methods on this dataset with the
parameters shown in the script produced the following distribution of
flags across observations:

  -----------------------------------------------------------------------
       Methods flagging the      Number of observations
           observation           
  ------------------------------ ----------------------------------------
         4 methods (All)         102

            3 methods            1,512

            2 methods            2,005

             1 method            164,376

             0 method            241,571
  -----------------------------------------------------------------------

The example plot shown below shows outliers in the male unemployment
rate for population aged 35 to 44 in 2020, which may be explained by the
COVID19 pandemic.

![- Anomaly plot - Monthly time
series](images/media/image6.png){width="6.5in"
height="4.045833333333333in"}

# Classifying and Explaining Anomalies Using AI

Once the anomaly detection process has flagged a set of candidate
observations, the next task is to determine what each one represents. An
anomalous data point is not inherently wrong---it may be a genuine error
that needs correction, or it may be a legitimate value that accurately
reflects an unusual real-world event. The goal of the classification
step is to make that determination efficiently and consistently, at a
scale that would be impractical to achieve through manual investigation
alone.

This section describes how artificial intelligence can assist data
curators in classifying and explaining flagged anomalies. It covers the
role of Large Language Models (LLMs) in generating evidence-based
explanations, the multi-model architecture used to reduce classification
errors, the taxonomy of classification outcomes, and how validated
AI-generated explanations can be repurposed as annotations that improve
the interpretability of the data for end users. The tools developed for
classification and explanation of anomalies are openly accessible on
Github (see <https://github.com/worldbank/ai4data>).

## From Detection to Classification

Anomaly detection produces a list of flagged observations, but not all
flags are equal. Depending on the models and thresholds applied, a
portion of flagged values will be false positives---observations that
appear statistically unusual but are in fact within normal variation, or
whose deviation is not large enough to warrant concern. Before
proceeding to classification, it is the data curator\'s responsibility
to apply filters that reduce the review set to the cases most likely to
require action.

These filters can be applied along several dimensions:

1.  The absolute z-score, or equivalent deviation measure, produced by
    the detection step. Observations that deviate more strongly from
    expected values are more likely to represent genuine anomalies and
    should be prioritized.

2.  The number of independent detection models that flagged the same
    observation. Agreement across multiple methods provides stronger
    evidence that something unusual is occurring, as opposed to an
    artifact of a single algorithm\'s sensitivity.

3.  Institutional relevance: indicators produced directly by the World
    Bank may be prioritized over third-party indicators redistributed by
    the Bank, where timely remediation has a direct operational and
    reputational impact.

4.  User impact: indicators with high query volume, recent reference
    periods, or proximity to an upcoming publication cycle warrant
    higher priority than older or less-used series.

After filtering, the curated subset is passed to the classification
step. Here, each anomaly must be assigned to one of two broad outcomes:
it is either a genuine error that requires correction or removal, or it
is a legitimate value that requires annotation. Making this
determination accurately, at scale, is the central challenge that
AI-assisted classification addresses.

After review, each flagged observation should be assigned to one of the
operational classification categories defined in Section 8.4. This
classification records the most likely explanation for the anomaly based
on the available evidence and should be maintained separately from any
subsequent treatment decision. Although the taxonomy uses the label
*insufficient data*, the intent is to indicate insufficient evidence to
determine whether the flag reflects a data error, a legitimate signal,
or a limitation of the available information.

+-----------------------------------------------------------------------+
| Note on scope                                                         |
|                                                                       |
| The classification step described in this section operates on the     |
| filtered output of anomaly detection. It is not a substitute for the  |
| filtering step itself, nor for the governance processes that govern   |
| treatment decisions described in Section 9.                           |
+=======================================================================+

## The Role of AI in Anomaly Classification

Manual classification of anomalies is a knowledge-intensive process. To
determine whether a sharp drop in a country\'s income distribution
indicator in a given year reflects a data error or a real economic
event, a curator needs contextual knowledge spanning economic history,
statistical methodology, and country-specific conditions, often across
dozens of countries and indicators simultaneously. At scale, this is not
feasible without assistance.

Large Language Models are well-suited to this task because explaining a
data anomaly is fundamentally a retrieval and reasoning problem. Given
an unusual observation in a particular country at a particular time, the
question is: what documented events or changes could account for it? The
answer space is large and heterogeneous, ranging from economic crises,
conflicts, natural disasters, policy changes, methodological revisions,
to statistical rebasings, but it is also well-documented in the kind of
text that LLMs are trained on.

An LLM that has processed a broad corpus of economic history and
statistical documentation is in a reasonable position to connect, for
example, a spike in a poverty indicator in a country undergoing
hyperinflation to a well-documented macroeconomic crisis, or to
recognize that a step-change in national accounts data aligns with a
rebasing exercise that preceded EU accession. These are not inferences
that require deep statistical reasoning; instead, they require the kind
of contextual knowledge that LLMs encode effectively. The proposed
method elicits embedded knowledge from various LLMs.

The AI is prompted to attempt an evidence-based explanation for each
flagged observation. If it identifies a documented reason, a named
event, a statistical revision, or a policy change with a clear date
range, it classifies the anomaly as a likely legitimate value and
provides a concise narrative. If it cannot find a credible explanation,
it classifies the observation as a likely error. In cases where the
evidence is ambiguous or insufficient to reach a confident conclusion,
it records the case accordingly rather than forcing a classification.

+-----------------------------------------------------------------------+
| Important                                                             |
|                                                                       |
| Treat AI-generated classifications as suggestions. They are designed  |
| to prioritize and inform expert review. All treatment actions,        |
| including corrections, removals, and annotations, require human       |
| validation and appropriate approvals before being applied to the      |
| database.                                                             |
+=======================================================================+

## Multi-model Architecture

A single LLM can be unreliable. It may hallucinate a plausible-sounding
but fabricated explanation, miss relevant context, or produce
overconfident assessments on ambiguous evidence. To reduce these risks,
the pipeline uses a multi-model architecture in which two independent
models evaluate each anomaly separately, and their outputs are then
compared in a synthesis step.

### Independent Evaluation

Two language models evaluate each flagged anomaly independently,
receiving identical prompts. Each prompt provides the model with the
indicator name and code, the country or economy, the flagged time
window, a trimmed excerpt of the surrounding time series including
imputation flags, and an instruction to identify an evidence-based
explanation if one exists.

Each model returns a structured response containing: a classification
from the controlled taxonomy described in Section 8.4, a free-text
explanation of its reasoning, a confidence score between 0 and 1, and a
list of evidence sources with names, date ranges, source type
classifications, and verifiability assessments.

The two models operate without knowledge of each other\'s outputs. This
independence is deliberate. It prevents one model from anchoring on the
other\'s framing, and it produces genuinely separate opinions that can
be compared and contrasted. When both models reach the same conclusion
independently, that agreement provides meaningful corroboration. When
they diverge, the divergence is informative as it flags a case where the
evidence is genuinely ambiguous.

### Synthesis and Agreement

After both models return their assessments, a synthesis step compares
their classifications and records the outcome in an agreement field.
Three states are possible:

  -----------------------------------------------------------------------
  Agreement     Meaning                  Implication for the curator
  ------------- ------------------------ --------------------------------
  Agree         Both models              Higher confidence in the
                independently reached    consensus verdict. Either
                the same classification. model\'s narrative can serve as
                                         the basis for an annotation,
                                         subject to expert review.

  Disagree      The two models reached   The case requires closer curator
                different                attention before any treatment
                classifications.         is applied. Both narratives
                                         should be reviewed alongside
                                         primary source documentation. A
                                         common pattern is one model
                                         returning an external driver
                                         classification and the other
                                         returning insufficient data,
                                         indicating that evidence exists
                                         but is not conclusive.

  Single        Only one model evaluated No cross-model validation is
                this record.             available. Confidence relies
                                         entirely on the single model\'s
                                         output and should be interpreted
                                         accordingly.
  -----------------------------------------------------------------------

The synthesis step also produces a consolidated confidence score,
aggregating the per-model values, and an evidence strength assessment
that characterizes the quality of the evidence cited. These two fields
together provide a richer signal for prioritization than either the
classification or the confidence score alone.

### Structured Evidence Sourcing

A key design requirement of the pipeline is that explanations must be
evidence-based rather than inferential. Models are prompted to identify
specific, documented events or changes, e.g., named crises, published
revisions, policy announcements with date ranges, rather than offering
generic macroeconomic reasoning. This serves two purposes: it makes
explanations verifiable against primary sources, and it produces
annotation text that is specific and credible.

When a model cites evidence, it records the source using a controlled
taxonomy of source types. The table below lists the full set of values
used.

**Controlled vocabulary for evidence source types used in AI-generated
anomaly explanations**

  -----------------------------------------------------------------------
  Source type          Description
  -------------------- --------------------------------------------------
  Economic crisis      Recession, financial crisis, hyperinflation,
                       currency collapse, or similar macroeconomic shock
                       with a documented impact on the measured
                       phenomenon.

  Policy change        Government policy shift, structural reform,
                       legislative change, or regulatory action with a
                       direct effect on the indicator.

  Conflict             Armed conflict, civil war, or political violence
                       disrupting economic activity or data collection
                       capacity.

  Natural disaster     Earthquake, flood, drought, cyclone, or other
                       natural event with measurable economic or social
                       impact.

  Global event         A worldwide event simultaneously affecting
                       multiple countries, such as a global financial
                       crisis or pandemic.

  Regional event       An event primarily affecting a specific region or
                       economic bloc, such as an EU accession wave or
                       regional financial contagion.

  Data revision        A formal revision to previously published data
                       issued by the national statistics office or the
                       World Bank.

  Rebasing             A change in the base year or reference period used
                       for national accounts or price indices.

  Data error           A suspected transcription, processing, or
                       aggregation error introduced during data handling.

  Other                Evidence that does not fit the categories above.
  -----------------------------------------------------------------------

## Classification Taxonomy

The pipeline assigns each anomaly one of four classification values.
These are not just descriptive labels --- each maps directly to a
different treatment path and carries specific implications for how the
observation should be handled. The table below sets out the full
taxonomy.

**Anomaly classification taxonomy, with typical evidence strength and
recommended treatment path**

  ------------------------------------------------------------------------
  Classification   Description          Typical      Treatment path
                                        evidence     
                                        strength     
  ---------------- -------------------- ------------ ---------------------
  External driver  A real-world event   Strong       Retain the value.
                   such as economic     direct       Validate the cited
                   crisis, conflict,                 explanation against
                   natural disaster,                 source documentation.
                   policy change,                    Use the narrative as
                   global or regional                an annotation once
                   shock, that                       validated by a
                   plausibly explains                subject matter
                   the observed                      expert.
                   movement. The data                
                   value is likely                   
                   correct; the anomaly              
                   reflects the world                
                   rather than a                     
                   problem with the                  
                   data.                             

  Measurement      The movement         Strong       Review methodology
  system update    reflects a change in direct       documentation for the
                   statistical                       period. Check for a
                   methodology, survey               published
                   design, national                  series-break note.
                   accounts rebasing,                Annotate the affected
                   or                                window accordingly.
                   system-of-accounts                
                   revision rather than              
                   a real change in the              
                   underlying                        
                   phenomenon. The                   
                   value may be correct              
                   within a revised                  
                   framework but                     
                   represents a series               
                   break.                            

  Data error       The movement is      No evidence  Open a data-quality
                   inconsistent with                 ticket. Cross-check
                   plausible real-world              against primary
                   variation and likely              source data and
                   reflects a                        revision history.
                   transcription error,              Consult the
                   processing artifact,              originating data
                   or implausible value              provider where the
                   introduced during                 data originates from
                   data handling.                    an external source.

  Insufficient     Neither a real-world No evidence  Deprioritize unless
  data             driver nor a data                 the magnitude is
                   issue could be                    large or domain
                   identified with                   knowledge points to a
                   sufficient                        specific cause. Flag
                   confidence. The                   for future review.
                   observation is                    
                   statistically                     
                   unusual but                       
                   unexplained.                      
  ------------------------------------------------------------------------

A companion field, **is_anomaly**, records whether the pipeline
confirmed the flag as a genuine anomaly. Observations classified as
insufficient data receive a value of false, indicating that the pipeline
found nothing conclusive enough to confirm the detection, which is
itself a signal to deprioritize the case rather than investigate
further.

### Evidence Strength

Alongside the classification, the pipeline records the quality of
evidence supporting it. This is an important secondary signal: a
high-confidence external driver classification backed by strong direct
evidence warrants different treatment from one backed only by weak or
speculative context. Four levels are used.

**Evidence strength levels used in anomaly explanation outputs**

  -----------------------------------------------------------------------
  Level               Description
  ------------------- ---------------------------------------------------
  Strong direct       Well-documented, directly relevant evidence exists
                      such as a named event with a clear date range and
                      an established record that directly connects to the
                      indicator and country in question.

  Moderate contextual Relevant context exists but does not conclusively
                      explain the movement. Some inferential gap remains
                      between the cited event and the specific
                      observation.

  Weak speculative    Evidence is present but highly speculative or
                      tangential to the specific observation.

  No evidence         No supporting evidence was identified. The
                      classification is based on the shape of the series
                      alone.
  -----------------------------------------------------------------------

## Using Explanations as Data Annotations

The classification and confidence fields support internal triage. But
the most durable output of the pipeline, the output that creates lasting
value beyond the curation workflow, is the free-text explanation. For
anomalies confirmed as legitimate outliers with documented causes, this
narrative is the basis for an annotation that should accompany the data
point wherever it is used: in databases, visualizations, APIs, and
publications.

Annotations matter because they change what the data communicates to its
users. An unexplained spike in an income distribution indicator looks
like a possible error. The same data point, annotated with a concise
reference to the economic or political shock that caused it, becomes
interpretable evidence. It tells users not only what happened but why,
and it signals that the value has been reviewed and its provenance
understood. This is especially important for development data, where
anomalous observations in crisis periods or fragile states are often the
most policy-relevant data points in an entire series.

### What Makes a Good Annotation

The pipeline is designed to produce annotation-ready text for external
driver and measurement system update cases, subject to expert
validation. Effective annotations share several characteristics:

5.  **Specificity.** A reference to a named event with a date range is
    more credible and verifiable than a generic statement about economic
    instability. Where a model cites the 2008 Icelandic financial crisis
    (September 2008 to December 2010), for example, the annotation can
    be checked against the historical record and the causal link to the
    income distribution indicator explained clearly.

6.  **Relevance to the indicator.** A good annotation does not simply
    note that an event occurred. It should explain the mechanism by
    which the event would be expected to affect the specific indicator
    being annotated. A financial crisis that erodes household incomes
    across all quintiles affects an income share indicator differently
    from one that concentrates losses at the top.

7.  **Concision.** An annotation is not a literature review. Two to
    three sentences that identify the cause and describe its effect on
    the series are sufficient for most use cases.

8.  **Verifiability.** Annotations derived from well-documented evidence
    should be presented with greater confidence than those based on
    partially documented or inferred connections. Where evidence is only
    moderately documented, the annotation should reflect that
    uncertainty.

The following example illustrates what the pipeline produces for a
well-documented external driver case:

  -----------------------------------------------------------------------
  Indicator             Income share held by second 20%
  --------------------- -------------------------------------------------
  Country               Iceland

  Anomaly window        2008--2010

  Classification        external_driver · both models agree · confidence:
                        0.9 · evidence: strong_direct

  Model A narrative     The sharp drop in 2008 followed by a rebound in
                        2009--2010 aligns with the impact of the 2008
                        Icelandic financial crisis, a well-documented
                        economic shock affecting income distribution.

  Model B narrative     The Icelandic financial crisis, which began in
                        late 2008, caused severe economic disruption and
                        likely impacted income distribution, leading to a
                        fluctuation and subsequent increase in the income
                        share held by the second 20% during this period.

  Evidence cited        2008 Icelandic financial crisis · Sep 2008 -- Dec
                        2010 · economic_crisis · well_documented
  -----------------------------------------------------------------------

And the following example shows a disagree case, where classification
requires closer curator judgment before any annotation is applied:

  -----------------------------------------------------------------------
  Indicator             Income share held by second 20%
  --------------------- -------------------------------------------------
  Country               Slovak Republic

  Anomaly window        2005--2006

  Classification        disagree: Model A → external_driver (conf. 0.8) ·
                        Model B → insufficient_data (conf. 0.5)

  Model A narrative     The fluctuation may reflect structural economic
                        changes following Slovakia's EU accession in
                        2004, which triggered significant adjustments in
                        income distribution.

  Model B narrative     No specific, well-documented event or data
                        revision could be identified to explain the
                        observed fluctuations during this period.

  Recommended action    Review both narratives against source
                        documentation before accepting the EU accession
                        explanation. Do not auto-populate an annotation
                        until a curator has validated the causal link.
  -----------------------------------------------------------------------

### Selecting and Applying Annotation Text

The pipeline produces candidate annotation text. The following guidance
applies to translating pipeline outputs into annotations accepted for
publication:

9.  **When both models agree and evidence is strong:** Either model\'s
    narrative can serve as the basis for the annotation, or a
    synthesized version can be constructed. Expert review is still
    required, but the bar for acceptance is lower when independent
    models have converged on the same explanation from strong evidence.

10. **When models disagree:** Do not auto-populate an annotation.
    Require curator review of both narratives alongside primary sources.
    If one model\'s explanation is accepted, the annotation should
    reflect the level of confidence --- for example, noting that the
    causal link is plausible but not definitively established.

11. **When evidence strength is moderate or weak:** Apply additional
    scrutiny. The model has identified a potential explanation but the
    connection to the specific movement is inferential. Consider whether
    a qualified annotation is more appropriate than a confident one, or
    whether the case should be flagged as pending review.

12. **For measurement system update cases:** The annotation should
    reference the specific methodological change and, where possible,
    link to the published documentation --- a rebasing note, a revision
    circular, or a national statistics office bulletin.

Once validated, annotations must be stored in the database and made
available to end users through all channels: data tables,
visualizations, and APIs. Annotations held only in internal curation
systems provide no benefit to users and do not fulfil the goal of
improving data interpretability.

![- Anomaly classification
reviewer](images/media/image7.png){width="6.591882108486439in"
height="4.544596456692913in"}

## Limitations and Safeguards

The AI-assisted classification pipeline is a tool for improving the
efficiency and consistency of curation, not one that eliminates the need
for human judgment. Understanding its limitations is as important as
understanding its capabilities.

### Known Limitations

13. **Knowledge cutoffs.** LLMs have training data cutoffs and will not
    be aware of events that occurred after their training was completed.
    For near-real-time detection workflows, very recent events may be
    misclassified as data errors or insufficient data. This limitation
    may require augmenting the pipeline with retrieval from current news
    sources or official statistical release calendars.

14. **Hallucination risk.** Models can generate plausible-sounding but
    fabricated explanations. A model may cite a real event that did not
    in fact affect the specific country or indicator in question, or
    connect a genuine event to a series movement through faulty
    reasoning. The structured evidence sourcing requirement, which
    compels models to name specific, verifiable events, reduces but does
    not eliminate this risk.

15. **Confidence calibration.** Confidence scores reflect the model\'s
    internal self-assessment, not an externally calibrated probability.
    A score of 0.9 should be read as a relative signal, i.e., higher
    confidence cases are more likely to be reliable, rather than as a
    statement that there is a 90 percent probability of correctness.

16. **Geographic coverage.** Models tend to be better calibrated for
    well-documented events in larger economies than for events in
    smaller or lower-income countries where English-language
    documentation is sparse. External driver explanations for anomalies
    in these contexts should be treated with additional scrutiny.

17. **Methodological changes.** Statistical rebasings, survey redesigns,
    and system-of-accounts updates are documented in technical
    publications that may not be well-represented in model training
    data. The measurement system update classification should always be
    verified against official methodology notes, regardless of model
    confidence.

### Built-in Safeguards

Several design features of the pipeline are specifically intended to
manage these limitations:

18. Independent dual evaluation reduces the risk that a single model\'s
    error goes undetected. When two models independently produce the
    same explanation, the probability of a shared fabrication is
    substantially lower than for a single-model output.

19. Structured evidence sourcing makes explanations verifiable. A model
    that must cite a named event with a date range is more constrained
    than one that can offer free-form reasoning, and vague or
    unverifiable explanations are easier to identify and reject.

20. Explicit disagreement surfacing ensures that ambiguous cases reach
    human attention rather than being silently resolved in favor of the
    more confident model.

21. The mandatory expert validation step before annotation acceptance
    means that the pipeline\'s limitations affect the efficiency of
    curation, not the quality of published annotations.

# Treating Anomalies

The anomaly detection and classification processes described in Sections
7 and 8 are designed to identify and interpret unusual observations in
the data. They produce a targeted and classified review list---but they
do not produce treatment decisions. All changes to the database, whether
corrections, removals, imputations, or annotations, must follow clearly
defined procedures, require appropriate approvals, and be fully
documented. This section describes how to translate classification
outputs into responsible treatment actions.

## Prioritization of Treatment

The detection and classification processes may surface a large number of
anomalies at once, and not all of them can or should be addressed with
the same urgency. A systematic approach to prioritization is essential
to avoid spreading curation effort thinly across the full set of flags
while genuine errors in high-impact series go unaddressed.

The combination of classification, evidence strength, model agreement,
and detection confidence together create a natural triage ordering. The
following sequence is recommended as a starting framework, to be adapted
to each team\'s institutional context and resources:

1.  **Data error cases first.** These represent the highest-probability
    genuine mistakes in the data and warrant immediate investigation.
    The absence of any documented external driver, combined with a
    movement that is inconsistent with plausible real-world variation,
    is the strongest available signal that something is wrong with the
    data rather than with the world.

2.  **Measurement system update cases.** These are often resolvable
    quickly: the relevant methodological documentation either exists or
    it does not. If a published series-break note confirms the change,
    annotation can proceed. If no documentation exists, the case should
    be escalated for methodological review.

3.  **External driver cases with strong direct evidence.** These
    represent the majority of confirmed anomalies. Where both models
    agree and evidence is strong, the curator\'s role is verification
    and annotation acceptance, not investigation from scratch. These
    cases can often be processed efficiently in bulk.

4.  **External driver cases with moderate or weak evidence.** These
    require more careful evaluation. The model has identified a
    plausible explanation but the connection to the specific indicator
    movement is not definitive. Treat these as hypotheses to test
    against source documentation rather than conclusions to accept.

5.  **Disagree cases, regardless of classification.** Model disagreement
    is a signal that the evidence is genuinely ambiguous. These cases
    should not be resolved by defaulting to the higher-confidence
    model\'s view --- they warrant independent curator review before any
    treatment is applied.

6.  **Insufficient data cases last.** In the absence of a documented
    explanation or a clear error signal, these are the hardest cases to
    resolve and the least likely to yield actionable findings.
    Deprioritize unless the magnitude is unusually large, the indicator
    is of high institutional importance, or domain knowledge suggests a
    specific direction to investigate.

In addition to the classification-based ordering above, the following
factors should inform prioritization across all categories:

- **Institutional relevance**: For the World Bank Group, indicators
  produced by the Bank (as opposed to third‑party indicators
  redistributed by the Bank) may be prioritized for timely remediation.

- **Recency.** Anomalies affecting recent reference periods, or those
  near an upcoming publication deadline, should be addressed before
  older flags in the same series.

- **User impact.** Indicators with high query volume or significant
  policy relevance should be prioritized over less-used series with
  equivalent anomaly characteristics.

- **Detection confidence.** Observations flagged by multiple independent
  detection methods and with high absolute z-scores should be considered
  more certain than those flagged by a single method at a lower
  threshold.

## Governance and Accountability

Treating anomalies means modifying or annotating data that users and
downstream systems depend on. This requires clear ownership, defined
workflows, and comprehensive documentation---both to ensure that
decisions are made by the right people and to provide a traceable record
of what was changed, why, and by whom.

### Roles and approvals

Before beginning a treatment cycle, the following roles should be
defined:

- Who is responsible for reviewing flagged anomalies and making initial
  treatment recommendations.

- Who has the authority to decide on treatment actions---corrections,
  removals, imputations, and annotations.

- Who approves changes before they are committed to the production
  database.

- Who is responsible for notifying external data providers when
  anomalies are identified in data originating from third parties.

### Versioning and audit trails

All changes to the data must be versioned and logged. The audit trail
for each modified observation should capture at minimum:

- The original value and the new value (or the fact of removal).

- The date and time the change was made.

- The identity of the person who made the change and, where applicable,
  the person who approved it.

- The method applied---recalculation, imputation, manual
  correction---and its basis.

This documentation is mandatory. It explains why each value is recorded
as it is and helps prevent known errors from returning in future data
refreshes.

## Treatment of Confirmed Errors

When an anomaly is confirmed as a data error---through curator review,
cross-checking against source data, or consultation with the originating
data provider---the goal is to replace it with a correct value wherever
feasible. The path to a correct value depends on the nature of the error
and the availability of source material, and should be determined in
consultation with subject matter experts who understand the indicator
and the data production process.

### Obtaining a correct value

The following methods may be used, in approximate order of preference:

- **Recalculation from source data.** Where the original microdata or
  source series is accessible, the correct value can be derived
  directly. This is the most defensible approach and should be used
  wherever possible.

- **Correction in the originating database.** For anomalies in data that
  originate from an external source, the data provider should be
  notified and the correction applied upstream. This prevents the error
  from being reintroduced on the next data refresh and ensures
  consistency across systems that draw from the same source.

- **Imputation.** Where a correct value cannot be obtained from source
  data, a statistically sound imputed value may be used as a
  replacement. The imputation method and its assumptions must be
  documented alongside the imputed value in the database. An
  **is_imputed** flag or equivalent indicator must be set, and the
  imputation must be visible to users through all data access channels.

- **Removal.** If a correct value cannot be obtained and imputation is
  not appropriate, the observation should be removed. Retaining a known
  error in the database is worse than a gap---it misleads users and any
  downstream analysis that relies on the series.

### Handling corrections from external sources

Where a correction is applied in the World Bank database but has not yet
been fixed in the originating source, the corrected value must be
accompanied by an annotation that records:

- That the value has been adjusted from the original source value.

- The nature of the correction and the basis for it.

- That the adjusted value should not be attributed to the originating
  provider as their published figure.

Tagged correction records also allow automated data refresh processes to
recognize previously corrected values and avoid reintroducing the
original error. Without this tagging, a future refresh that overwrites
the corrected value with the erroneous source value will silently undo
the curation work.

## Treatment of Legitimate Outliers

Anomalies confirmed as genuine outliers---statistically extreme but
correct values that accurately reflect an unusual real-world
event---should be retained and annotated. The annotation serves two
purposes: it explains the value to end users who might otherwise
question its accuracy, and it signals that the observation has been
reviewed and its provenance understood.

### Annotation content

Annotations for legitimate outliers should be concise, specific, and
written in plain language accessible to a non-specialist user. They
should identify the cause of the anomaly and describe, in general terms,
its effect on the indicator. Where the cause is a well-documented
external event, the annotation should name it. Where the cause is a
methodological change, the annotation should reference the change and,
where possible, link to the documentation.

AI-generated narratives from the explanation pipeline are an appropriate
starting point for annotation text, subject to validation by a subject
matter expert. The expert\'s role is to confirm that the cited
explanation is accurate for the specific country and indicator, that the
causal mechanism described is plausible, and that the language is
appropriate for the intended audience.

### Annotation dissemination

An annotation that exists only in a curation system provides no benefit
to data users. Once validated, annotations must be made available across
all channels through which users access the data:

- In database tables, as a metadata field attached to the observation or
  series.

- In data visualizations, as tooltip text or footnotes that appear when
  users interact with anomalous data points.

- In APIs, as structured fields in the response payload that can be
  programmatically retrieved alongside the data value.

- In downloaded datasets, as accompanying notes or a dedicated
  annotations column.

Where technically feasible, it is also valuable to expose evidence
source citations through these same channels. Transparency about how
anomalies were identified and why they were classified as legitimate
supports user interpretation and strengthens trust in the data.

## Memory and Suppression of Reviewed Anomalies

Once an anomaly has been reviewed, classified, and either corrected or
annotated, it should be suppressed from future detection runs for the
same observation. If the detection process is run again---as part of a
periodic refresh, a re-run after a data update, or a quality audit---it
will by design re-flag the same unusual observations. Without a
suppression mechanism, curators will repeatedly encounter anomalies that
have already been investigated and resolved, wasting review capacity and
eroding confidence in the detection system.

The suppression record should be stored alongside the audit trail for
the original treatment decision and should include the classification
assigned, the treatment applied, and the date of review. It should be
associated with the specific (indicator, country, year window)
combination so that future detection runs can identify and exclude
previously reviewed cases.

Suppression does not mean permanently ignoring the observation. If new
data becomes available that changes the context---a source revision that
contradicts a previous correction, or a newly published methodology note
that reframes a previously unexplained anomaly---the suppression record
can be reviewed and lifted. The goal is to avoid redundant
re-investigation of settled cases, not to prevent updates when new
information warrants them.

![- Displaying annotations in
charts](images/media/image8.png){width="6.4956364829396325in"
height="5.561096894138233in"}

# References

1.  Bengtsson H (2025). *R.utils: Various Programming Utilities*. R
    package v2.13.0. doi:10.32614/CRAN.package.R.utils.

2.  Borchers H (2025). **pracma**: Practical Numerical Math Functions.
    doi:10.32614/CRAN.package.pracma.
    \<https://doi.org/10.32614/CRAN.package.pracma\>, R package version
    2.4.6, \<https://CRAN.R-project.org/package=pracma\>.

3.  Cortes D (2026). **outliertree**: Explainable Outlier Detection
    Through Decision Tree Conditioning.
    doi:10.32614/CRAN.package.outliertree
    \<https://doi.org/10.32614/CRAN.package.outliertree\>, R package
    version 1.10.0-1,
    \<https://CRAN.R-project.org/package=outliertree\>.

4.  Cortes D (2025). **isotree**: Isolation-Based Outlier Detection\_.
    doi:10.32614/CRAN.package.isotree
    \<https://doi.org/10.32614/CRAN.package.isotree\>, R package version
    0.6.1-4, \<https://CRAN.R-project.org/package=isotree\>.

5.  Fisch A, Grose D, Eckley IA, Fearnhead P, Bardwell L (2024).
    "**anomaly**: Detection of Anomalous Structure in Time Series Data."
    Journal of Statistical Software\_, \*110\*(1), 1-24.
    doi:10.18637/jss.v110.i01 \<https://doi.org/10.18637/jss.v110.i01\>.

6.  Hyndman R, Athanasopoulos G, Bergmeir C, Caceres G, Chhay L,
    O\'Hara-Wild M, Petropoulos F, Razbash S, Wang E, Yasmeen F (2026).
    ***forecast**: Forecasting functions for time series and linear
    models*.
    [doi:10.32614/CRAN.package.forecast](https://doi.org/10.32614/CRAN.package.forecast),
    R package version 9.0.2,
    \<[https://pkg.robjhyndman.com/forecast/\>](https://pkg.robjhyndman.com/forecast/).

7.  Krantz S (2025). ***collapse**: Advanced and Fast Data
    Transformation in R*.
    \<[doi:10.5281/zenodo.8433090](https://doi.org/10.5281/zenodo.8433090)\>,
    R package version 2.1.6, \<<https://fastverse.org/collapse/>\>.

8.  López-de-Lacalle J (2024). **tsoutliers**: Detection of Outliers in
    Time. Series. Doi:10.32614/CRAN.package.tsoutliers.
    \<https://doi.org/10.32614/CRAN.package.tsoutliers\>, R package
    version 0.6-10, \<https://CRAN.R-project.org/package=tsoutliers\>.

9.  O\'Hara-Wild M, Hyndman R, Wang E (2024). ***feasts**: Feature
    Extraction and Statistics for Time Series*. R package v0.4.1.
    doi:10.32614/CRAN.package.feasts.

10. O\'Hara-Wild M, Hyndman R, Wang E (2024). ***fabletools**: Core
    Tools for Packages in the \'fable\' Framework*. R package v0.5.0.
    doi:10.32614/CRAN.package.fabletools.

11. Pedersen T (2025). ***patchwork**: The Composer of Plots*. R package
    v1.3.1. doi:10.32614/CRAN.package.patchwork.

12. Perfetti Villa L, L. Newhouse D, Dupriez O (2026). **macroanomaly**:
    Macroanomaly: Detects outliers in time series. R package version
    \<https://github.com/worldbank/macroanomaly\>.

13. R Core Team (2025). **R**: A Language and Environment for
    Statistical Computing. R Foundation for Statistical Computing,
    Vienna,
    Austria. [https://www.R-project.org/](https://www.r-project.org/)

14. Solatorio A, Dupriez O (2026). **ai4data.anomaly**: Classifying and
    eliciting explanation of data anomalies using LLMs. Python package
    \<<https://github.com/worldbank/ai4data/tree/main/src/ai4data/anomaly/explanation>\>.

15. Solatorio A, Dupriez O (2026). Data anomaly classification and
    explanation demo notebook:
    \<<https://github.com/worldbank/ai4data/blob/main/notebooks/data-anomaly/Timeseries_Anomaly_Explanation_with_LLMs.ipynb>\>

16. Steffen Moritz, Thomas Bartz-Beielstein (2017). "**imputeTS**: Time
    Series Missing Value Imputation in R." The R Journal\_, \*9\*(1),
    207-218. doi:10.32614/RJ-2017-009
    \<https://doi.org/10.32614/RJ-2017-009\>.

17. Wang E, Cook D, Hyndman RJ (2020). \"A new tidy data structure to
    support exploration and modeling of temporal data.\" *Journal of
    Computational and Graphical Statistics*, 29(3), 466-478.
    doi:10.1080/10618600.2019.1695624.

18. Wickham H (2016). ***ggplot2**: Elegant Graphics for Data Analysis*.
    Springer-Verlag New York. ISBN 978-3-319-24277-4,
    \<[https://ggplot2.tidyverse.org](https://ggplot2.tidyverse.org/)\>.

19. Wickham H (2023). ***stringr**: Simple, Consistent Wrappers for
    Common String Operations*. R package v1.5.1.
    doi:10.32614/CRAN.package.stringr.

20. Wickham H, Vaughan D, Girlich M (2025). **tidyr**: Tidy Messy Data.
    doi:10.32614/CRAN.package.tidyr.
    \<https://doi.org/10.32614/CRAN.package.tidyr\>, R package version
    1.3.2, \<https://CRAN.R-project.org/package=tidyr\>.

21. Wickham H, Kuhn M, Vaughan D (2025). ***generics**: Common S3
    Generics not Provided by Base R Methods Related to Model Fitting*. R
    package v0.1.4. doi:10.32614/CRAN.package.generics.
