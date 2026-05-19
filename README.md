# Hospital Emergency Room (ER) Simulation

**Authors:** Neria Yaakov and Bezalel Spolter  
**Date:** 2025  

## Project Overview and Importance
This project involves a Discrete Event Simulation (DES) of a hospital Emergency Room (ER). The objective is to understand and optimize the system's performance under various load conditions and arrival rates. ERs are among the most critical systems in healthcare, where life-saving decisions are made under strict time constraints, limited resources, and volatile demand. 

The simulation models the complete patient journey, from arrival to discharge, admission, or death. There is differentiation between injury severity levels (Easy, Moderate, Severe) and prioritizing critical cases. Built upon insights gathered from an interview with a senior ER physician in the US, the model incorporates resources (doctors, nurses, beds), service times, queues, and critical events. This simulation serves as a data-driven decision-making tool for resource allocation, operational prioritization, and healthcare planning under uncertainty.

## Research Questions
To evaluate system performance under varying conditions, the following research questions were defined:
* What is the average Length of Stay (LOS) for a patient in the system, the LOS for a severe patient, and the average number of severe patient deaths? How are these affected by various factors in the treatment process?
* What is the optimal number of doctors and nurses to maximize ER efficiency?
* What is the optimal allocation of resources for blood tests and X-ray machines to maximize system throughput?
* How does the patient admission policy impact the mortality rate in the ER?
* How do the main KPIs change in response to different arrival rates?

## System Description
Based on a real-world ER, a Steady-State simulation model was built. The model tracks a patient from arrival to departure. The system includes several treatment stations: initial treatment, blood tests, X-rays, and final doctor diagnosis. Each station has associated resources and queues. The model accounts for three severity levels and applies probability-based processes and randomized service times based on defined distributions.

### System Components
* **Entities:** Patients with varying conditions: Easy, Moderate, Severe.
* **Resources:**
  * Beds: 90
  * Doctors (Shifts): 10 during the day (08:00-20:00), 8 at night (20:00-08:00).
  * Nurses: 10 during the day, 8 at night.
  * X-Ray Machines: 10 during the day, 5 at night.
  * Blood Test Kits: 10 during the day, 8 at night.
* **Queues:**
  * Initial Treatment: Requires an available bed, nurse, and doctor.
  * X-Ray: Requires an available X-ray machine.
  * Blood Test: Requires an available blood test kit.
  * Doctor Diagnosis (Final Treatment): Requires an available doctor.
* **Event Types:** Patient Arrival, Initial Treatment, X-Ray, Blood Test, Final Treatment/Diagnosis, Death of a severe patient.

### Mathematical Distributions and Inputs
**Patient Arrivals:**
Patients arrive via three Poisson processes based on severity:
$P_E(t) \sim Pois(\lambda_E \cdot t)$
$P_M(t) \sim Pois(\lambda_M \cdot t)$
$P_S(t) \sim Pois(\lambda_S \cdot t)$

The total number of patients is the union of these processes, forming a non-stationary Poisson process:
$P(t) \sim Pois(\int_0^t \lambda(s)ds)$

With the overall rate function defined as:
$$
\lambda(t) = \begin{cases} 5 & 0 \le t < 8 \\ 15 & 8 \le t < 20 \\ 5 & 20 \le t < 24 \end{cases}
$$

**Service Times (Exponentially Distributed):**
* **Initial Treatment (TI):** $TI_E \sim exp(20)$, $TI_M \sim exp(2)$, $TI_S \sim exp(4)$
* **X-Ray ($T_{IMG}$):** $T_{IMG} \sim exp(4/3)$ (Average of 45 mins, regardless of severity)
* **Blood Test ($T_{BT}$):** $T_{BT} \sim exp(2)$ (Average of 30 mins, regardless of severity)
* **Final Treatment (TF):** $TF_E \sim exp(4)$, $TF_M \sim exp(2)$, $TF_S \sim exp(4/3)$
* **Time until Death (Severe patients only):** Uniformly distributed $T_{DEATH} \sim U[0,24]$

## Assumptions
To simplify the model while highlighting its scope and limitations, the following assumptions were made:
1. **Parameter Validity:** Distributions and resource quantities are based on an interview with a senior ER doctor with over 20 years of experience in the US.
2. **Static Condition:** A patient's condition does not change during the simulation, except for severe patients who may die.
3. **Continuous Workflow:** Staff works at a constant pace without breaks.
4. **No Failures:** X-rays and blood tests are executed without errors or machine breakdowns.
5. **Instant Communication:** Information transfer and patient movement between stations happen instantly.
6. **Mortality Window:** A severe patient is at risk of dying at any point before reaching the final doctor's diagnosis.

## Experiment Design
The experiments are divided into three categories:
1. **Arrival Rates:** Testing higher and lower arrival rates to check system stability under varying loads.
2. **Resource Quantities:** Altering the number of doctors, nurses, beds, and equipment to find optimal utilization.
3. **Admission Policy:** Comparing a strictly "FIFO" (First-In, First-Out) policy against a policy that prioritizes "Severe" patients at every queue.

## Challenges and Solutions
In steady-state simulations, we addressed two primary statistical issues:
* **Warm-up Period (Relaxation Time):** The system takes time to reach a steady state. We ran 25 replications and determined that the system stabilizes after 100 hours. Data collection only begins after this $d=100h$ warm-up period to prevent skewed results.
* **Autocorrelation:** To solve data correlation issues, metrics were collected using a Batch Means approach (dividing data into 24-hour batches).

## Key Results
**Baseline Model Metrics (Post Warm-up):**
| Metric | Average | 95% Confidence Interval |
| :--- | :--- | :--- |
| Overall System LOS | 5.58 hours | [4.91, 6.24] |
| LOS - Easy | 4.88 hours | [4.10, 5.66] |
| LOS - Moderate | 6.50 hours | [5.72, 7.28] |
| LOS - Severe | 2.21 hours | [2.19, 2.22] |
| Queue Time - Initial Treatment | 2.98 hours | [2.31, 3.66] |
| Queue Time - Diagnosis | 1.46 hours | [1.41, 1.52] |
| Deaths per Month | 1 per replication | N/A |

*Note: The bottleneck resource is the Doctors, evident by the longest queue times at Initial Treatment and Final Diagnosis.*

## Conclusions and Recommendations
1. **Alleviate the Bottleneck:** Doctors are the primary bottleneck. We recommend increasing daytime doctors from 10 to 12. Alternatively, shift non-critical initial treatment tasks for Easy/Moderate patients to nurses to free up doctors for severe cases.
2. **Resource Optimization:** The hospital can safely reduce certain resources without harming KPIs: daytime nurses can be reduced to 5-6, blood test kits to 6, and active X-ray machines to 9.
3. **Triage Policy:** Prioritizing severe patients at all queues significantly improves outcomes. Changing to a strict FIFO policy increased the average death rate from 1 to 3.4 per month.
4. **Sensitivity to Load:** The system is highly sensitive to arrival rates. If the overall daytime arrival rate increases by just 1 patient (to 16/hour), the system becomes unstable, pushing average LOS from 5.6 hours to over 10 hours.

## Future Work: Economic and Financial Layer
A follow-up project will expand this operational DES model by adding a financial analysis layer to evaluate ED profitability. The expanded model will include:
* **Costs:** Fixed and variable costs (salaries, equipment maintenance, daily operations, and costs associated with extended patient LOS).
* **Revenue:** Treatment-based income, linking patient pathways to specific pricing structures (based on HMO rates or internal pricing).
* **ROI Testing:** Evaluating the financial impact of adding shifts, raising salaries, or buying new equipment against the clinical metrics (mortality and LOS) to find the perfect balance between economic sustainability and standard of care.
