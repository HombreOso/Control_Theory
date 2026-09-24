# Duality between regulation and estimation (Table A.1)

This is the **system-theoretic duality** (adjoint system) between the LQ regulator and the least-squares estimator or Kalman filter. It is not the Lagrange/convex duality from Appendix C. The table says that a regulator and an estimator are the same mathematical problem, with the roles of the matrices swapped and time running backwards.

## The two problems

- **Regulator:** x⁺ = Ax + Bu, y = Cx. You choose u to minimize Σ (x'Qx + u'Ru) plus a terminal cost.
- **Estimator:** x⁺ = Ax + Gw, y = Cx + v. You choose the estimate to fit measurements, where w has covariance Q and v has covariance R.

The book derives the dual system on p. 678: (A, B, C, D) → (A', C', B', D'). The estimator's input matrix is renamed G, so the regulator's C maps to G'.

## Variable by variable

|Regulator|Estimator|Physical meaning|
|---|---|---|
|**A**|**A'**|Same dynamics, transposed. The adjoint system has the same eigenvalues and hence the same stability.|
|**B**|**C'**|**Actuation ↔ sensing.** B is where your inputs push the states. C is where your sensors look at the states.|
|**C**|**G'**|**What is penalized ↔ what is disturbed.** C selects the states the regulator cares about (state cost C'QC). G selects the states that process noise enters.|
|**k**|**l = N − k**|**Time reversal.** The regulator's Riccati equation runs backwards from the final time N. The estimator's runs forwards from time 0. It is the same recursion in opposite directions.|
|**Π(k), Π(k−1), Π**|**P⁻(l), P⁻(l+1), P⁻**|**Cost-to-go ↔ prior error covariance.** Π gives the cost V(x) = ½x'Πx of driving state x to zero. P⁻ is the uncertainty of the state estimate before the new measurement. Both are large when the system is hard to steer or hard to observe. Π and P⁻ are steady-state in the last pair.|
|**Q**|**Q**|State/output weight ↔ **process-noise covariance**. A large Q means you care a lot about state error, or you distrust the model, so you react more strongly.|
|**R**|**R**|Input weight ↔ **measurement-noise covariance**. A large R means control is expensive, or the sensor is noisy, so you react more cautiously.|
|**Q(N)** (= P_f in Ch. 4)|**Q(0)** (= P⁻(0))|**Terminal cost ↔ initial uncertainty.** It is the boundary condition of the Riccati recursion, at the end for the regulator and at the start for the estimator.|
|**K**|**−L̃'**|**Feedback gain ↔ filter gain.** K maps state to input. L̃ maps measurement innovation to state correction. The minus sign comes from the sign convention u = Kx versus x̃⁺ = (A − L̃C)x̃.|
|**A + BK**|**(A − L̃C)'**|Closed-loop matrix ↔ estimation-error dynamics (transposed).|
|**x**|**ε**|**State ↔ estimate error.** The regulator drives x → 0. The estimator drives the error x − x̂ → 0.|

## The stability conditions (second table)

Both problems need R > 0 and Q > 0. The remaining conditions swap the same way:

- **Regulator (A,B) stabilizable ↔ estimator (A,C) detectable.** Can you push the unstable modes? ↔ Can you see the unstable modes?
- **Regulator (A,C) detectable ↔ estimator (A,G) stabilizable.** Does the cost see the unstable modes? ↔ Does the noise excite them? If not, the covariance behaves badly. Exercise 4.18 explores this.

## Why it matters

1. **Intuition:** controllability is to observability as "cost of state deviation" is to "uncertainty of state".
2. **Theorems transfer:** a convergence proof for the regulator Riccati equation (Theorem 4.12) immediately becomes one for the estimator. Exercise 4.17 asks you to do this translation.
3. **Code reuse:** since A→A', B→C', C→G', an LQR solver gives you a Kalman gain. Call the LQR routine with (A', C', GQG', R) and transpose the result.

If it helps, I can walk through the Riccati equations side by side to show that the index mapping Π(k−1) ↔ P⁻(l+1) really produces identical recursions.

