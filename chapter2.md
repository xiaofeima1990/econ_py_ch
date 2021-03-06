<script type="text/javascript"
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

MCMC STUDY  
========================================================
#  Monte Carlo Integration



# Importance sampling
problem we want to know the aproximate result of [-5,5] rather than the whole range of the distribution. from the brute force draw. it is very inefficient. So we use an instrumental distribution to **truncate** the distribution which raise the efficiency. 


For instance, if we consider a distribution with support restricted to $(4:5; 1)$,
the additional and unnecessary variation of the Monte Carlo estimator due to
simulating zeros (i.e., when $x < 4.5$) disappears. A natural choice is to take g as
the density of the exponential distribution $exp{1}$ truncated at $4.5$,

$$g(y)=e^{-y}/\int^{\infty}_{4.5}e^{-x}dx=e^{-(y-4.5)}$$

and the corresponding importance sampling estimator of the tail probability is

where the $Y_i$ 's are $i.i.d$ generations from $g$. The corresponding code is


```r
Nsim=10^3
y=rexp(Nsim)+4.5
weit=dnorm(y)/dexp(y-4.5)
plot(cumsum(weit)/1:Nsim,type="l")
abline(a=pnorm(-4.5),b=0,col="red")
```

![plot of chunk unnamed-chunk-1](figure/unnamed-chunk-1-1.png) 

```r
mean(cumsum(weit)/1:Nsim)
```

```
## [1] 3.559918e-06
```

```r
pnorm(-4.5)
```

```
## [1] 3.397673e-06
```



MCMC Methods: Gibbs and Metropolis


## MH

### theratical 



### R programming 


**simulating from a normal with zero mean and unit variance(normal is the given distribution)  using a Metropolis algorithm with uniform proposal distribution(uniform is the conditional density)**

```r
MH_norm=function (n, alpha) 
{
  vec= vector("numeric", n)
  x= 0
  vec[1]= x
  for (i in 2:n) {
    innov= runif(1, -alpha, alpha)
    can= x + innov
    aprob= min(1, dnorm(can)/dnorm(x))
    u= runif(1)
    if (u < aprob) 
      {x= can;} 
    
    vec[i] = x
  }
  return(vec)
}
normvec= MH_norm(1000,1)
```
So, innov is a uniform random innovation and can is the candidate point.aprob is the acceptance probability. The decision on whether or not to accept is then carried out on the basis of whether or not a U(0,1) is less than the acceptance probability. We can test this as follows.


```r
plot(normvec,cex=.1,lwd=2,lty=2)
lines(normvec,lwd=2,col="orange3")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

## Gibbs Sampling
The basic idea is to split the multidimensional $\theta$ into blocks (often scalars) and sample each block separately, conditional on the most recent values of the other blocks

Suppose we have a joint distribution p(\theta1, . . . , \thetak ) that we want to
sample from. We can use the Gibbs sampler to sample from the joint distribution
if we knew the **full conditional** distributions for each parameter

For each parameter, the full conditional distribution is the
distribution of the parameter conditional on the known information
and all the other parameters $p(\theta_j| \theta_{-j} , y )$

Question: How can we know the joint distribution simply by knowing the full
conditional distributions?

Suppose we have a joint density $f(x, y)$. The theorem proves that we can write out the joint density in terms of the conditional densities $f(x|y)$ and $f(y|x)$

$$ f(x,y)=\frac{f(y|x)}{\int \frac{f(y|x)}{f(x|y)}dy }$$
which  
$$\int \frac{f(y|x)}{f(x|y)}dy = \int \frac{f(y)}{f(x)} dy=\frac{1}{f(x)}$$

Then RHS is 
$$\frac{f(y|x)}{\frac{1}{f(x)}} = f(y|x)f(x) $$

### Steps to Calculating Full Conditional Distributions

Suppose we have a posterior $p(\theta|y)$. To calculate the full conditionals for each \theta, do the following: 

1. Write out the full posterior ignoring constants of proportionality.
2. Pick a block of parameters (for example, $\theta_1$) and drop everything that doesn’t depend on $\theta_1$.
3. Use your knowledge of distributions to figure out what the normalizing constant is (and thus what the full conditional distribution $p(\theta_1 | \theta_{−1}, y)$ is).
4. Repeat steps 2 and 3 for all parameter blocks.


Let’s suppose that we are interested in sampling from the posterior $p(\theta|y)$(we know the form of distribution), here $\theta$ consists of $k$ blocks $\theta_1, \theta_k, ... ,\theta_k$ . Here we assume $\theta$ is a vector of three parameters, $\theta1$, $\theta2$, $\theta3$.
The steps to a Gibbs Sampler (and the analogous steps in the MCMC process) are
1. Pick a vector of starting values $\theta^{0}$. (Defining a starting distribution $\Pi^{0}$ and drawing $\theta^{0}$ from it.)

2. Start with any $\theta$ (order does not matter, but I’ll start with $\theta_1$ for convenience). Draw a value $\theta^{1}_1$ from the full conditional 
$p(\theta1|\theta^{0}_2, \theta^{0}_3, y).$

3. Draw a value $\theta^{1}_2$ (again order does not matter) from the full
conditional $p(\theta_2|\theta^{1}_1, \theta^{0}_3, y)$ . Note that we must use the
updated value of $\theta^{1}_1$.

4. Draw a value $\theta^{1}_3$ from the full conditional $p(\theta3|\theta^{1}_1, \theta^{1}_2, y)$ using both updated values. (Steps 2-4 are analogous to multiplying Q{0} and P to get $\Pi^{1}$ and then drawing $\theta^{1}$ from $\Pi{1}$.)

5. Draw $\theta^{2}$ using $\theta^{1}$ and continually using the most updated
values.

6. Repeat until we get M draws, with each draw being a vector $\theta^(t)$.

7. Optional burn-in and/or thinning


### Example
Suppose we have data of the number of failures (yi) for each of 10 pumps in a nuclear plant. We also have the times (ti) at which each pump was observed. 


```r
y = c(5, 1, 5, 14, 3, 19, 1, 1, 4, 22)
t = c(94, 16, 63, 126, 5, 31, 1, 1, 2, 10)
rbind(y, t)
```

```
##   [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10]
## y    5    1    5   14    3   19    1    1    4    22
## t   94   16   63  126    5   31    1    1    2    10
```

We want to model the number of failures with a Poisson likelihood, where the expected number of failure $λi$ differs for each pump. Since the time which we observed each pump is different, we need to scale each $λi$ by its observed time ti

Our likelihood is $\Pi^{10}_{i=1} Poisson(λiti)$.

Let’s put a $Gamma(\alpha, \beta)$ prior on $λi$ with $\alpha = 1.8$, so the $λi$s are drawn from the same distribution. Also, let’s put a $Gamma(\gamma, \delta)$ prior on β, with $\gamma = 0.01$ and $\delta = 1$

So our model has 11 parameters that are unknown ($10 λis$ and $\beta$) 

* First use  $Gamma(\gamma, \delta)$ to draw $\beta$ 
* Then use   $Gamma(\alpha, \beta)$ to draw all $\lambda$ since each $\lambda$ only depend on $\beta$, I can draw all the $\lambda$ simutaneously 
* repeat and get the convergence value 


```r
beta.cur = 1

gibbs = function(n.sims, beta.start, alpha, gamma, delta, y, t, burnin = 0, thin = 1) {
 
 # define the starting parameter  
 beta.draws = c()
 lambda.draws = matrix(NA, nrow = n.sims, ncol = length(y))
 
 beta.cur = beta.start
 # define the update function
 # generate all lambda together
 lambda.update = function(alpha, beta, y, t) {
 rgamma(length(y), y + alpha, t + beta) 
 }
 
 # generate the beta depend on previous lambda
 beta.update = function(alpha, gamma, delta, lambda,
 y) {
 rgamma(1, length(y) * alpha + gamma, delta + sum(lambda))
 }
 
 
 for (i in 1:n.sims) {
 lambda.cur = lambda.update(alpha = alpha, beta = beta.cur,
 y = y, t = t)
 beta.cur = beta.update(alpha = alpha, gamma = gamma,
 delta = delta, lambda = lambda.cur, y = y)
 if (i > burnin & (i - burnin)%%thin == 0) {
 lambda.draws[(i - burnin)/thin, ] = lambda.cur
 beta.draws[(i - burnin)/thin] = beta.cur
 }
 }
 return(list(lambda.draws = lambda.draws, beta.draws = beta.draws))
 }
```



```r
posterior <- gibbs(n.sims = 10000, beta.start = 1, alpha = 1.8, gamma = 0.01, delta = 1, y = y, t = t)

colMeans(posterior$lambda.draws)
```

```
##  [1] 0.07093451 0.15310175 0.10377757 0.12331931 0.65644897 0.62443440
##  [7] 0.85100821 0.84856751 1.34547975 1.91840687
```

```r
mean(posterior$beta.draws)
```

```
## [1] 2.410094
```

```r
apply(posterior$lambda.draws, 2, sd)
```

```
##  [1] 0.02697097 0.09238869 0.03940324 0.03082050 0.30954527 0.13707702
##  [7] 0.54380561 0.53630513 0.60921568 0.40652026
```

```r
sd(posterior$beta.draws)
```

```
## [1] 0.6864925
```


