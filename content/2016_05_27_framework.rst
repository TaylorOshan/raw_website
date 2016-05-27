SpInt Framework
###############

:date: 2016-5-27 8:45
:tags: GSOC
:category:
:slug: framework
:summary:

Given the API deisgn discussed in the previous post, the initial few days of
coding were used to build the general framework and core classes that will be
used in SpInt. Since the gravity-type models make up the majority of existig
model specifications, the initial focus for developing the general framework
essentially means focusing on developing classes for gravity models.

It was decided to split the four primary model specfications (unconstrained or
traditional gravity model, production-constrained, attraction-constrained,
doubly-constrained) into four separate "user" style classes (see pysal.spreg
module) that inherit from one base class (BaseGravity). These user classes
serve to accept structured array inputs and configurations from the user and
then carry out model-specific checks of the data. Then they pass the inputs into
an __init__ method for BaseGravity. Depending on which inputs are passed and the
type of user class that __init__ is called from, BaseGravity then further checks the
data and preapres it into an endogenous dependent variable, y, and and a
set of exogenous explanatory variables, X. Finally, BaseGravity then calls an
__init__ method to the class that it inherits from, CountModel, with these newly
formatted inputs and any estimation options, and a fit() method. The init_method
carries out a simple check to make sure the dependent variable, y, is indeed a
count-based variable and the fit method carries out the selected estimation
routine. Currently the default estimatation framework/method is MLE using the generalized linear model (GLM)/iteratively re-weighted least squares (IWLS) in statmodels,
though others can be added such as GLM/gradient search or non-linear
formulations/gradient search. 

The design was carried out in this manner (CountModel --> BaseGravity -->
Gravity/Production/Attraction/Doubly) in order to promote flexibility and modularity.
For example, CountModel can be expanded by adding tests for overdisperison,
additional estimation routines and count-based probability models other than
Poisson (i.e., negative binomial), which can be useful throughout pysal for other count-based
modeling tasks. This also means that SpInt is not a priori limited to any single
type of estimation technique or probability model in the future. Then the BaseGravity
class does the heavy lifting in terms of data munging and common data integrtiy
checks. And finally, the user classes aim to restrcit input for specific types
of gravity models, carry out model-specific checks, and then to organize and
prepare the model results for the user. 

This code and an exmple of its use in an ipython notebook can be found here_. At
the moment estimation can only be carried out using a GLM/IWLS in statmodels and
only basic results are available, though a more user-friendly presentation of
the results will be created, such as the results.summary() method in
statsmodels. 

Looking forward, this framework will be filled out with the additons already
described as well potentially adding a mechanism that allows users to flexibly
use different input formats. Currently, input consists of all arrays, but it
would be helpful to allow users to pass in a pandas DataFrames and the names of the columns that refer to different arrays. I will also begin
to explore sparse data structures/algebra and when they will be the most
beneficial to employ.


.. _here: https://github.com/TaylorOshan/pysal/blob/glm_spint/pysal/contrib/spint
