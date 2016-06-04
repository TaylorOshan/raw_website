Generalized Linear Models and Sparse Design Matrices
####################################################

:date: 2016-6-3 16:12
:tags: GSOC
:category:
:slug: sparse_glm
:summary:

This week was primarily focused on exploring different options for estimating generalized linear models (GLM). First, I built an option into the gravity model class that would use exisitng GLM code through the statmodels project. Next, I put together my own code to carry out iteratively weighted least squares estimation for GLM's. Of immediate interest was a comparison of the speed of the code-bases for carrying out the estimation of a GLM. Each technique was used to calibrate each model variety (unconstrained, production-constrainedm atttraction-constrainem, and doubly constrained) when the number of origin-destination pairs (sample size) was N = 50, 100, 150, and 200. Some quick results can be see here in this gist_. Expectedly, the results show that for either technique the unconstrained models are the fastest to calibrate (no fixed effects), followed the by the singly-constrained models (N fixed effects), and finally, the doubly-constrained models take the longest (N*2 fixed effects). The more fixed effects in the model, the larger the deisgn matrix, X, and the longer the estimation routines will take. More surprisingly, the custom GLM (from now on just GLM) was way faster than the statsmodels GLM (from now on SMGLM). The statsmodels uses either the pseudoinverse or a QR decomposiiton to compute the least sqaures estimator, which would have been thought to be way quicker than the direct computations used in my code. For now, I have only tested the pseudo-indverse SMGLM, as the flag to switch to QR decomposition is not actually available to the SMGLM api (on to the weight least sqaures class at a lower level of abstraction). Perhaps, the SMGLM takes longer to compute because it has a fuller suite of diagnostics or perhaps the psuedo-inverse is not as quick when there is a sparse design matrix (i.e., many fixed effects).

After some additional exploring, I found that there was an upper limit to the
number of locations (N = locations**2) that could considered for the
singly-constrained or doubly-constrained models given the high number of fixed
effects. Somewhere between 1000 and 2000 locations, my notebook would run out of
memory. To cirvument this, I next developed a version of the custom GLM code
that was compatible with the sparse data strcutres from the Scipy library, which
cna be found in this branch_. This required a custom version of the
categorical() function from statsmodels (exampe here_) to create the dummy variables needed for constrained spatial interaction models and then altering the least squares computations to make sure that no large dense arrays were being created throughout the routine. Specifically, it was necessary to change the order in which some of the operations were carried out to avoid the creations of large dense arrays. Now it is possible to calibrate constrained spatial interaction models using GLM's for larger N. The most demanding model estimation I have tested_ was a doubly-constrained model with 5000 locations, which implies N=25,000,000 and a design matrix with the dimensions of (25,000,000, 10001). Looking forward, I will further test this sparse GLM framework to see if there
are any losses assocaited with it when N is small to moderate. 

This week I also explored gradient optimization packages that coyld be used in
lieu of a GLM framework. So far the options seem to be between autograd/scipy or
Theano. I was able to create a working example_ in autograd/scipy, though this
has not been developed any further yet and will likely be pushed off until later
in the project. For now, the focus will remain on using GLM's for estimation.

Next week I will begin to look at diagnostics for count models, zand
alternatives when we have overdispersed/zero-inflated dependent variables.

.. _gist: https://gist.github.com/TaylorOshan/ccac7e5489b5b2b1799bf79ef001d922
.. _branch: https://github.com/TaylorOshan/pysal/tree/sparse_glm_spint/pysal/contrib/spint
.. _here: https://gist.github.com/TaylorOshan/45cd01bb08e23280549aee770a05cdfe
.. _tested: https://gist.github.com/TaylorOshan/42d90dbf219b50f3b0d54e06ba4e8b5b
.. _example: https://gist.github.com/TaylorOshan/13c3b4a1d9489e03ea70ee52ecb0b61d
