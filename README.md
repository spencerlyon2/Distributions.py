# distcan

Probability **dist**ributions for python in their **can**onical form. Documentation (TODO: link)

`scipy.stats` is the go to library for working with probability distributions in python. It is an impressive package that exposes an *internally consistent* API for working with almost 100 distributions. But, there are some shortcomings...

* Instead of using the common names for the parameters of distributions (e.g. Normal distribution mean and standard deviation being named mu and sigma), `scipy.stats` has keyword arguments (or combinations of them) `loc`, `scale`, and `shape` *assume* the roles of canonical parameters
* Related to the non-conventional parameter naming, the documentation displays expressions for the pdf that often doesn't match the canonical form of the pdf easily found online or in standard references. This makes it difficult to tell exactly what distribution you are working with
* Some distributions are included in `scipy.stats`, but under a different name a different documented form for the pdf. For example, to create an InverseGamma(5, 6) distribution, you would call `scipy.stats.invgamma(5, scale=6)`

## Enter `distcan`

The `distcan` library aims to address these problems in an easily extensible way. Some goals of this project are

* Represent probability distributions in their canonical form, with parameters given their standard names
* Expose an API that is encompasses functionality in `scipy.stats` and [`Distributions.jl`](https://github.com/JuliaStats/Distributions.jl) (a Julia package that motivated the creation of `distcan`), with naming conventions that are consistent for users of both packages
* Have documentation that accurately describes the distribution being used

By leveraging the great code in `scipy.stats`, we are well on our way to completing these goals.

### Adding a new distribution

Adding a new distribution, based on one in `scipy.stats`, is extremely easy. To see just how easy it is, we will consider an example and then walk through how it works. Below is the actual implementation of the InverseGamma distribution from `distcan` (as of 2015-01-08):

```python
class InverseGamma(CanDistFromScipy):                                       # 1

    _metadata = {
        "name": "InverseGamma",
        "pdf_tex": (r"p(x;\alpha,\beta)=\frac{\beta^{\alpha}}{\Gamma(\alpha)}"
                    + r"x^{-\alpha-1}\exp\left(-\frac{\beta}{x}\right)"),

        "cdf_tex": r"\frac{\Gamma(\alpha, \beta / x)}{\Gamma(\alpha)}",

        "param_names": ["alpha", "beta"],

        "param_descrs": ["Shape parameter (must be >0)",
                         "Scale Parameter (must be >0)"],

        "_str": "InverseGamma(alpha=%.5f, beta=%.5f)"}                      # 2

    # set docstring
    __doc__ = _create_class_docstr(**_metadata)                             # 3

    def __init__(self, alpha, beta):                                        # 4
        self.alpha = alpha                                                  # 5
        self.beta = beta

        # set dist before calling super's __init__
        self.dist = st.invgamma(alpha, scale=beta)                          # 6
        super(InverseGamma, self).__init__()                                # 7

    @property                                                               # 8
    def params(self):
        return (self.alpha, self.beta)
```

I have labeled certain lines of the code above. Let's analyze what is happening item by item:

1. Notice that we are subclassing `CanDistFromScipy`. This class is defined in `distcan.scipy_wrap` and does almost all the work for us, including defining methods and setting docstrings for each method.
2. `_metadata` is a dict that is used to set the docstring of the class and  each method as well as the `__str__` and `__repr__` methods. For an explanation of what this dict should contain and how it is used, see the [metadata section](TODO: link) of the docs
3. This line uses the information from the `_metadata` dict to set the docstring for the class
4. The arguments to `__init__` method are the canonical parameters and associated names for the distribution
5. These arguments are stored as attributes of the class
6. Create an internal instance of the distribution, based on the implementation in `scipy.stats`. This is where we map canonical parameter names into the notation from `scipy.stats`
7. Call the `__init__` method of `CanDistFromScipy`. This is where the heavy lifting happens
8. Set an additional property called `params` that returns the parameters of the distribution

To create another distribution based on a distribution found in `scipy.stats`, you simply need to define a class that inherits from `CanDistFromScipy` and implements the 8 points listed above.

### Functionality

All the functionality of `scipy.stats`, plus a few other convenience methods, is exposed by each distribution. This includes the following methods:


* `pdf`: evaluate the probability density function
* `logpdf`: evaluate the log of the pdf
* `cdf`: evaluate the cumulative density function
* `logcdf`: evaluate the log of the cdf
* `rvs`: draw random samples from the distribution
* `moment`: evaluate nth non-central moment
* `stats`: some statistics of the RV (such as mean, variance, skewness, kurtosis)
* `fit` (when available in scipy.stats): return the maximum likelihood estimators of the distribution, given data
* `sf` (also given name ccdf): compute the survival function (or complementary cumulative density
function)
* `logsf` (also given name logccdf): compute the log of the survival function (or complementary cumulative
density function)
* `isf`: compute the inverse of the survival function (or complementary
cumulative density function)
* `ppf` (also give name quantile): compute the percent point function (or quantile), which is the inverse
of the cdf. This is commonly used to compute critical values.
* `loglikelihood` (not in scipy): the loglikelihood of the distribution with respect to all the samples
in x
* `invlogcdf` (not in scipy): evaluate inverse function of the logcdf
* `cquantile` (not in scipy): evaluate the complementary quantile function. Equal to `d.ppf(1-x)` for
x in (0, 1). Could be used to compute the lower critical values of a
distribution
* `invlogccdf` (not in scipy): evaluate inverse function of the logccdf

Additionally, each distribution has the following properties (accessed as `dist_object.property_name` -- i.e. without parenthesis):

* `mean`: the mean of the distribution
* `var`: the var of the distribution
* `std`: the std of the distribution
* `skewness`: the skewness of the distribution
* `kurtosis`: the kurtosis of the distribution
* `median`: the median of the distribution
* `mode`: the mode of the distribution
* `isplaykurtic`: boolean indicating if kurtosis is greater than zero
* `isleptokurtic`: boolean indicating if kurtosis is less than zero
* `ismesokurtic`: boolean indicating if kurtosis is equal to zero
* `entropy`: the entropy of the distribution
* `params` (not in scipy): return a tuple of the distributions parameters

## Distributions currently implemented

Univariate-continuous:

* Chi
* Chisq (also called Chi2)
* F (also called FDist)
* Gamma
* InverseGamma
* LogNormal
* Normal
* NormalInverseGamma
* T (also called TDist)
* Uniform

Multivariate-continuous


* MultivaraiteNormal

Matrix Variate:

* Wishart
* InverseWishart

## Contributors

* Spencer Lyon (spencer.lyon@stern.nyu.edu)
* Chase Coleman (cc7768@gmail.com)

