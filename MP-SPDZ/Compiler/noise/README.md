# The Discrete Gaussian for Differential Privacy

This repository consists of code for the following paper.

>[Clément Canonne, Gautam Kamath, Thomas Steinke. _The Discrete Gaussian for Differential Privacy_. 2020.](https://arxiv.org/abs/2004.00010)

In particular, the plots in the paper are generated by the script `plots.py`. The code is implemented in Python 3.

We hope that the sampling code and other utilities will be useful to others.

## Sampling

The main file is `discretegauss.py`. This includes the exact sampling methods.

The function `sample_dgauss(sigma2)` will generate one sample from the discrete Gaussian. Note that the argument to the function is the square of the scale, rather than the scale parameter. 

To generate a sample from the discrete Laplace distribution use `discretegauss.sample_dlaplace(scale)` instead.

Both functions use [`random.SystemRandom()` ](https://docs.python.org/3/library/random.html#random.SystemRandom) to obtain high-quality randomn bits (on unix this should use `/dev/urandom`). A different random number generator may be specified via the optional argument `rng`. The parameters sigma2 and scale are cast to a rational number ([`fractions.Fraction`](https://docs.python.org/3/library/fractions.html)) to allow exact arithmetic.

_Testing:_ Simply executing `python3 discretegauss.py` will run a basic test. A more thorough and sophisticated test is provided by the script `testing-kolmogorov-discretegaussian.py`, but be warned that this will take several hours to run.

## Differential Privacy

The file `cdp2adp.py` contains utilities for analyzing the differential privacy properties of the discrete Gaussian and, more generally, algorithms satisfying concentrated differential privacy. Unlike the sampling methods, these functions use floating point arithmetic and are, therefore, not exact.

The function `dg_delta(sigma2,eps)` computes the smallest delta such that adding discrete Gaussian noise with parameter sigma2 to a sensitivity-1 function (e.g., a counting query) provides (eps,delta)-differential privacy; `cg_delta(sigma2,eps)` performs the equivalent calculation for continuous Gaussian noise.

More generally, `cdp_delta(rho,eps)` computes a delta such that any algorithm satisfying rho-concentrated differential privacy satisfies (eps,delta)-differential privacy. (Recall that adding discrete or continuous Gaussian noise with parameter sigma2 to a sensitivity-1 function provides rho-concentrated differential privacy for rho=1/(2*sigma2).) For comparison, the standard bound `cdp_delta_standard(rho,eps)` is also provided.
Alternatively, given rho and delta, `cdp_eps(rho,delta)` computes a eps value such that rho-concentrated differential privacy implies (eps,delta)-differential privacy. Similarly, `cdp_rho(eps,delta)` computes a rho such that rho-concentrated differential privacy implies (eps,delta)-differential privacy.

## Example

The following example code shows how to release several counts in a (1,10^-6)-differentially private manner.

```python
import math
from fractions import Fraction

import discretegauss
import cdp2adp

#load some data, one line = one person
data = [x for x in open("mydata.txt")]
#define some queries, count the number of rows that contain search strings
search_strings = ["Asparagus","Broccoli","Carrot"]

#set overall DP parameters
eps=1
delta=1e-6
#convert to concentrated DP
rho=cdp2adp.cdp_rho(eps,delta)
print(str(rho)+"-CDP implies ("+str(eps)+","+str(delta)+")-DP")
#number of queries
k=len(search_strings)
#divide privacy budget up amongst queries
#Each query needs to be (rho/k)-concentrated DP
#cast to Fraction so subsequent arithmetic is exact
rho_per_q = Fraction(rho)/k 
#compute noise variance parameter per query
sigma2=1/(2*rho_per_q)
#actual variance, at most sigma2
var = discretegauss.variance(sigma2)
print("standard deviation for each count = "+str(math.sqrt(var)))

#evaluate queries and privatize
for q in search_strings:
    #count number of people whose data string contains term q
    true_ans = sum(1 if q in x else 0 for x in data)
    assert isinstance(true_ans,int) #important that it is rounded to an integer
    #sample noise and add
    noise = discretegauss.sample_dgauss(sigma2)
    privatized_ans = true_ans + noise
    #output
    print(str(privatized_ans)+"\t"+q)
```

## Disclaimer

This is research code.

## Licence

[Apache](https://www.apache.org/licenses/LICENSE-2.0)

© Copyright IBM Corp. 2020