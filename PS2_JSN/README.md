# Computational Macro - Problem Set 

## How to run the code 
The notebook has been written using the course environment. You can set up a new environment by uncommenting the first chunk of code (given the Manifest.toml and Project.toml from the course are in the same folder as this notebook).
This is a Jupyter Notebook, so select the Julia Kernel set up via the course environment. Then all the code can be run - piece by piece or as a whole.

## Use of AI
I have used Claude to check my work (derivations, codes, answers to questions) and to give me input once I was stuck (mainly in coding, e.g. for making tables), and explaining to me what I didn't understand.

## Answers to Questions

### Question 2.1
We get the following results:
- for the discretized problem:  mu = 0.038462, nu = 0.247189, f = 0.95766213
- for the Gaussian closed form: mu = 0.038462, nu = 0.247253, f = 0.95766091
This induces the difference:   d(mu) = 0,   d(nu) = 0.000063, d(f) = 0.00000121

We see that the error in mu is an exact zero - both in the closed form and in the discretized version, the coefficient is exactly μ = r / Rp (see the functions exact_coefficients and closed_form_gauss), so the difference must necessarily be 0.
The difference in nu and f are non-zero, with the one in nu being larger by an order of magnitude (i.e. 6.4* 10⁻⁵ vs 1.2* 10⁻⁶). The error in consumption induced by this is 1.7 * 10⁻⁵. This is small relative to mean consumption equal to 1 (i.e. 1.7 * 10⁻³ percent).
The error is small relative to the error induced by the solvers (see Question 2.2).

### Question 2.2
#### a) Comparing methods
method             iterations  time (s)  ms/iter       e_W       e_G    EE max  EE mean log10
VFI on the grid           314      4.47    14.23   4.6e-02   1.8e-01   1.4e-02           -2.2
VFI off the grid          314     33.25   105.91   4.2e-06   1.8e-01   1.8e-07           -7.0
Howard                     15      2.84   189.03   1.5e-07   1.8e-01   7.9e-08           -8.0
Time iteration            136      3.94    28.97   4.6e-05   1.8e-01   3.3e-06           -5.9
EGM                       136      0.07     0.52   1.9e-05   1.8e-01   8.1e-07           -6.2

The iteration counts of my solution matches what is filled in already. As expected, VFI on and off the grid (both based on Bellman operator), and time iteration and EGM (using the Euler euqation) have the same number of iterations.
Howard has by far the least number of iterations but also the most expensive ones. EGM is by far the fastest method as it speeds up each iteration by fixing future assets and not having to find the root.
The error in consumption is constant for all methods if we look at the whole grid - this is due to how the grid is constructed and hence independent of the method. For the relevant window, it is best for Howard, and negligible for all methods except VFI on the grid.

The first set of plots shows consumption for the given income z=1.24 over the whole grid for all methods relative to the exact policy. We see that at the end of the grid, all methods diverge massively. Zooming into the relevant window, it shows that VFI on the grid has a non-negligible distance to the exact policy while the other methods perform very well (i.e. the error is non-visible).

The second set of plots shows the error and the Euler equation error. As seen in the table, for the whole grid we have the same maximum error for all methods due to how the grid is built on the edges of the grid. In the relevant window however, methods differ: VFI on the grid performs poorly, while Howard performs best. VFI off the grid, time iteration and EGM perform similarly well.

#### b) Comparing e_W and time per iteration across methods with N=200, 400, 800
method             N_a       e_W       e_G   EE(max)  EE(mean)   ms/iter 
____________________________________________________________________________
VFI on the grid    200   8.8e-02   1.8e-01   3.6e-02      -1.9       4.8
VFI off the grid   200   4.2e-06   1.8e-01   1.5e-07      -7.0      59.0
Howard             200   1.2e-07   1.8e-01   6.4e-08      -8.1     133.6
Time iteration     200   2.5e-04   1.8e-01   1.6e-05      -5.2      11.4
EGM                200   1.9e-05   1.8e-01   8.1e-07      -6.2       0.4
____________________________________________________________________________
VFI on the grid    400   4.6e-02   1.8e-01   1.4e-02      -2.2      14.7
VFI off the grid   400   4.2e-06   1.8e-01   1.8e-07      -7.0     114.6
Howard             400   1.5e-07   1.8e-01   7.9e-08      -8.0     178.7
Time iteration     400   4.6e-05   1.8e-01   3.3e-06      -5.9      24.4
EGM                400   1.9e-05   1.8e-01   8.1e-07      -6.2       0.4
____________________________________________________________________________
VFI on the grid    800   2.1e-02   1.8e-01   1.1e-02      -2.8      82.0
VFI off the grid   800   4.2e-06   1.8e-01   2.0e-07      -7.0     202.3
Howard             800   1.3e-07   1.8e-01   8.9e-08      -8.0     294.9
Time iteration     800   6.8e-06   1.8e-01   4.7e-07      -7.0      52.3
EGM                800   1.9e-05   1.8e-01   8.1e-07      -6.2       1.3
(Note that there are potentially slight deviations between the numbers for N_a=400 and the table from a), which results from me potentially not copying this from the same run. Small deviations between runs are normal.)

The error in consumption inside the window does not fall with the number of asset grid points by an order of magnitude for any method.
For VFI off the grid and EGM it stays exactly the same,  while for Howards it changes marginally and without a clear pattern. This is to be expected as N_a only tackles the error following from the asset grid - since these methods use interpolation, this becomes less of a problem.
The most obvious changes are for VFI on the grid and for time iteration: VFI on the grid halves the error for a doubling of the asset grid point. This makes sense as the error is likely due to the fact that the asset choice is forced to be on the grid and now the distance to the next grid point is exactly halfed. 
The error in time iteration falls by a factor of 5 to 6. It has to interpolate between a's, so again, more points make it more precise. In contrast to EGM, it interpolates between a and not c, which is non-linear.

The time per iteration quadruples for VFI on the grid, while it roughly doubles for all others
The quadrupluation for VFI on the grid comes from the fact that the number of asset-income combinations doubles for doubling the number of asset points. Comparing double the number of states to double the number of asset choices quadruples the run time.
The other methods avoid this problem: Howard and VFI off the grid use the more efficient Golden section search where the number of steps is independent of $N_a$, time iteration uses bisection, and EGM uses inversion plus iterpolation instead of a search. Hence, their run time per iteration only scales linearly, not quadratically.


#### c) Comparing e_W and time per iteration across methods with tighter tolerance
method             tolerance  ms/iter   e_W
____________________________________________________________________________
VFI on the grid      1.0e-06    13.75   4.6e-02
VFI off the grid     1.0e-06   116.78   4.2e-06
Howard               1.0e-06   194.97   1.5e-07
Time iteration       1.0e-06    32.65   4.6e-05
EGM                  1.0e-06     0.62   1.9e-05
____________________________________________________________________________
VFI on the grid      1.0e-08    16.60   4.6e-02
VFI off the grid     1.0e-08   119.72   1.4e-07
Howard               1.0e-08   184.61   1.4e-07
Time iteration       1.0e-08    23.30   6.5e-05
EGM                  1.0e-08     0.45   2.3e-07

The time for each iteration does not change substantially for any method due to the tighter tolerance. This makes sense as we only specify after how many iterations to stop, not what to do in each iteration step.

The error in the relevant window is unchanged for VFI on the grid, which is plausible since the error comes from the grid (as we saw in b)), not from the tolerance. The same holds for time iteration. 
Howard does not change substantially as the error was very small in the first place due to its fast contraction rate.
For VFI off the grid and EGM, there is a clear reduction in the error. Together with the result from b), this shows that the error comes from the stopping rule, not from the grid. This make sense of course given that they both search continuously and interpolate between grid points. 


#### d) Euler equation error
States at which a grid bound binds are excluded from the calculation of the two Euler equation error measures because near the grid ends, the Euler equation becomes an inequality. So the equality does not have to hold true there anyways.

The max Euler error ranks the methods as follows (from best to worst): Howard - VFI off grid - EGM - Time iteration - VFI on grid.
The absolut error in c ranks the methods as follows (from best to worst): Howard - VFI off grid, EGM - Time iteration - VFI on grid
Hence, the two methods rank the error the same way. Looking at the second set of graphs, we see that the Euler equation error is often much smaller, especially closer to the edges. Hence, it becomes clear that the Euler equation error is more positive about the solution for consumption than it should be. This is because it can only measure whether today's relative to future's consumption is right, but it cannot measure whether the level of consumption is right. So if the error in today's and tomorrow's consumption is similar, they cancel out and we get a small Euler equation error. So in general, the Euler equation error is good to detect large errors, but a small Euler equation error does not necessarily mean that the error is truly small.
In addition, the Euler equation error has to exclude all points where the borrowing constraint is binding, as the Euler equation is an inequality there. 

Sidenote: Comparing mean and max Euler equation also gives the same ranking. If there was a difference this would be due to the first measure looking at the largest error while the second looks at the mean. So if a method was very precise for a long time but had one spike, its max could look worse than the one of a method with no spike but higher imprecision. The mean smooths this out.

### Question 3.1

#### a) Comparing utility function, marginal utility, and inverse marginal utility
The utility function for CARA is $u(c)=-e^{-\gamma c}/\gamma$ while for CRRA it is $u(c)=\dfrac{c^{1-\gamma}}{1-\gamma}$. The first derivative of the utility function for CARA is $u_c(c)=e^{-\gamma c}$ while for CRRA it is $u_c(c)=c^{-\gamma}$. The inverse of the first derivative of the utility function for CARA is $u_c^{-1}=-\ln(m)/\gamma$ while for CRRA it is $u_c^{-1}=m^{-1/\gamma}$ for $m=u_c(c)$.

We see that utility is negative for both (given $\gammma>1$). From the first derivative however, we see that they are both upward sloping. Both are concave - so they are standard utility functions with risk aversion. However, CARA has constant absolute ($-u_cc/u_c=\gamma/c$), CRRA constant relative ($-u_cc*c/u_c=\gamma$) risk aversion. This materialises in the functions such that CRRA is more curved for c<1 and less curve for c>1 relative to CARA. 
The CRRA utility has an asymptote at c=0 (i.e. dividing by 0). Negative values for c produce positive utility values. It is hence only economically sensible to look at $c>0$. This must be enforced on the solver in practice. For CARA, $c<0$ is defined and is also allowed in the program.
The inverses need positive values for marginal utility, which implies that they are defined for all real numbers (CARA) vs for positive c (CRRA). E.g. with EGM, that implies that we do not need to impose bounds for CRRA, they are imposed automatically.

#### b) Comparing income paths and discretisation
For CARA, we had a linear, mean-reverting income process around 0: $z=\bar{z}+\e$ where $e=\rho e_{-1}+\sigma_\epsilon \epsilon$. For CRRA, we have a linear-in-log mean-reverting income process around 0: $\ln(z')=\rho \ln(c)+\sigma_\epsilon \epsilon'$. Hence, both build on an AR(1) process with high persistence ($\rho=0.9).
For CARA, the discretisation is performed on the AR(1) process, i.e. the rouwenhorst algorithm returns values for e (CARA) or ln(z) (CRRA), and transition probabilities. For CARA, the levels of e are then plugged into the linear income process with no further transformation needed. For CRRA, the income grid points are all exponentiated to get from log to levels. So the level of the grid points differs, the transition probabilities don't.

#### c) Comparing the asset grids (bounds, spacing, interpretation)
Question 2 used 400 equidistant asset grid points on [−15, 25]. The problem itself was unbounded, i.e. no borrowing constraint and no stationary wealth distribution in the long term, so the bounds were artificial.
In Question 3, we have a borrowing limit of $a'\geq 0$, so the asset points must all be weakly positive. Also, the upper limit can be determined here: With CRRA utility, households always dissave for states with high income and a certain level of assets, so there exists a limit of assets that is never crossed. We can set the upper bound of the asset grid slighlty above that level of assets (i.e. stationary wealth distribution).
For low income states and low current assets, the borrowing constraint binds. Hence, we need more grid points close to the borrowing limit (i.e. close to 0) to be able to distinguish more closely whether the limit is still binding or not. A log spacing instead of equidistance can produce that grid with more points at lower values. It cannot hold a=0 however. The alternative is quadratic spacing.

#### d) Admissible consumption in states given borrowing constraint and grids
In Question 2, consumption can be both negative (due to CARA) and very large (no borrowing constraint). We define the asset grid as [-15, 25], and consumption follows from $c=x-a'=z+Ra-a'$. So in practice for state (z,a), admissible consumption is [z+Ra-25, z+Ra+15]. The bounds just come from the asset grid.
In Question 3, consumption is bounded from above by the borrowing constraint and it cannot get negative. In practice, consumption is also bounded by the upper limit of assets accumulated. Given the asset grid bounds (0, a_upper) (where a_upper is the limited described in c), admissible consumption in state (z,a) is [max{0,z+Ra-a_upper}, z+Ra]. The upper bound has an economic reason, the lower bound comes from the asset grid bound (and the Inada conditions).

### Question 3.2
#### a) Parameters
The lower bound of the asset grid is given by the borrowing constraint - it has to be zero. The upper bound must respect the stationary wealth distribution, which I get from the slide deck 6 of the lecture. The deck shows that almost all households live between 0 and 100, so I set the upper bound to 100. This will lead to around 0.1% of states to be at the upper grid point. However, setting the mass to an exact zero requires a much higher upper bound. This means however that a large part of my grid covers a very small part of households (i.e. from 50 to 250) and lower coverage towards the lower bound, where it would really matter. So I stick with 100.
To evaluate the Euler equation error, I choose a window that respects two distortions from linear interpolation at the asset grid points: at the lower bound, the borrowing constraint may bind, which turns the Euler equation into an inequality. To avoid this, I set the lower bound of the error to 10. At the upper bound, the asset bound may bind for a very small share of households (though not for the stationary distribution). Since most households have assets below 50, I choose 50 as the upper bound for the Euler equation error window of evaluation.

#### b) Model diagnostics
My solutions reproduce the iteration count. 
The consumption in the left graph shows a smooth, concave policy for all methods for a low income state. For a high income state with low assets (below 10), we see how VFI on the grid fails (kinks at grid points). The level of consumption is much higher for a high income state (as expected).

#### c) Ranking methods using Euler equation error
EGM clearly dominates all other methods, both for the maximum and the mean log Euler equation error. The maximum Euler equation error is lower by two orders of magnitude. From the table, it seems that we can trust the results up to 6 to 7 digits. However, from 2.2 d), we see that the true error is always larger than the Euler equation error by roughly one order of magnitude. So we can only trust the EGM results up to 4 digits.
The other methods perform similarly well except for VFI on the grid. They are much more inaccurate than EGM, we should only take roughly up to 3 digits as correct. For VFI on the grid, we should not trust the results, the error is very large.

### Question 3.3.
For all methods, optimal savings are zero for 2% of all asset grid points at the lowest income state, and roughly 1.5-2% of all asset grid points for the second lowest income state. For all other income states, there are no asset grid points where zero saving is optimal. With a'=0, the borrowing constraint binds - the household would like to consume even more, but cannot borrow due to the constraint, so a'=0 is the optimal choice. Given that the household would like to borrow even more, the marginal utility of consumption today is higher than the discounted expected marginal utility of consumption tomorrow. Hence, the Euler equation becomes an inequality: $u_c(c)>\beta R E(u_c(c'))$.
For all methods except VFI on the grid, the highest Euler error over the whole grid is at the highest income state and at the upper bound of the asset grid. Note that this boundary is artificial, it is not part of the economic problem. Due to the quadratic spacing, the asset grid is coarser towards the upper limit. Interpolation errors are stronger here and so we see the highest errors at the upper limit of the grid.
It is however not important as the households never visit this high asset state - amax is set according to the stationary distribution. So the largest error over the grid doesn't matter for our evaluation of how well our numerical approximation is. This is why we specify a window.