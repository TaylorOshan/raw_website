Further tests with sparse data structure for spatial interaction models 
#######################################################################

:date: 2016-6-10 16:30
:tags: GSOC
:category:
:slug: more-sparse
:summary:

This week the effort to build sparse compatibility for spatial interaciton
models was continued. Rather than write tests, and then change the code and need
to re-write tests, it was decided to finalize the framework and then follow up
with the tests. One of the main bottlenecks for spatial interaction models are
the huge number of dummy variables used by the constrained variants. Therefore,
after conferring with my mentores more effort was put into exloring functions to
generate sparse dummy variables, some of which can be seen here_. Thus far it
has been a sucess, as the ability to produce the diesign matrix with the
approriate dummy variables for a model using ~3000 locations (like the us
counties), which implies 9 million OD flows now takes minutes to estimate rather
than an hour+.

Once the sparse implementation was complete, some tests_ were run to compare the
sparse and dense framework speeds. Obviously, in the constrained variants,
sparse always wins hands down, but for the unconstrained model, dense matrices
are actually faster. Therefore, it was decided that the codebase will adapt
according to which model is being estimated. Finally, the branches containing the desne
and sparse implementation were merged into the glm_spint_ branch, which is the
main working feature branch for this GSOC project. Unfortuantely, the end of the
week was spent resolving some bugs that resulted from the merging of the sparse
and dense frameworks, and I was unable to finish developing unittests for the
general spint framework. This will be the first task next week before adding
features to extend the framework to handle over-dispersed and zero-inflated
data.



.. _here:  https://gist.github.com/TaylorOshan/7845a68dcb10bfeb5319e851c2495c22
.. _tests: https://gist.github.com/TaylorOshan/042c26197b7daeb5d6b47ed025e3d460
.. _glm_spint: https://github.com/TaylorOshan/pysal/tree/glm_spint
