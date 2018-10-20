## Education Background

#### Wuhan University (09/2015-06/2019)

*Bachelor of Science in **Statistics**, School of Mathematics and Statistics*

Major curriculum: Probability, Mathematical Statistics, Multivariate Statistical Analysis, Applied Regression Analysis, Stochastic Processes, Functional Analysis, etc.

GPA: 89.0/100 (3.72/4.00)

## Honors and Awards

09/2018 (2017-2018 Academic Year) Merit Student Scholarship of Wuhan University (TOP 15%)

02/2018 H Award in American Undergraduate Mathematical Contest in Modeling (MCM)

09/2016 (2015-2016 Academic Year) Merit Student Scholarship of Wuhan University (TOP 15%)

11/2015 Excellent Freshman Scholarship of Wuhan University (TOP 15%)

## Project Experience

#### Proposed a Mapping to Divide Given Points into Group (06/2018)                              

* Considered *labeled data* in $\mathbb{R^2}$ : some are in category *A*, the rest are in category *B*. Aimed to construct a transformation that took any points in  $\mathbb{R^2}$ to return either category *A* or *B*.
* Constructed such transformation based on artificial neural network.
* Use MATLAB to train this network: initialize all weight and biases using randn and set a constant learning rate; use the iteration summarized above and store the value of cost function as each iteration; use the semiology command to visualize the process of iteration.
* **semiology** Plot shows cost undergoes a flood period towards the start of process; After this plateau, the cost decayed at a very slow linear rate. When extra data point is added, after repeating this process, the other points almost remain in the same groups as before.

#### Parameter Estimation of Finite Normal Mixture Samples (05/2018)

* Aimed to use *EM algorithm* to estimate unknown parameters of finite normal mixture like: $f(x\|\theta)=\sum^k_{i=1}P_if_i(x)$, where $P_i > 0$, $\sum^k_{i=1}P_i=1$ and $f_i(x)$ is the density of $N(\mu_i,\sigma^2)$; here $(P, \mu, \sigma^2)$ is parameter that needs estimation.
* By introducing latent variable $z$, applied *EM algorithm* to get iteration of unknown parameters.
* Based on data that chose from random finite normal mixture, got the estimation of unknown parameter by R. 
* Estimation mentioned above showed less than 5% relative errors compared with real data; Though the number of iterations was kind of big, about 1500~3000 times, the cost of iteration is relatively low and running time is less than a minute.
#### Preconditioned Conjugated Gradient through MATLAB (11/2016)
* Considered a method to *precondition* n-by-n symmetric positive definite linear system $Ax=b$ so that the matrix of coefficient assumed well conditioned or had just a few distinct eigenvalues.
* Transformed the system above into $\mathbf{A}\tilde{x}=\tilde{b}$, where $\mathbf{A}=C^{-1}AC^{-1}$, $\tilde{x}=Cx$ , $\tilde{b}=C^{-1}b$, $C$ was a symmetric positive definite; tried to choose proper $C$ by an *Incomplete Cholesky Factorization* of $A$.
* Applied *Conjugated gradient* to transformed system and get the method called *Preconditioned Conjugated Gradient*.
* Tested this algorithm by MATLAB: Produce a random positive definite linear system $Ax=b$ through MATLAB; Based on the same number of iterations, used MATLAB to compare the running time and relative error before *preconditioned* with those after *preconditioned*.
* The result from *Preconditioned Conjugated Gradient* shows 28% ~60% fewer relative errors than that from *Conjugated Gradient*, depending on the order of $A​$ and the number of iterations. 

## Internship Experience

#### China Logistic Information Index Co., Ltd, Manager Assistant (08/2018-09/2018)

* Participated in PMI investigation and assisted in distributing over 700 questionnaire surveys. 
* Gathered survey results and calculated related PMI indexes; adjusted the data in accordance with the season; calculated the correlation coefficient among PMI indexes using R. 
* Compared the calculated indexes with official data released by National Bureau of Statistics to validate the results; created index variation diagram and analyzed the index fluctuation reasons.

#### China Logistic Information Center, Assistant (09/2018-10/2018)

- Gathered the logistics information report from each department and calculated the China Highway Freight & Logistics Weekly Index; created index variation diagram. 
- Analyzed the fluctuation of the weekly indexes and wrote a corresponding report, which got published on the official website of China Logistic Information Center. 
- Assisted in completing China Logistic Enterprise Business Environment Report. 
- Participated in significant meetings such as *Transportation, Energy and Yangtze River Economic Belt Situation Analysis Session*.

## On-campus Activities

#### Summer Vacation Social Practice, Associate Team Leader (07/2016-08/2016)

* Conducted on-spot visits to the ruins of ancient buildings in Xi’an, Shaanxi and distributed over 300 questionnaire surveys.
* Applied factor analysis and linear regression analysis to summarize ancient building current preservation situation and its deficiencies.

#### Women’s Volleyball Team of school of Mathematics and Statistics, Team Leader (09/2017-06-2018)

* Firth place of 2018 WHU Torch Cup Women’s Volleyball Competition

## Skills

#### Language skills

* TOEFL: 102 (L: 29, R: 26, W: 25, S: 22) - May 6, 2018
* GRE: 320(V: 152, Q: 168, W:3.5) - September 1, 2018
* Common European Framework of Reference for Languages (French A1)

#### Professional skills

proficiency in applying technologies including R, SAS, MATLAB; have a good command of C Language and Python

## Personal Information
DOB: October 15, 1997
Email: chloe.zhao1015@gmail.com
Tel.: +86 18991305181 Add.: Xinxilan Community, No. 129, Science and Technology Road, Yanta District, Xi'an, Shaanxi, China