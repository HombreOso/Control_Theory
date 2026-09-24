Here is the extracted table from the image, formatted for clarity:

---

Percentage of Survey Respondents on Control Technology Impact (Samad et al., 2020)

| **Control Technology**                   | **Current Impact (% High)** | **Future Impact (% High)** |
| ---------------------------------------- | --------------------------- | -------------------------- |
| PID control                              | 91%                         | 78%                        |
| System Identification                    | 65%                         | 72%                        |
| Estimation and filtering                 | 64%                         | 63%                        |
| **Model-predictive control**             | **62%**                     | **85%**                    |
| Process data analytics                   | 51%                         | 70%                        |
| Fault detection and identification       | 48%                         | 78%                        |
| Decentralized and/or coordinated control | 29%                         | 54%                        |
| Robust control                           | 26%                         | 42%                        |
| Intelligent control                      | 24%                         | 59%                        |
| Discrete-event systems                   | 24%                         | 39%                        |
| Nonlinear control                        | 21%                         | 42%                        |
| Adaptive control                         | 18%                         | 44%                        |
| Repetitive control                       | 12%                         | 17%                        |
| Hybrid dynamical systems                 | 11%                         | 33%                        |
| Other advanced control technology        | 11%                         | 25%                        |
| Game theory                              | 5%                          | 17%                        |
Here’s a structured breakdown of the content from the image, focusing on the **Model Predictive Control (MPC) with state observation and disturbance estimation**:

---

### **Key Concepts**

1. **Extended Model with Disturbances**: The system dynamics are augmented to include **constant unknown disturbances** (dkd_kdk​) in the measured outputs:
    
    {xk+1=Axk+Bukdk+1=dkyk=Cxk+dk\begin{cases} x_{k+1} = A x_k + B u_k \\ d_{k+1} = d_k \\ y_k = C x_k + d_k \end{cases} ⎩⎨⎧​xk+1​=Axk​+Buk​dk+1​=dk​yk​=Cxk​+dk​​
    
    - xkx_kxk​: State vector.
    - uku_kuk​: Control input.
    - dkd_kdk​: Unknown disturbance (assumed constant).
    - yky_kyk​: Measured output.
2. **State Observer Design**:
    
    - Use the extended model to design a **state observer** (e.g., Kalman filter).
    - The observer estimates both the **state** (x^(t)\hat{x}(t)x^(t)) and the **disturbance** (d^(t)\hat{d}(t)d^(t)) from the measured output y(t)y(t)y(t).

---

### **Offset-Free Tracking in Steady-State**

- **Intuition**:
    
    - The observer ensures: Cx^(t)+d^(t)→y(t)C \hat{x}(t) + \hat{d}(t) \to y(t)Cx^(t)+d^(t)→y(t) _(Estimation error → 0)_.
    - The MPC controller ensures: Cx^(t)+d^(t)→r(t)C \hat{x}(t) + \hat{d}(t) \to r(t)Cx^(t)+d^(t)→r(t) _(Predicted tracking error → 0)_.
    - Combined, this leads to: y(t)→r(t)y(t) \to r(t)y(t)→r(t) _(Actual tracking error → 0)_.
- **Steady-State Behavior**: The term d^(t)\hat{d}(t)d^(t) compensates for **model mismatch**, ensuring offset-free tracking.
    

---

### **Reference**

For further details, see the survey paper: [Pannocchia, Gabiccini, Artoni, 2015](https://doi.org/10.1016/j.jprocont.2015.07.002).

Here’s a structured summary of the key points from the image regarding **Model Predictive Control (MPC) and Interior-Point (IP) methods**:

---

### **Linear System in Interior-Point Iterations**

Each iteration of the interior-point method involves solving a linear system of the form:

[QE′A′0E000A00I00SZ][ΔxΔyΔzΔs]=[−rQ−rE−rA−rS]\begin{bmatrix} Q & E' & A' & 0 \\ E & 0 & 0 & 0 \\ A & 0 & 0 & I \\ 0 & 0 & S & Z \end{bmatrix} \begin{bmatrix} \Delta x \\ \Delta y \\ \Delta z \\ \Delta s \end{bmatrix} = \begin{bmatrix} -r_Q \\ -r_E \\ -r_A \\ -r_S \end{bmatrix} ​QEA0​E′000​A′00S​00IZ​​​ΔxΔyΔzΔs​​=​−rQ​−rE​−rA​−rS​​​

- **Variables**:
    - Q,E,AQ, E, AQ,E,A: Matrices defining the system.
    - S=diag(s)S = \text{diag}(s)S=diag(s), Z=diag(z)Z = \text{diag}(z)Z=diag(z): Diagonal matrices of slack and dual variables.
    - Δx,Δy,Δz,Δs\Delta x, \Delta y, \Delta z, \Delta sΔx,Δy,Δz,Δs: Increments for primal and dual variables.

---

### **Exploiting MPC Structure**

- The **structure** xk+1=Axk+Bukx_{k+1} = A x_k + B u_kxk+1​=Axk​+Buk​ in MPC can be **heavily exploited** to:
    - **Factorize** the linear systems efficiently.
    - **Solve** them more effectively.
- References: [Rao, Wright, Rawlings, 1998](https://doi.org/10.1137/S1052623497325952), [Wright, 2018](https://doi.org/10.1137/16M1082569).

---

### **Interior-Point (IP) Method Performance**

1. **Convergence**:
    
    - IP typically provides **good solutions within 10–15 iterations**.
    - Linear systems tend to become **ill-conditioned at convergence**.
2. **Efficiency**:
    
    - IP is **usually faster** for **sparse and large-scale Quadratic Programs (QPs)** (e.g., problems with >500 variables and constraints).

---

### **References**

- (Nocedal, Wright, 2006)
- (Gondzio, Terlaki, 1994)

Here’s a structured breakdown of the content from the image, focusing on **multiparametric Quadratic Programming (mpQP) and its application to linear Model Predictive Control (MPC)**:

---

### **Key Concept: Continuous & Piecewise Affine Solution**

- **Problem Formulation**: The solution z∗(x)z^*(x)z∗(x) of a **strictly convex multiparametric Quadratic Program (mpQP)** is **continuous and piecewise affine** in xxx. The optimization problem is defined as:
    
    z∗(x)=arg⁡min⁡z(12z′Qz+x′F′z)subject toGz≤W+Sxz^*(x) = \arg\min_z \left( \frac{1}{2} z' Q z + x' F' z \right) \quad \text{subject to} \quad G z \leq W + S x z∗(x)=argzmin​(21​z′Qz+x′F′z)subject toGz≤W+Sx
    
    - QQQ: Positive definite matrix.
    - F,G,W,SF, G, W, SF,G,W,S: Matrices defining the linear constraints.
    - zzz: Optimization variable.
    - xxx: Parameter vector.
- **Reference**: (Bemporad, Morari, Dua, Pistikopoulos, 2002).
    

---

### **Corollary: Linear MPC is Continuous & Piecewise Affine**

- The optimal control input u0∗(x)u_0^*(x)u0∗​(x) for **linear MPC** is also **continuous and piecewise affine** in the state xxx. It can be expressed as:
    
    u0∗(x)={F1x+g1if H1x≤K1⋮FMx+gMif HMx≤KMu_0^*(x) = \begin{cases} F_1 x + g_1 & \text{if } H_1 x \leq K_1 \\ \vdots \\ F_M x + g_M & \text{if } H_M x \leq K_M \end{cases} u0∗​(x)=⎩⎨⎧​F1​x+g1​⋮FM​x+gM​​if H1​x≤K1​if HM​x≤KM​​
    
    - Fi,giF_i, g_iFi​,gi​: Affine functions for each region.
    - Hi,KiH_i, K_iHi​,Ki​: Define the polyhedral regions in the state space.

---

### **Implementation & Tools**

- A **new mpQP solver** based on **Nonlinear Least Squares (NNLS)** is available (Bemporad, 2015).
- This solver is included in the **MPC Toolbox** since **R2014b** (Bemporad, Morari, Ricker, 1998–today).

---

Here’s a structured explanation of how to use **LTV-MPC (Linear Time-Varying Model Predictive Control) machinery** to handle **nonlinear MPC**, based on the image:

---

### **Key Idea**

To handle **nonlinear MPC**, solve a **sequence of LTV-MPC problems** at each time step ttt.

---

### **Algorithm Steps**

For h=0h = 0h=0 to hmax−1h_{\text{max}} - 1hmax​−1, perform the following steps iteratively:

1. **Simulate**:
    
    - Simulate the system from the current state x(t)x(t)x(t) using inputs UhU_hUh​.
    - Obtain the **state trajectory** XhX_hXh​.
2. **Linearize and Discretize**:
    
    - Linearize the system around the trajectory (Xh,Uh)(X_h, U_h)(Xh​,Uh​).
    - Discretize the linearized system in time.
3. **Solve QP**:
    
    - Compute Uh+1∗U_{h+1}^*Uh+1∗​ as the **QP (Quadratic Programming) solution** of the corresponding LTV-MPC problem.
4. **Line Search**:
    
    - Perform a **line search** to find the optimal step size αh∈(0,1]\alpha_h \in (0, 1]αh​∈(0,1].
5. **Update Inputs**:
    
    - Update the inputs using:
        
        $Uh+1=(1−αh)Uh+αhUh+1∗U_{h+1} = (1 - \alpha_h) U_h + \alpha_h U_{h+1}^* Uh+1​=(1−αh​)Uh​+αh​Uh+1∗$​
        
6. **Return Solution**:
    
    - After completing the iterations, return the solution $UhmaxU_{h_{\text{max}}}Uhmax$​​.

---

### **Special Case: Real-Time Iteration**

- If only **one iteration** is solved with α=1\alpha = 1α=1, it is referred to as **Real-Time Iteration (RTI)**.
- This approach is equivalent to **LTV-MPC**.

---

### **Reference**

- (Diehl, Bock, Schlöder, Findiesen, Nagy, Allgöwer, 2002).


Here’s a structured breakdown of the **Extended Kalman Filter (EKF)** for **output feedback** in the context of **Model Predictive Control (MPC)**, based on the image:

---

### **Extended Kalman Filter (EKF) for State Estimation**

The EKF is used for **state estimation** in nonlinear systems. It is based on the same nonlinear model as the system, with additional noise terms:

{x(k+1)=f(x(k),u(k),ξ(k))y(k)=g(x(k))+ζ(k)\begin{cases} x(k+1) = f(x(k), u(k), \xi(k)) \\ y(k) = g(x(k)) + \zeta(k) \end{cases} {x(k+1)=f(x(k),u(k),ξ(k))y(k)=g(x(k))+ζ(k)​

- x(k)x(k)x(k): State vector at time kkk.
- u(k)u(k)u(k): Control input at time kkk.
- ξ(k)\xi(k)ξ(k): Process noise.
- y(k)y(k)y(k): Measured output at time kkk.
- ζ(k)\zeta(k)ζ(k): Measurement noise.
- f(⋅)f(\cdot)f(⋅): Nonlinear state transition function.
- g(⋅)g(\cdot)g(⋅): Nonlinear measurement function.

---

### **Measurement Update**

The measurement update step corrects the state estimate using the latest measurement y(k)y(k)y(k):

1. **Compute Jacobian C(k)C(k)C(k)**:
    
    C(k)=∂g∂x(x^k∣k−1)C(k) = \frac{\partial g}{\partial x}(\hat{x}_{k|k-1}) C(k)=∂x∂g​(x^k∣k−1​)
    
    - x^k∣k−1\hat{x}_{k|k-1}x^k∣k−1​: Predicted state estimate at time kkk given measurements up to k−1k-1k−1.
2. **Compute Kalman Gain M(k)M(k)M(k)**:
    
    M(k)=P(k∣k−1)C(k)′[C(k)P(k∣k−1)C(k)′+R(k)]−1M(k) = P(k|k-1) C(k)' \left[ C(k) P(k|k-1) C(k)' + R(k) \right]^{-1} M(k)=P(k∣k−1)C(k)′[C(k)P(k∣k−1)C(k)′+R(k)]−1
    
    - P(k∣k−1)P(k|k-1)P(k∣k−1): Predicted state covariance matrix.
    - R(k)R(k)R(k): Measurement noise covariance matrix.
3. **Update State Estimate**:
    
    x^(k∣k)=x^(k∣k−1)+M(k)(y(k)−g(x^(k∣k−1)))\hat{x}(k|k) = \hat{x}(k|k-1) + M(k) \left( y(k) - g(\hat{x}(k|k-1)) \right) x^(k∣k)=x^(k∣k−1)+M(k)(y(k)−g(x^(k∣k−1)))
    
    - This step is **consumed by MPC** for feedback.
4. **Update Covariance Matrix**:
    
    P(k∣k)=(I−M(k)C(k))P(k∣k−1)P(k|k) = \left( I - M(k) C(k) \right) P(k|k-1) P(k∣k)=(I−M(k)C(k))P(k∣k−1)
    
    - III: Identity matrix.

---

### **Time Update**

The time update step predicts the state and covariance matrix for the next time step:

1. **Predict State**:
    
    x^(k+1∣k)=f(x^(k∣k),u(k))\hat{x}(k+1|k) = f(\hat{x}(k|k), u(k)) x^(k+1∣k)=f(x^(k∣k),u(k))
    
2. **Compute Jacobians A(k)A(k)A(k) and G(k)G(k)G(k)**:
    
    A(k)=∂f∂x(x^k∣k,u(k),E[ξ(k)])A(k) = \frac{\partial f}{\partial x}(\hat{x}_{k|k}, u(k), E[\xi(k)]) A(k)=∂x∂f​(x^k∣k​,u(k),E[ξ(k)])
    
    G(k)=∂f∂ξ(x^k∣k,u(k),E[ξ(k)])G(k) = \frac{\partial f}{\partial \xi}(\hat{x}_{k|k}, u(k), E[\xi(k)]) G(k)=∂ξ∂f​(x^k∣k​,u(k),E[ξ(k)])
    
    - E[ξ(k)]E[\xi(k)]E[ξ(k)]: Expected value of process noise.
3. **Predict Covariance Matrix**:
    
    $P(k+1∣k)=A(k)P(k∣k)A(k)′+G(k)Q(k)G(k)′P(k+1|k) = A(k) P(k|k) A(k)' + G(k) Q(k) G(k)' P(k+1∣k)=A(k)P(k∣k)A(k)′+G(k)Q(k)G(k)′$
    
    - Q(k)Q(k)Q(k): Process noise covariance matrix.

---