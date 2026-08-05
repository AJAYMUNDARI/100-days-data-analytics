#Problem Statement: Let’s say we want to launch a re-design of a landing page to improve the click-through rate. We can do this by implementing an AB test. Given that we launch an AB test, how would you infer if the results of the click-through rate were statistically significant or not?

#A/B test experimental set up 

#1. Define primary metric
   a. North start metric/primary metric: click-through rate
   b. Guadrill metric: revenue generated

#2. Set thresholds 
    alpha=> Exceptable 
    Type I error=> 0.05 
    Power=> Probability of seeing true effect=> 0.8 
    MDE=> Minimum effect size needed=> generally set by stakeholders 
    
#3. Sample size and Duration Based on alpha, power and MDE calculate sample size and based on smaple size calculate duration, make sure duration to be atleast 2 weeks 

#4. Group Assignment Split data into two groups, make sure groups are randomised and balanced. If there is network effect go for cluster sampling otherwise prefer stratified sampling.

After A/B test is setup, 
define Null hypothesis and alternate hypothesis 
  H0: pa-pb = 0 
  H1: pa-pa <> 0 
  
  now calculate test statistic Z = (pa - pb)/sqrt(pa(1-pa)/na + pb(1-pb)/nb) based on Z-value find p-value If p-value is less than alpha, then we can reject the null hypothesis, and conclude two groups difference is statistically significant. 
  Consider effect size, guardrail metrics (like revenue), and implementation cost before rolling out changes.
