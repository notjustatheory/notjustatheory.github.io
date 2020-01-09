---
title: "Factor Graphs For Robotics (Part1: Theory)"
excerpt: "Lets use factor graphs for our robotic projects!."
header:
  teaser: /assets/contents/theories/1_9_2020/1.png 
---
Factor Graphs For Robotics
-------------

In the context of robotics and SLAM systems, factor graphs are powerful
tools for optimal state estimation. In this
post, we provide a brief overview of the theories and principles
behind this method. Here, we explain the concepts in the context of
state estimation and then present the generalization of this tool to other
optimization scenarios.

In robotics, sensor fusion can be formulated as a Bayesian inference
problem as follows: $$\label{eq:bayes_fusion}
\begin{aligned} X^{M A P} &=\underset{X}{\operatorname{argmax}} p(X | Z) \\ &=\underset{X}{\operatorname{argmax}} \frac{p(Z | X) p(X)}{p(Z)} \end{aligned}$$
where $Z$ and $X$ respectively represent the observations by sensors and
the system's states. The Maximum A Posteriori (MAP) estimate of the
states, $X^{M A P}$, which is the fusion result is found trough
maximizing the posterior distribution $p(X | Z)$ using the ML[^1]
algorithm. In this formulation, $p(Z | X)$ is the likelihood function
which represents the observation model of sensors while $P(X)$ is the
prior knowledge of the system's state. As we will see later, in a
dynamical system, this prior knowledge is modeled using the state
evolusion function.

If we assume $X$ to be a Markovian stochastic process and $p(Z | X)$ and
$p(X)$ to be normal distributions, the ML algorithm leads to a Least
Squares (LS) problem. Let us consider the following nonlinear system:

$$X_{k+1}=f(X_k,u_k)+w(k),\\
z_k=h(X_k)+v(k)$$ where $f(X)$ and $h(X)$ are system and measurement
models respectively. Furthermore, $v(k)$ and $w(k)$ are additive
Gaussian noise with covariance $R$ and $Q$, respectively representing
the measurement noise and model uncertainty. The vector $u_k$ is an
optional input to the system and $X$ is the system's states. The
likelihood function may be written as follows:

$$\label{eq:meas_model}
p(z | x)=\mathcal{N}(z ; h(x), R)=\frac{1}{\sqrt{|2 \pi R|}} \exp \left\{-\frac{1}{2}\|h(x)-z\|_{R}^{2}\right\}$$
The intuition is that when the state is $X$, we expect the observation
to be $h(x)$. Thus, $z$ measurements that are very different from $h(x)$
are less likely to happen. Furthermore, the less noisy the sensor is,
the more confident we can be about this expectation. This intuition is
modeled in the above equation using a Gaussian distribution function
with a center at $h(x)$ and a variance $R$.

On the other hand, the prior $p(X)$ captures our exception of the new
state given no information other than the previous estimate. This
knowledge comes from our understanding of the dynamical system, which in
this case, is presented by the function $f(X)$ in Equation
[\[eq:system\_model\]](#eq:system_model){reference-type="ref"
reference="eq:system_model"}. Therefore, similar to the likelihood
function, the prior distribution may be modeled using the following
Gaussian density: $$\label{eq:system_model}
p\left(x_{k+1} | x_{k}, u_{k}\right)=\frac{1}{\sqrt{|2 \pi Q|}} \exp \left\{-\frac{1}{2}\left\|f\left(x_{k}, u_{k}\right)-x_{k+1}\right\|_{Q}^{2}\right\}$$

Now, Given these densities, the ML algorithm leads to the following
nonlinear LS problem:
$$\hat{X_k}=\underset{X}{min}(\sum_{i=0}^{k}(\|f(x_{i}, u_{i})-x_{i+1}\|_{Q}^{2}+\|h(x_i)-z_i\|_{R}^{2}))$$

The Bayesian probabilistic formulation mentioned so far can be presented
graphically using Bayesian networks. A Bayesian network or BayesNet is a
directed graphical model with random variables as its nodes and edges
representing the dependencies (Causality) between these nodes. In other
words, a BayesNet can define a joint probability distribution over all
random variables as the product of conditional densities. Figure
[\[fig:fusion\_bayesnet\]](#fig:fusion_bayesnet){reference-type="ref"
reference="fig:fusion_bayesnet"} illustrates the Bayes Net for our
example nonlinear system. The corresponding joint probability
distribution may be written as follows:

![A BayesNet corresponding to our example state estimation
problem.](/assets/contents/theories/1_9_2020/1.png)

[\[fig:fusion\_bayesnet\]]{#fig:fusion_bayesnet
label="fig:fusion_bayesnet"}

$$\label{eq:toy_fusion_bayesnet}
\begin{aligned} p(X, Z) &=p\left(x_{1}\right) p\left(x_{2} | x_{1}\right) p\left(x_{3} | x_{2}\right) p\left(x_{4} | x_{3}\right) \\ & \times p\left(z_{1} | x_{1}\right)  p\left(z_{2} | x_{2}\right)  p\left(z_{3} | x_{3}\right)  p\left(z_{4} | x_{4}\right) \end{aligned}$$
In the above equation, the first line represents the system model and
the second line represents the observation models.

The joint distribution in Equation
[\[eq:bayes\_fusion\]](#eq:bayes_fusion){reference-type="ref"
reference="eq:bayes_fusion"} relates to the posterior distribution
$p(X|Z)$ through the conditional probability formula:
$$p(X|Z)=\frac{p(X,Z)}{p(Z)}$$ $p(Z)$ in this equation acts as
normalization factor thus, $p(X|Z)$ is proportional to the joint
distribution $p(Z,X)$. The MAP estimate would be equivalent to
maximizing the likelihood function $l(X;Z)$ which is proportional to
$p(Z,X)$,: $$l(X ; Z) \propto p(Z | X)$$
$$X^{M A P}=\underset{X}{\operatorname{argmax}} l(X ; Z)$$ Furthermore,
We can remove the constant scales in the conditional probabilities of
Equations [\[eq:meas\_model\]](#eq:meas_model){reference-type="ref"
reference="eq:meas_model"} and
[\[eq:system\_model\]](#eq:system_model){reference-type="ref"
reference="eq:system_model"} and write the likelihood function $l(X;Z)$
as follows:

$$\label{eq:fusion_likelihood}
\begin{aligned} l(X; Z) &=l\left(x_{1}\right) l\left(x_{2} | x_{1}\right) l\left(x_{3} | x_{2}\right) p\left(x_{4} | x_{3}\right) \\ & \times l\left(z_{1} | x_{1}\right)  l\left(z_{2} | x_{2}\right)  l\left(z_{3} | x_{3}\right)  l\left(z_{4} | x_{4}\right) \end{aligned}$$
where

$$l(X;z) = \exp \left\{-\frac{1}{2}\left\|\operatorname{g} \left(X\right)-z\right\|_{R}^{2}\right\},\\
l(X_{k+1};X_k) = \exp \left\{-\frac{1}{2}\left\|\operatorname{f} \left(X_k,u_k\right)-X_{k+1}\right\|_{Q}^{2}\right\}$$

Therefore, the likelihood function to be maximized is a factorized
product of smaller likelihood functions. While the joint probability
distribution in Equation
[\[eq:toy\_fusion\_bayesnet\]](#eq:toy_fusion_bayesnet){reference-type="ref"
reference="eq:toy_fusion_bayesnet"} is best described using a BayesNet,
the factorized likelihood function of Equation
[\[eq:fusion\_likelihood\]](#eq:fusion_likelihood){reference-type="ref"
reference="eq:fusion_likelihood"} may be best represented by a factor
graph.

A factor graph $G$ defines the factorization of a function $f(X)$ as a
product of factors $f_i$. It is a bipartite and undirected graphical
model with two type of nodes, factor nodes $f-i \in F$ and variable
nodes $x_i \in X$. An edge $e_{i j} \in \mathcal{E}$ is present in the
graph if and only if $f_i$ is a function of $x_j$ [@Kaess2012]. Figure
[\[fig:fusion\_factorgraph\]](#fig:fusion_factorgraph){reference-type="ref"
reference="fig:fusion_factorgraph"} illustrates the factor graph
corresponding to the likelihood function in Equation
[\[eq:fusion\_likelihood\]](#eq:fusion_likelihood){reference-type="ref"
reference="eq:fusion_likelihood"}. In this figure, the yellow factors
correspond to the measurement likelihoods while the black ones
correspond to the system model.

![The factor graph corresponding to our example state estimation
problem.](/assets/contents/theories/1_9_2020/4.png)

[\[fig:fusion\_factorgraph\]]{#fig:fusion_factorgraph
label="fig:fusion_factorgraph"}

Finally, through the ML algorithm, the MAP problem transforms into a
nonlinear LS problem:
$$\begin{aligned} X^{M A P} &=\underset{X}{\operatorname{argmax}} l(X;Z) \\ &=\underset{X}{\operatorname{argmax}} \: Ln(\prod_{i} l_{i}\left(X_{i};Z_i\right)) \\
&=\underset{X}{argmin}(\sum_{i=0}^{k}(\|f(x_{i}, u_{i})-x_{i+1}\|_{Q}^{2}+\|h(x_i)-z_i\|_{R}^{2}))\end{aligned}$$
In the case of navigation systems, $X$ represent the tracked poses which
connected to each other through motion model factors. On the other hand,
each state is also constrained by measurement model factors.

The solution to an LS problem corresponding to a factor graph may be
categorized into four groups. In the following subsections, we introduce
these four methods.

### Batch Optimizers

In batch optimization, all factors from the beginning of time up to the
current moment participate in the optimization algorithm. Since all the
information participate in the optimization problem, this procedure
yields the most accurate result. However, it is an off-line algorithm
due to its heavy computational workload.

### Fixed Lag Smothers and Filters

Batch optimizers grow in complexity with time so it would be intractable
to run them in real-time. Therefore, one solution is to remove old
information by marginalizing them and adding their accumulative effect
as a prior factor at the tail of the truncated factor graph. This
marginalization and truncation happen at each iteration of the
algorithm. The scale of the problem remains constant in this case and
depends on the window size.

In filtering, which is an extreme case of fixed lag smoothing, all the
prior data up to the current node shrink into a single prior factor.
This case has the lowest computational and storage overhead. Even though
this truncation of the factor graph has no degenerative effects for
linear systems, for nonlinear systems, it leads to the accumulation of
errors in the prior factor. This accumulation is due to the modification
of linearization points at each iteration, which can no longer propagate
back into the previous nodes.

### Incremental Smoothers

Incremental smoothing presented in [@Kaess2012] is a novel algorithm for
incrementally augmenting and solving the factor graph as a new data
comes in. Since state estimation problems in robotics generally lead to a Markov
chain, the generated factor graph is often sparse, and the required
matrix factorization during the execution of iterative nonlinear
optimization can be done more efficiently. The authors in [@Kaess2012]
exploit iterative matrix factorization methods to develop their
algorithm.

Due to the Markovian nature of the problem, the generated BayesNet
morphs a chordal graph. This chordal graph is then transformed into a
Bayes tree. New information may be added easily without having to rerun
all the computations. Even though the computational overhead of this
method is much lower, the memory requirements do not change.
Furthermore, one can not be sure about how long each iteration of this
algorithm would take. Thus, in general, this method can not provide
deterministic estimates, and based on the problem and observed
measurements, the computation overhead may vary significantly. For
example, when we add a loop constraint, the new iteration requires a
more significant portion of the tree to be processed, which requires a
much longer computational time.

### Concurrent Filtering And Smoothing

[@Kaess2012] presents a framework with a running filter alongside a
smoother. At each smoother's iteration, the filter can iterate multiple
times providing real-time estimates. Then, at the end of each smoother's
iteration, these two systems get synchronized.

The separation of these two systems is achieved through an intermediate
separator variable in the Bayes tree. Given this variable, the filter
would be statistically independent of the smoother. At the
synchronization step, the separator updates such that newly added states
to the filter move to the smoother's tree.

Factor Graphs for System Identification
---------------------------------------

In the previous section, we introduced factor graphs using a state
estimation example application. However, factor graphs may also be
utilized for solving identification problems [@Reinke]. In this case,
the parameters to be identified will be added to the robot's factor
graph as if they were new states to be estimated. Depending on the
application, these newly added nodes will be constrained by specific
factors. As an example, here we consider the kinematic calibration
problem.

The kinematic equations of a robot relate the task-space and work-space
quantities using nonlinear kinematic models as follows:
$$\mathbf{X}=FK(\mathbf{q},\mathbf{\pi}),\: q=IK(\mathbf{X},\mathbf {\pi})$$
Were $\mathbf{X}$ is the robot's task-space pose and $\mathbf{q}$ is the
corresponding joint angles/lengths. $\mathbf{\pi}$ in the above equation
is the vector of kinematic parameters which depends of the robot's
geometry.

Let us assume these parameters are not accurately available. However, we
assume that $X$ and $q$ are measured using appropriate sensors. We
command the robot to undergo a random trajectory within its work-space
and record the $\mathbf{X}$ and $\mathbf{q}$ measurements. First, let us
assume the kinematic parameters are stationary and do not change during
the robot's operation.

As illustrated in Figure
[\[fig:calibration\_factorgraph\_1\]](#fig:calibration_factorgraph_1){reference-type="ref"
reference="fig:calibration_factorgraph_1"} we can construct a factor
graph corresponding to this experiment. In this figure, the red and
green factors respectively correspond to $\mathbf{X}$ and $\mathbf{q}$
measurements. Furthermore, the orange nodes between the $X$ states
represent the system model $\mathbf{X_{k+1}}=f(\mathbf{x_k})+w(k)$. This
model may also be represented by a random walk motion model
$\mathbf{x_{k+1}}=\mathbf{x_{k}} + w(k)$. The black factor corresponds
to the kinematic equations of the robot and is a function of joint
variables, task-space pose, and parameter $\mathbf{\pi}$.

Optimizing the LS problem corresponding to this graph yields the
identified parameters $\mathbf{\pi}$, which we may use later for
estimating the robot's pose without having any task-space sensors.

![The factor graph corresponding to the identification problem with
stationary parameters.](/assets/contents/theories/1_9_2020/3.png)

[\[fig:calibration\_factorgraph\_1\]]{#fig:calibration_factorgraph_1
label="fig:calibration_factorgraph_1"}

This approach towards identification leads to an off-line batch
optimization problem. Another useful scenario is when the parameters are
not constant, and we need to estimate them in real-time. In this case,
the factor graph in Figure
[\[fig:calibration\_factorgraph\_1\]](#fig:calibration_factorgraph_1){reference-type="ref"
reference="fig:calibration_factorgraph_1"} should be modified to account
for the non-stationary parameters. Figure
[\[fig:calibration\_factorgraph\_2\]](#fig:calibration_factorgraph_2){reference-type="ref"
reference="fig:calibration_factorgraph_2"} illustrates this new graph.
The yellow factors in this figure impose random walk constraints on the
estimated parameters to prevent them from sudden fluctuations. This
constraint stems from the fact that in a physical system, we expect the
parameters to evolve gradually and without sudden changes.

![The factor graph corresponding to the identification problem with
non-stationary parameters.](/assets/contents/theories/1_9_2020/2.png)

[\[fig:calibration\_factorgraph\_2\]]{#fig:calibration_factorgraph_2
label="fig:calibration_factorgraph_2"}

Finally, this new graph can be solved in real-time using fixed-lag or
incremental smoothing approaches.

### GTSAM Library

As mentioned in the previous sections, factor graphs are powerful tools
for formulating state estimation and calibration problems. Due to their
flexibility, they have been a backbone for constructing map optimization
problems in SLAM systems. Therefore, there are many open-source, and
efficient solvers developed to implement this tool. One particular
library is the GTSAM [@gtsam-reference]. Developed at Georgia Tech
university, GTSAM provides a full-fledged API for implementing custom
factors, defining factor graphs, and solving them efficiently.

Furthermore, GTSAM also supports optimizing on manifolds, which is
essential when we deal with optimization problems subject to parameters
from special Euclidean groups. This library also provides a Matlab
interface for fast development. Finally, there are many factors already
implemented in this library ready to be exploited for robotic and SLAM
problems. In future posts, we will try GTSAM alongside the ROS operating
system to implement a cool fusion project!

[^1]: Maximum Likelihood
