# Mixed-Integer Programming Formulation of MAPH Problems

- [Mixed-Integer Programming Formulation of MAPH Problems](#mixed-integer-programming-formulation-of-maph-problems)
  - [Introduction](#introduction)
  - [Discrete-Time Discrete-Space](#discrete-time-discrete-space)
    - [Objective](#objective)
    - [Constraints](#constraints)
      - [Connectivity Constraints](#connectivity-constraints)
      - [Conflict Constraints](#conflict-constraints)
  - [Continuous-Time Discrete-Space](#continuous-time-discrete-space)
    - [Objective](#objective-1)
    - [Constraints](#constraints-1)
      - [Connectivity Constraints](#connectivity-constraints-1)
      - [Conflict Constraints](#conflict-constraints-1)

## Introduction

We investigate the general formulation of MAPH problems as binary mixed-integer programming.

In plain language, a Multi-Agent Path Finding (MAPH) problem is that a group of agents wants to travel through a network. Each agent has its own starting point and destination. We need to find a routing schedule that determines the path of all agents where they arrive at their destinations without colliding each other.

Let's first introduce some important notations.

- The network is modeled as a directed graph $\mathcal{G} = (\mathcal{V}, \mathcal{E})$
- The agents form a set $\mathcal{A}$ where each agent $i$ has
  - a starting vertex $v^{(s)}_i \in \mathcal{V}$ with a departure time $t_i^{(s)}$, and
  - a destination (goal) vertex $v^{(g)}_i \in \mathcal{V}$

The agents are only allowed to travel from vertex $u$ to another vertex $v$ if such edge $e = (u,v) \in \mathcal{E}$ exists in the graph.

Commonly, the two most common objectives for MAPH problems are as follows. Let $t^{(g)}_i$ be the time where agent $i$ arrives at its destination vertex.

1. `Makespan`: The system time required for the whole fleet of agents to achieve their destinations.

   ```math
   \max_{i \in \mathcal{A}} t^{(g)}_i - \min_{j \in \mathcal{A}} t^{(s)}_j
   ```

2. `Sum of costs`: Sum of all agents' travel time.

   ```math
   \sum_{i \in \mathcal{A}} t^{(g)}_i - t^{(s)}_i
   ```

In this article, we focus the philosophy of `sum of costs`

## Discrete-Time Discrete-Space

Traditional setting as what other paper proposed.

### Objective

```math
\sum_{t=1}^T \sum_{i \in \mathcal{A}} \left( \sum_{e \in \mathcal{E}} x_{i,e,t} c_{e}(i;t) + \sum_{v \in \mathcal{V}} y_{i,v,t} c_v(i;t) \right)
```
- $x_{i,e,t} \in \lbrace 0, 1 \rbrace$: $x_{i,e,t} = 1$ when agent $i$ travels through edge $e$ at time slot $t$
- $c_e(i;t) \in \mathbb{R}^+$: cost for agent $i$ to travel through edge $e$ at time $t$

- $y_{i,v,t} \in \lbrace 0, 1 \rbrace$: $y_{i,e,t} = 1$ when agent $i$ stays in vertex $v$ at time $t$
- $c_v(i;t) \in \mathbb{R}^+$: cost for agent $i$ to stay in vertex $v$ at time $t$

Such the costs can be the required time to travel through an edge or stay at a vertex, but it can expand to more cost functions.

### Constraints

#### Connectivity Constraints

To guarantee the liveliness of a program, for each agent $i$, we restrain the departure from the source vertex $v_i^{(s)}$ and the arrival of the destination (goal) vertex $v_i^{(g)}$.

```math
\sum_{t=1}^T \left( \sum_{e = (v_i^{(s)}, u) \in \mathcal{E}} x_{i, e,t} - \sum_{e' = (u, v_i^{(s)}) \in \mathcal{E} } x_{i,e',t} \right) = 1 \;, \forall i \in \mathcal{A}
```
```math
\sum_{t=1}^T \left( \sum_{e = (u, v_i^{(g)}) \in \mathcal{E}} x_{i, e,t} - \sum_{e' = (v_i^{(g)}, u) \in \mathcal{E}} x_{i,e',t} \right) = 1 \;, \forall i \in \mathcal{A}
```

This also applies to vertex selection variable
```math
y_{i, v_i^{(s)}, t_i^{(s)}} = y_{i, v_i^{(g)}, t_i^{(g)}} = 1 \;, \forall i \in \mathcal{A}
```

The Kirchhoff's flow equation needs to be applied to guarantee the path between an agent's source and destination is connected.

```math
\sum_{e = (u, v) \in \mathcal{E}} x_{i,e} = \sum_{e' = (v, w) \in \mathcal{E}} x_{i,e'} \; ,\forall i \in \mathcal{A}, \forall v \in \mathcal{V} - \lbrace v_i^{(s)}, v_i^{(g)} \rbrace
```

Also, the vertex traversal can be determined by edges, i.e.
```math
y_{i,v} = 
\begin{cases}
1 & \text{if } \sum_{e = (u,v) \in \mathcal{E}} x_{i,e} + \sum_{e' = (v,w) \in \mathcal{E}} x_{i,e'} > 0 \\
0 & \text{otherwise}
\end{cases}
```

This is equivalent to the below constraints, for all agent $i$,
```math
\begin{split}
\sum_{e = (u,v) \in \mathcal{E}} x_{i,e} + \sum_{e' = (v,w) \in \mathcal{E}} x_{i,e'} & \geq 1 - M (1 - y_{i,v}) \\
\sum_{e = (u,v) \in \mathcal{E}} x_{i,e} + \sum_{e' = (v,w) \in \mathcal{E}} x_{i,e'} & \leq M y_{i,v}
\end{split}
```
where $M$ is a large enough number.

#### Conflict Constraints

Let $\tau_{i,v}^{(V)} \in \mathbb{Z}^+$ be the number of time slots for agent $i$ to stay at vertex $v$; and $\tau_{i,e}^{(E)} \in \mathbb{Z}^+$ be the number of time slots for agent $i$ to travel through edge $e$.

Now, we would like to formulate the timing of agents' whereabouts to detect conflicts. Let $\bar{t}_{i,v} \in \mathbb{Z}^+$ be the time when agent $i$ arrives at vertex $v$.

Since we allow an agent $i$ to stay at vertex $v$ with time $\tau_{i,v}^{(V)} \in \mathbb{Z}^+$ which is separated from the edge traveling time $\tau_{i,e}^{(E)} \in \mathbb{Z}^+$, the timing sequence for an agent to travel through a vertex and an edge is as follows
- The time agent $i$ just arrives at vertex $v$ is $\bar{t}_{i,v}$
- The time agent $i$ finishes waiting at vertex $v$ and start to move along an edge $e = (v, v')$ is $\bar{t}_{i,v} + \tau_{i,v}^{(V)}$
- The time agent $i$ finishes traveling through edge $e = (v, v')$ is $\bar{t}_{i,v} + \tau_{i,v}^{(V)} + \tau_{i,e}^{(E)} = \bar{t}_{i,v'}$

The above sequence is true when the agent actually travels through the path, thus the true constraints in our problem is as follows:

```math
\bar{t}_{i,v} = \bar{t}_{i,u} + y_{i,v}\tau_{i,u}^{(V)} + x_{i,e} \tau_{i, e}^{(E)} \; \forall e = (u,v) \in \mathcal{E}
```

With the base case
```math
\bar{t}_{i,v_i^{(s)}} \geq t_i^{(s)} \; \forall i \in \mathcal{A}
```

- **Vertex Conflict**: at any given time, a vertex can only have no more than 1 agent. i.e. for any pair of agents $i,j \in \mathcal{A}$ and any vertex $v$. The article [Linear Optimization - Putting gaps between scheduled items?](https://math.stackexchange.com/questions/2491500/linear-optimization-putting-gaps-between-scheduled-items?noredirect=1&lq=1) provides detail and reasoning about the formulation. In short, suppose we have the starting and ending time of two events $i,j$ being $S_i, E_i, S_j, E_j$. The formulation of the scheduling the 2 events with at least a time gap $G$ in the middle is as follows

  ```math
  \begin{split}
  S_i & \geq E_j + G - M \delta_{i,j} \\
  S_j & \geq E_i + G - M (1 - \delta_{i,j})
  \end{split}
  ```
  where $M$ is a large enough number as the planning length.

  This formulation is equivalent to
  ```math
  \begin{cases}
  S_i \geq E_j + G & \text{ if } \delta_{i,j} = 1 \\
  S_j \geq E_i + G & \text{ if } \delta_{i,j} = 0
  \end{cases}
  ```
  
  With such formulation in mind, we have the equivalent formulation in our MAPH problem.

  ```math
  \begin{split}
  \bar{t}_{i,v} - y_{i,v} \xi_{i,v}^- & \geq \bar{t}_{j,v} + y_{j,v} \tau_{j,v}^{(V)} + y_{j,v} \xi_{j,v}^+ - M \delta_{i,j,v} \\
  \bar{t}_{j,v} - y_{j,v} \xi_{j,v}^- & \geq \bar{t}_{i,v} + y_{i,v} \tau_{i,v}^{(V)} + y_{i,v} \xi_{i,v}^+ - M (1 - \delta_{i,j,v})
  \end{split} 
  ```
  where 
  - $M$ is a large enough number as the planning length;
  - $\xi_{i,v}^- \in \mathbb{Z}^+$ is the prior safe time-interval before agent $i$'s arrival at vertex $v$;
  - $\xi_{i,v}^+ \in \mathbb{Z}^+$ is the posterior safe time-interval after agent $i$'s departure at vertex $v$;
  - $\delta_{i,j,v} \in \{0,1\}$ is the binary variable for the decision making.

  This means agent $i$ occupies vertex $v$ from time $S_i = \bar{t}_{i,v} - y_{i,v} \xi_{i,v}^-$ until time $E_i = \bar{t}_{i,v} + y_{i,v} \tau_{i,v}^{(V)} + y_{i,v} \xi_{i,v}^+$ without any enforced gap $G=0$. During this time period, no overlapping can happen between any two agents.


- **Edge Conflict**: Similar to the vertex's formulation, we prevent any time overlapping of any pairs of agents to travel through any edge. Now the starting and ending time for agent $i$ to travel on edge $e = (u,v)$ is

  ```math
  \begin{split}
  S_i & = \bar{t}_{i,u} + y_{i,u} \tau_{i,u}^{(V)} - x_{i,e} \xi_{i,e}^- \\
  E_i & = \bar{t}_{i,v} + x_{i,e} \xi_{i,e}^+
  \end{split}
  ```

  Therefore, the non-overlapping constraint becomes

  ```math
  \begin{split}
  \bar{t}_{i,u} + y_{i,u} \tau_{i,u}^{(V)} - x_{i,e} \xi_{i,e}^- & \geq \bar{t}_{j,v} + x_{j,e} \xi_{j,e}^+ - M \delta_{i,j,e} \\
  \bar{t}_{j,u} + y_{j,u} \tau_{j,u}^{(V)} - x_{j,e} \xi_{j,e}^- & \geq \bar{t}_{i,v} + x_{i,e} \xi_{i,e}^+ - M (1 - \delta_{i,j,e})
  \end{split}
  ```


## Continuous-Time Discrete-Space

One may observe that the timing criteria does not necessarily need to be integer. Following the similar philosophy, we can define the MAPH problem in continuous-time as below.

In this formulation, we demonstrate the formulation where the time can be continuous and the agents are allowed to have different speed. We can also separate the **cost** function from the travel time, we can have different cost like budget, etc.

### Objective

```math
\sum_{i \in \mathcal{A}} \left( \sum_{e \in \mathcal{E}} x_{i,e} c_{e}(i) + \sum_{v \in \mathcal{V}} y_{i,v} c_v(i) \right)
```
- $x_{i,e} \in \lbrace 0, 1 \rbrace$: $x_{i,e} = 1$ when agent $i$ travels through edge $e$
- $c_e(i) \in \mathbb{R}^+$: cost for agent $i$ to travel through edge $e$

- $y_{i,v} \in \lbrace 0, 1 \rbrace$: $y_{i,e} = 1$ when agent $i$ stays in vertex $v$
- $c_v(i) \in \mathbb{R}^+$: cost for agent $i$ to stay in vertex $v$

As the costs $c_e(i)$ and $c_v(i)$ are some constants, the only variables are $x_{i,e}$ and $y_{i,v}$.

In the conventional setting, the costs $c_e(i)$ can be the time of travel and $c_v(i)$ is the required time of stay in the station (agent may be forced to stop for a while.)

Consider the vehicle routing problem where agents are vehicles and the edges are roads. Let $\tau_{i,e}^{(E)} \in \mathbb{R}^+$ be the time required for agent $i$ to travel through edge $e$.

For each agent, the travel time is
```math
t_i^{(g)} - t_i^{(s)} = \sum_{e \in \mathcal{E}} x_{i,e} \tau_{i,e}^{(E)} = \sum_{e \in \mathcal{E}} x_{i,e} \frac{\ell(e)}{\nu_{i,e}^{(max)}} = \sum_{e \in \mathcal{E}} x_{i,e} \frac{\ell(e)}{\min(\nu_i^{(max)}, \nu_e^{(max)})}
```

where $\ell(e)$ is the length of edge $e$; $\nu_{i,e}^{(max)}$ is the maximum speed of agent $i$ on edge $e$. This $\nu_{i,e}^{(max)}$ can be further divided into 2 components, $\nu_{i}^{(max)} being $the maximum speed of agent $i$ and the $\nu_e^{(max)}$ maximum speed allowed on edge $e$.


### Constraints

#### Connectivity Constraints

To guarantee the liveliness of a program, for each agent $i$, we restrain the departure from the source vertex $v_i^{(s)}$ and the arrival of the destination (goal) vertex $v_i^{(g)}$.

```math
\sum_{e = (v_i^{(s)}, u) \in \mathcal{E}} x_{i,e} - \sum_{e' = (u, v_i^{(s)}) \in \mathcal{E}} x_{i,e'} = 1 \;, \forall i \in \mathcal{A}
```
```math
\sum_{e = (u, v_i^{(g)}) \in \mathcal{E}} x_{i,e} - \sum_{e' = (v_i^{(g)}, u)} x_{i,e'} = 1 \;, \forall i \in \mathcal{A}
```

This also applies to vertex selection variable
```math
y_{i, v_i^{(s)}} = y_{i, v_i^{(g)}} = 1 \;, \forall i \in \mathcal{A}
```

The Kirchhoff's flow equation needs to be applied to guarantee the path between an agent's source and destination is connected.

```math
\sum_{e = (u, v) \in \mathcal{E}} x_{i,e} = \sum_{e' = (v, w) \in \mathcal{E}} x_{i,e'} \; ,\forall i \in \mathcal{A}, \forall v \in \mathcal{V} - \lbrace v_i^{(s)}, v_i^{(g)} \rbrace
```

Also, the vertex traversal can be determined by edges, i.e.
```math
y_{i,v} = 1 \iff \sum_{e = (u,v) \in \mathcal{E}} x_{i,e} + \sum_{e' = (v,w) \in \mathcal{E}} x_{i,e'} > 0
```

This is equivalent to the below constraints, for all agent $i$,
```math
\begin{split}
\sum_{e = (u,v) \in \mathcal{E}} x_{i,e} + \sum_{e' = (v,w) \in \mathcal{E}} x_{i,e'} & \geq 1 - M (1 - y_{i,v}) \\
\sum_{e = (u,v) \in \mathcal{E}} x_{i,e} + \sum_{e' = (v,w) \in \mathcal{E}} x_{i,e'} & \leq M y_{i,v}
\end{split}
```
where $M$ is a large enough number.

The reason behind this formulation can be found in this article [how-to-write-if-else-statement-in-linear-programming](https://math.stackexchange.com/questions/2500415/how-to-write-if-else-statement-in-linear-programming)

#### Conflict Constraints

Let $\bar{t}_{i,v} \in \mathbb{R}^+$ be the time when agent $i$ arrives at vertex $v$.

Since we allow an agent $i$ to stay at vertex $v$ with time $\tau_{i,v}^{(V)} \in \mathbb{R}^+$ which is separated from the edge traveling time $\tau_{i,e}^{(E)} \in \mathbb{R}^+$, the timing sequence for an agent to travel through a vertex and an edge is as follows
- The time agent $i$ just arrives at vertex $v$ is $\bar{t}_{i,v}$
- The time agent $i$ finishes waiting at vertex $v$ and start to move along an edge $e = (v, v')$ is $\bar{t}_{i,v} + \tau_{i,v}^{(V)}$
- The time agent $i$ finishes traveling through edge $e = (v, v')$ is $\bar{t}_{i,v} + \tau_{i,v}^{(V)} + \tau_{i,e}^{(E)} = \bar{t}_{i,v'}$

The above sequence is true when the agent actually travels through the path, thus the true constraints in our problem is as follows:

```math
\bar{t}_{i,v} = \bar{t}_{i,u} + y_{i,v}\tau_{i,u}^{(V)} + x_{i,e} \tau_{i, e}^{(E)} \; \forall e = (u,v) \in \mathcal{E}
```

With the base case
```math
\bar{t}_{i,v_i^{(s)}} \geq t_i^{(s)} \; \forall i \in \mathcal{A}
```

- **Vertex Conflict**: at any given time, a vertex can only have no more than 1 agent. i.e. for any pair of agents $i,j \in \mathcal{A}$ and any vertex $v$. The article [Linear Optimization - Putting gaps between scheduled items?](https://math.stackexchange.com/questions/2491500/linear-optimization-putting-gaps-between-scheduled-items?noredirect=1&lq=1) provides detail and reasoning about the formulation. In short, suppose we have the starting and ending time of two events $i,j$ being $S_i, E_i, S_j, E_j$. The formulation of the scheduling the 2 events with at least a time gap $G$ in the middle is as follows

  ```math
  \begin{split}
  S_i & \geq E_j + G - M \delta_{i,j} \\
  S_j & \geq E_i + G - M (1 - \delta_{i,j})
  \end{split}
  ```
  where $M$ is a large enough number as the planning length.

  This formulation is equivalent to
  ```math
  \begin{cases}
  S_i \geq E_j + G & \text{ if } \delta_{i,j} = 1 \\
  S_j \geq E_i + G & \text{ if } \delta_{i,j} = 0
  \end{cases}
  ```
  
  With such formulation in mind, we have the equivalent formulation in our MAPH problem.

  ```math
  \begin{split}
  \bar{t}_{i,v} - y_{i,v} \xi_{i,v}^- & \geq \bar{t}_{j,v} + y_{j,v} \tau_{j,v}^{(V)} + y_{j,v} \xi_{j,v}^+ - M \delta_{i,j,v} \\
  \bar{t}_{j,v} - y_{j,v} \xi_{j,v}^- & \geq \bar{t}_{i,v} + y_{i,v} \tau_{i,v}^{(V)} + y_{i,v} \xi_{i,v}^+ - M (1 - \delta_{i,j,v})
  \end{split} 
  ```
  where 
  - $M$ is a large enough number as the planning length;
  - $\xi_{i,v}^-$ is the prior safe time-interval before agent $i$'s arrival at vertex $v$;
  - $\xi_{i,v}^+$ is the posterior safe time-interval after agent $i$'s departure at vertex $v$;
  - $\delta_{i,j,v} \in \{0,1\}$ is the binary variable for the decision making.

  This means agent $i$ occupies vertex $v$ from time $S_i = \bar{t}_{i,v} - y_{i,v} \xi_{i,v}^-$ until time $E_i = \bar{t}_{i,v} + y_{i,v} \tau_{i,v}^{(V)} + y_{i,v} \xi_{i,v}^+$ without any enforced gap $G=0$. During this time period, no overlapping can happen between any two agents.


- **Edge Conflict**: Similar to the vertex's formulation, we prevent any time overlapping of any pairs of agents to travel through any edge. Now the starting and ending time for agent $i$ to travel on edge $e = (u,v)$ is

  ```math
  \begin{split}
  S_i & = \bar{t}_{i,u} + y_{i,u} \tau_{i,u}^{(V)} - x_{i,e} \xi_{i,e}^- \\
  E_i & = \bar{t}_{i,v} + x_{i,e} \xi_{i,e}^+
  \end{split}
  ```

  Therefore, the non-overlapping constraint becomes

  ```math
  \begin{split}
  \bar{t}_{i,u} + y_{i,u} \tau_{i,u}^{(V)} - x_{i,e} \xi_{i,e}^- & \geq \bar{t}_{j,v} + x_{j,e} \xi_{j,e}^+ - M \delta_{i,j,e} \\
  \bar{t}_{j,u} + y_{j,u} \tau_{j,u}^{(V)} - x_{j,e} \xi_{j,e}^- & \geq \bar{t}_{i,v} + x_{i,e} \xi_{i,e}^+ - M (1 - \delta_{i,j,e})
  \end{split}
  ```
