# Computational Macro - Problem Set 3

## How to run the code
The notebook has been written using the course environment. You can set up a new environment by uncommenting the first chunk of code (given the Manifest.toml and Project.toml from the course are in the same folder as this notebook).
This is a Jupyter Notebook, so select the Julia Kernel set up via the course environment. Then all the code can be run - piece by piece or as a whole.

## Use of AI
I have used Claude to check my work (derivations, codes, answers to questions) and to give me input once I was stuck (mainly in coding, e.g. for making tables), and explaining to me what I didn't understand.

## Answers to Questions

### Question 2.1: PRANC vs Reiter results for CARA
All checks are successfully completed (correct $\mu, \nu$, eigenvalues for PRANC, consumption change for shock and peak of capital after shock).

#### a) r, K, C, $\beta$
| Variable  | PRANC     | Reiter      |
|-----------|-----------|-------------|
| r (in %)  | 4.00000   | 4.00012     |
| K         | 2.5714    | 2.57139     |
| C         | 0.7429    | 0.74286     |
| β         | 0.97471   | 0.97471     |

#### d) How well does the solver with household heterogeneity approximate the dynamics of the model?
The solver very well approximates the steady state of the model: The steady state interest rate, capital and consumption are correct to 3 digits. So the starting point for dynamics is highly accurate.

Regarding the dynamics, the two solution methods agree that consumption increases by 0.552% on impact for a 1% TFP shock, and that capital peaks at 1.026% in period 11. The shape of the impulse response functions are align very well. The largest deviations in the change of C and K is about 5*10⁻⁶ percentage points, so very small relative to a 0.552% change on impact for C and 1.026% at peak for K.

### Question 2.2: Varying the grids
#### a) Across grid sizes: number of variables in the linear system, r*, run times
 N_a   vars     r* (%)   t_ss  t_jac   t_qz
  40    401    4.00012    0.4    0.3    0.7
  80    801    4.00012    0.4    1.3    3.7
 120   1201    4.00012    0.6    1.5   11.7
#### b) Which of your results depend on the number of asset grid points Na , and which do not?
We are looking at a CARA utility here. That implies that we have a linear policy, which makes the EGM interpolation exact. There is no room for improvement with more grid poitns.
However, the grid influences how well Young's lottery approximates the true distribution. The error cancels out when we look at aggregates, as the mean is preserved by Young, but the shape is not correct, and can be improved by a finer grid.

This is reflected in the results: The interest rate as well as capital and consumption in the steady state are unchanged. Also, the impulse response functions are the same for different grid points. This is due to the fact that we look at aggregate values.

The distribution of cash on hand in steady state however gets more accurate the more grid points. It is a step function as households are forced on the grid by the lottery. The more grid points, the smaller the steps and so the closer the computed CDF to the true one.

#### c) How do the run times of the Jacobians and of Klein’s method grow with the number of variables?
The time for the linearisation increases overproportionally in the number of grid points. This is due to 2 facts:
For one, the jacobian computes the derivative. To do so, it evaluates F (the residual) for a small deviation for each variable separately in both directions (+h and -h). It does so for today's and tomorrow's x. This amounts to 4n evaluations for n variables.

Secondly, to build the Jacobian in the first place, we need the residuals at the perturbed points. That requires one step of EGM and one of Young. This also grows in the number of variables. Together, this scales the run time quadratically.

The time for the QZ decomposition increases overproportionally as well. It uses the n x n matrices A and B and decomposes them. As discussed in the course, QZ is cubic in the size of A and B - hence the growth in run time by order 3.

#### e) Which term of (4) is responsible for the change in r∗ ?
 N_e       r* (%)
   3      4.00024
   5      4.00012
   9      4.00006

We see that r* gets closer to the analytical benchmark the more income states we use in our discretisation. This is due to the precaution term:
A coarse grid gets the mean, variance and autocorrelation rgith, but not the tails. So it is less likely to have very bad draws, so households save less out of precaution. This pushes up the interest rate (i.e. less capital availabe, so the price must be higher). For more grid points, the approximation of tail risks is better and precautionary saving rises to the analytical benchmark level, thus decreasing the steady state interest rate r*.

### Question 2.3: Income risk
#### a)
   σ_e          β        β R
   0.0    0.97563    1.01465
   0.1    0.97471    1.01370
   0.3    0.96739    1.00609

#### b) Why does β have to change if the income risk changes?
$\beta$ has to change as for a given $\beta$, the income risk changes as the income risk reduces the interest rate through higher precautionary savings. To still match the targeted interest rate of 4%, households must discount the future by more.

#### c) How much does income risk change the aggregate dynamics, and why?
Capital reacts by more for higher income risk. This is due higher income risk increasing precautionary savings - a one-time TFP increase by 1% increases wages, and so in absolute terms, income risk is higher (by the last term in (4)), which makes them save more.

This directly implies that C has to jump by less. The change is hardly visible though as C changes little in each period, while changes in K add up over time. C should also increase after a while as more capital is available.

Output is predetermined in the first period. From then on, it should be slightly higher due to more capital available, but again the per-period change is small.


### Question 3.2: CRRA - The steady state
β          = 0.97352
β R        = 1.01246
% at a'=0  : 1.55%
mass at a_1: 5.6e-02,
mass at a_N: 6.6e-08

### Question 3.3: CRRA - The dynamics
The impulse response functions are very close. We see that the C jumps a tiny little bit higher on impact for the representative agent and K peaks at a higher value. However, the curves overlap very closely. This hints at that for aggregates, heterogeneity in household does not produce large changes in impulse responses. However, this may not hold for the distribution.

### Question 3.4: CRRA - Income risk
#### a) Share at limit and $\beta$ at different income risk levels
   σ_e          β        β R       at limit (%)
   0.0    0.97471    1.01370               0.00
   0.1    0.97352    1.01246               1.55
   0.3    0.96056    0.99898               9.27

#### b) In which of the three economies do the aggregates differ most from the representative agent, and why?
The economy with the largest income risk - i.e. highest variance of income process - is the furthest away from the representative agent in terms of impulse response in aggregates. The higher the income risk, the more consumption jumps, the lower the peak of capital, the faster output declines.

We see that a large share of households is at the borrowing limit for higher income risk. This is due to the fact that households are in low income states more often and would like to borrow there in expectation of higher future income. Households at the limit have a high MPC. So the increase in income from the income shock makes them consume all of the additional resources they get. Since they are a larger share of the population, C jumps by more. As less is saved, K peaks at a lower value and output declines faster.

The precautionary channel from the CARA economy is missing as relative income risk is not influenced by the shock here, it stays the same as income is proportional to w.



#### c) The households of the economy with σ_e = 0 still differ from each other. In what, and why is this economy not the representative agent either?
With σ_e = 0, there is no agent at the borrowing limit and there are no precautionary savings - just like with the representative agent. However, there is still death - each period, some agents die and new agents with zero assets enter. I.e. wealth differs across cohorts, so younger agents consume less. This makes aggregate consumption lower in the heterogeneous agents model compared to the representative agent.
