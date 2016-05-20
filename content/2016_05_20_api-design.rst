Initial GSOC meeting - API Design
#################################

:date: 2016-5-20 11:23
:tags: GSOC
:category:
:slug: api-design
:summary:

During the community bonding period of the GSOC program I have been thinking
largely about the design of the API for SpInt. 

First, the API would need to accommodate a basic gravity model and then also support
extensions that include SAR terms, a spatial filter term, or a competing
destinations term. Typically the competing destinations term is nothing more
than a sum of distance weighted attributes, which should be simple. I think it
shouldn't be too hard use the existing pysal spatial weights module as a base to
create a function to create OD-based SAR spatial weights. And there is some code
available for computing spatial filters in python_. So I think in order to accommodate the three
extensions it might be simplest to have the user first compute these and then
pass them in  as optional arguments to the basic gravity model. In the case of
the competing destinations or spatial filter model, there is no new estimator
required and so its as easy as including the extra variable (within an OLS or
Poisson regression framework). If the optional SAR term is passed in, then under
the hood a new estimator could be used rather than the default OLS or  Poisson
estimator. I imagine the basic call could look something like 

spint.gravity(required_arguments, cd=None, sf=None, w=None)

where cd would be an optional competing destinations variable, sf the optional
spatial filter term, and w would be optional spatial weight, each of which would
be pre-computed using code from the weights module or other utility functions.

The other alternative could be to have separate calls for each extension where
the corresponding optional term now becomes required:

spint.gravity(required_arguments)
spint.gravity.cd(required_arguments)
spint.gravity.sf(required_arguments)
spint.gravity.lag(required_arguments)

I am inclined to think the former is simpler, esepcially if one would like to
combine models.

Another layer to the API would be each "member" to the Wilson-type "family" of
models: unconstrained or basic gravity model, production or origin constrained,
attraction or destination constrained, and doubly constrained. This is achieved
technically by adding in either balancing factors or fixed effects during
estimation, depending on the estimation technique. Then building from the first
two examples the API could look like either:

(1)

spint.gravity.unconstrained(required_arguments, cd=None, sf=None, w=None)

spint.gravity.production(required_arguments, cd=None, sf=None, w=None)

spint.gravity.attraction(required_arguments, cd=None, sf=None, w=None)

spint.gravity.doubly(required_arguments, cd=None, sf=None, w=None)

(2)

spint.gravity(required_arguments, constraint=none)

spint.gravity.cd(required_arguments, constraint=none)

spint.gravity.sf(required_arguments, constraint=none)

spint.gravity.lag(required_arguments, constraint=none)

I think that option (1) may be more intuitive because the
doubly-constrained model is unique in that it requires a squared OD matrix (all
origins are also destinations) and so it might be natural since each of the four
varieties may have different input properties that need to be checked. 

In terms of model fitting techniques, I think it would be neat to use something
similar to stats models like:

model = spint.gravity.unconstrained(required_arguments, cd=None, sf=None,
w=None)

results = model.fit(fit_arguments)
 
which would be natural if everything is being built onto of a GLM framework. The
reasoning behind using a GLM framework was to be able take advantage of
additional count models such as negative binomial relatively easily.
"fit_arguments" could include the type of probability model (gaussian, poisson,
negative binomial, etc.), and the fit technique (iteratively re-weighted least
squares or Theano/Autograd MLE). It could then be possible to build a higher
level of abstraction on top of this that more closely matches the base/user
class organization used in the spreg module within pysal.  

Aside from gravity models, there are other spatial interaction models, but that I sort of think of as separate
from those described above because they are non-parametric methods, which are either
deterministic "universal" models that mostly come out of the human mobility
literature or use a neural network approach. I think it would
make sense to accommodate non-parametric  models so that they are separate from
gravity models. For example:

spint.mobility.radiaton()

spint.mobility.inv_pop_weighted()

spint.neural()

To summarize, the API needs to accomodate the "family" of spatial interaction
model varieties (unconstrained, production-constrained, attraction-constrained,
doubly-constrained), where each of these varieties can then have several
extensions to include a competing destinationas terms, a spatial filtering term,
or a SAR term. The API also needs to acommodate non-parametric methods such as
dterminstic models and neural network based models. Framework (1) from above
will be adopted first as the main API. 

During the first GSOC meeting with my mentors we decided it would be best to
first get as many model variations working as possible. Then we could tweak the
API accoridngly and focus on optimizing the computational speed. We've also set
up a Trello board so that we can collaborate on assigning/managing tasks for the
project. 

Finally, I have started a wiki page_ on the pysal group on Github where it will
be possible to discuss model specifications and their related literature, such
as the Poisson SAR lag model.

.. _Python: https://github.com/bchastain/esf

.. _page: https://github.com/pysal/pysal/wiki/SpInt-Development
