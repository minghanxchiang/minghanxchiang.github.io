+++
date = '2024-12-07T15:42:54+08:00'
draft = true
tags = ['Causal Inference', 'Natural Experiment', 'Econometrics']
title = 'Introduction to Regression Discontinuity Design - Lessons from an Onboarding Project (1/2)'
description = "This article lays the groundwork for understanding Regression Discontinuity Design (RDD), a robust causal inference method, by examining its core assumptions and setting the stage for a follow-up on conducting the analysis and interpreting the results."
excerpt = "This article lays the groundwork for understanding Regression Discontinuity Design (RDD), a robust causal inference method, by examining its core assumptions and setting the stage for a follow-up on conducting the analysis and interpreting the results."
+++


*This article lays the groundwork for understanding Regression Discontinuity Design (RDD), a robust causal inference method, by examining its core assumptions and setting the stage for a follow-up on conducting the analysis and interpreting the results.* 

Regression Discontinuity Design (RDD) has long been recognized as *the identification strategy* for precisely estimating treatment effects in non-experimental setups. When a project satisfies all the assumptions and requirements, RDD performs almost as well as a randomized control trial (RCT). However, unlike online RCTs, RDD often receives limited attention in the IT industry. For major social platforms and app-based products, running standardized, fast-paced, and stratified experiments on a subsample of users is relatively cheap[^1]. It’s understandable that they often overlook RDD for impact evaluation.

For small-to-medium-sized startups, though, RDD offers a valuable lens for analyzing historical data. These organizations often roll out changes or release new builds in ways that unintentionally satisfy the requirements for a robust RDD study.

RDD is termed a "design" for a reason. It requires a specific *you-either-have-it-or-you-don’t* quality inherent in a policy or program, rather than being a strategy one can apply universally. The essence of this method lies in identifying a strict, exogenous *rule-defining cutoff*—be it age, time, income, or another continuous variable—that determines treatment assignment. Analyzing data points around this cutoff mimics a tiny RCT because the cutoff introduces a degree of randomness for participants near it.

---

## A Running Example

Let’s explore a typical scenario in a growing startup. People are often curious about the net impact of a design change or want to understand whether past decisions were effective. Unfortunately, proper experimental designs may be absent or incorrectly implemented.

In my case, a teammate approached me to investigate a decline in the user onboarding conversion rate. By tracking the conversion rate over time and reviewing recent onboarding process changes, I identified a key alteration: the addition of a button to pair our hardware product with users' mobile devices.

At the time this button was introduced, our hardware product had not been officially released. This button effectively directed new users to a dead end. Promoting hardware products was not a focus of our acquisition strategy, and most new users were unaware of the hardware ecosystem when first using our app. If I had been a new user, I would likely have been confused as well.

![targets](/img/post_img/2024-12-07_RDD_onboarding_pt1/new_lay_out.png "This is an AI generated illustrative layout, we changed from something like the right one to something like the left one")

This led to my hypothesis: the button impaired the onboarding process, discouraging users from completing it and becoming valid users.

“Why RDD?” you might ask. This design change was launched without any randomization, ruling out the possibility of using ordinary least squares (OLS) regression or confounder-free instrumental variables (IV). It also lacked control units, eliminating the difference-in-difference (DiD) approach. However, the build with this change was released—like many app updates—without prior notice. This gave us an exogenous cutoff to explore further.

---

## Method Overview: What's RDD and Why Fuzzy RDD?

RDD includes two specifications: *Sharp RDD* and *Fuzzy RDD*. For this analysis, I used the latter. If you're unfamiliar with these methods, I recommend reading [[Lee and Lemieux (2010)](https://www.princeton.edu/~davidlee/wp/RDDEconomics.pdf)]. In brief, sharp RDD applies when the running variable’s discontinuity solely determines treatment assignment[^2], while fuzzy RDD is used when the discontinuity only affects the probability of treatment assignment. Despite these differences, *both sharp and fuzzy RDD share the same set of assumptions*, making their implementation dependent on meeting these foundational requirements.

In this example, the running variable is the time when a user first signed up, with the build’s release time serving as the cutoff. Since the global staged rollout of the update increased users’ probability of exposure to the treatment from 0% to a value between 0% and 100%, fuzzy RDD was the appropriate framework.

The essence of fuzzy RDD is to *partial out potential confounders using an exogenously established cutoff* as an instrumental variable (IV). Ideally, the cutoff possesses the key characteristics of a valid IV: *relevance* and *exclusion restriction*.

- **Relevance:** This is inherent because the discontinuity directly affects treatment assignment.
- **Exclusion Restriction:** This is validated by examining coinciding treatments, density, and smoothness.


---

## Assumptions for RDD

### Coinciding Treatments

This assumption addresses whether other factors simultaneously affect the outcome of interest. Overlooking this is a common mistake. Analysts should verify the specifics of the build in team communication channels or documentation.

In my example, the only change to the onboarding process was the new layout. Since the build passed quality assurance, I could safely assume no other coinciding treatments occurred during this update.

### Density

For RDD to be valid, participants must not be able to manipulate their position relative to the cutoff. If individuals can strategically place themselves on one side of the cutoff, the exogeneity assumption breaks down, and causal inference is compromised. For example, an airline loyalty program offering lounge access after 50,000 miles might encourage someone with 49,800 miles to take an extra trip, making the cutoff endogenous.

While statistical tests are valuable, *having a strong argument for why manipulation is unlikely is equally, if not more, critical.* Before diving into tests, analysts should carefully consider the context of their data and the specific policy or change being evaluated. In my case, the cutoff was determined by the timing of a build release—something users couldn’t influence or anticipate. This contextual understanding strengthens confidence in the assumption that no manipulation occurred.

To supplement this reasoning, statistical methods can be employed to detect potential manipulation. One common approach is to examine the density of the running variable around the cutoff. If individuals cannot anticipate or influence the cutoff, the distribution of data points near the threshold should be smooth. Plotting this distribution allows analysts to visually inspect for any irregularities, such as sparse or dense regions that could indicate manipulation. 

The **local polynomial density test**, developed by a team of cross-institutional researchers, is a more rigorous method for evaluating density smoothness. This non-parametric test estimates the "jump" in the density at the cutoff and comes with tools for bandwidth selection and hypothesis testing. The approach involves using kernel structures and bandwidth optimizations to assess the entire support of the running variable[^3]. For my project, I tested the density of sample points within a two-day interval around the cutoff. The test failed to reject the null hypothesis of no manipulation, further supporting the validity of the RDD design.

While tests are useful, they are not a substitute for context-specific reasoning. A combination of thoughtful arguments and robust testing ensures the reliability of an RDD analysis.

### Smoothness

Another critical assumption for a valid RDD is that no factors other than the running variable exhibit discontinuity at the cutoff. If covariates—observable characteristics that might influence the outcome—are not smooth across the cutoff, it becomes impossible to attribute any observed treatment effect solely to the policy or intervention under study.

The conventional approach to addressing this assumption is to test for discontinuities *exhaustively* across all observable covariates. While it’s impractical to test for every conceivable confounder, examining all collected or collectable variables provides a solid foundation for validating the smoothness assumption. This step ensures that the cutoff doesn’t inadvertently align with changes in other factors that could influence the outcome.

When performing these tests, it is crucial to explore various functional forms for the relationship between covariates and the running variable. Analysts should test parametric forms, such as linear or polynomial models, as well as non-parametric approaches like local polynomials. Using diverse specifications helps ensure robustness and guards against misleading results caused by choosing an inappropriate functional form.

For instance, in my project, I tested for discontinuities in all key observables using both linear and polynomial models, as well as non-parametric methods. A typical specification for a linear spline model is:

$$
X_{i} = \alpha + \beta (rv_{i} - c) + \gamma \ \textbf{I}(rv_{i} \geq c) + \delta (rv_{i} - c)*\textbf{I}(rv_{i} \geq c) + \epsilon_{i}
$$

Here \(rv_{i}\) is the value of the running variable, \(c\) is the cutoff, and \(\textbf{I}(\cdot)\) is an indicator function for whether the running variable exceeds the cutoff. Non-parametric methods, such as local polynomial estimation, allow for greater flexibility and often include automatic bandwidth selection to capture nuances in the data.

In my tests, none of the observable covariates showed significant discontinuities across the cutoff, regardless of the method used. This consistency across parametric and non-parametric forms provided strong evidence that the smoothness assumption held, granting further credibility to the RDD results.


---

In conclusion, RDD, particularly fuzzy RDD, can be a powerful tool for startups to draw causal insights from historical data. This project highlights how the method’s assumptions and requirements, when met, allow analysts to explore causal relationships even in non-experimental settings.

[^1]: As experiments become more frequent in an organization, it’s crucial to consider not only estimating the Average Treatment Effect (ATE) but also the implications for false positive rates and statistical power. For further insights, see Kohavi, Deng, and Vermeer (2022): [A/B Testing Intuition Busters](https://bit.ly/ABTestingIntuitionBusters).

[^2]: Medicare enrollment or Taiwan's National Pension enrollment are excellent examples. In Taiwan, citizens aged 25 and above are mandatorily and automatically enrolled in the national pension system, with no exceptions. This can be expressed mathematically as \(\textbf{P}(T_{i} = 1 | age_{i} \geq 25 ) = 1\).

[^3]: The manipulative test framework in the referenced paper primarily addresses cases where the running variable has finite support (e.g., the margins of a senate election with support \([-100, 100]\)). For scenarios with indefinite support, such as the example in this series, I focused on a relevant subset — specifically, the interval of one day before and after the update launch—to ensure practical and meaningful analysis.
