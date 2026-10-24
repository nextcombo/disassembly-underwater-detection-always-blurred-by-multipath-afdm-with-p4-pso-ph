# Disassembly | Underwater Detection Always "blurred" by Multipath? AFDM with P4 + PSO phase perturbation significantly improves detection probability [Code link at the end]

[ISAC Communication Perception Integrated]
Disassembly | Underwater Detection Always "Blurred" by Multipath? AFDM with P4 + PSO phase perturbation Detection probability significantly improved (with simulation ideas and code snippets attached). High probability of having fallen into this pit: The autocorrelation sidelobes in the waveform are quite low in simulation. It looks pretty nice. 1. Throw it into the "multipath + Doppler" acoustic channel. Matched filter output immediately sprouted several "ghost" sidelobes. Weak targets are directly submerged. Fully drifting in range and speed measurement.
The issue often lies in the receiver's algorithm, rather than the blurred function of the transmitted waveform.
This article aims to dissect the complete thought process presented in an IEEE GLOBECOM 2025 paper: using AFDM as the underlying waveform, employing a wideband fuzzy function (WAF) as the "examination instrument," and adding a segment of PSO-optimized bounded phase perturbation (ORP-P4) on top of the P4 phase-encoded basis. This reduces the peak sidelobes of the weighted WAF. The core code snippet provided at the end of the article demonstrates the logic behind this approach, allowing you to replicate the link yourself once you understand the logic.
I. Background: ISAC has entered the underwater domain, why it becomes more challenging to integrate communication and perception (ISAC). ISAC on the ground is already a hot topic, but underwater ISAC is still in an early exploration phase. The reason is very realistic:
Slow speed (about 1500 m/s): The target's low-speed motion will result in considerable Doppler shifts, and the echoes are time-scale stretched, not simply frequency shifted. The narrowband fuzzy function will be distorted.
Dual-selective channels: multipath, time-varying, large propagation loss, severe Doppler spread, OFDM's pride in subcarrier orthogonality is easily penetrated by Doppler in the underwater environment.
Bandwidth, power, and real-time computing capabilities are all limited, preventing the arbitrary resource allocation that can be achieved with terrestrial 6G.
The key technical bottleneck of the integrated underwater detection and communication (IUDC) system, one of which is waveform design + coding design. Adaptive Frequency Division Multiplexing (AFDM) has been discussed repeatedly in recent years because it can achieve near-OTFS bit error rate (BER) in high maneuverability/dual-extension channels with lower complexity and less pilot overhead. This article takes this a step further by focusing on the domain symbol sequence (i.e., "coding") in the Direct Sequence Adaptive Frequency Modulation (DAFT) domain.
Three waveforms are compared in one sentence:
Waveform
Modulation domain
Multipath resistance
Complexity/Pilot overhead
Underwater compatibility
OFDM
Frequency domain
Weak (ICI)
Low
General
OTFS
Delay-Doppler domain
Strong
Higher
Good but heavy
AFDM
DAFT (chirp) domain
Stronger
Lower
Cost-effective
Here's an important point that might be misunderstood: The entire optimized "surgery table" operates at the transmitter side for encoding, with almost no changes required at the receiver side. Why is this designed? Because the detection and communication share the same waveform, any modification as long as it involves the receiver's demodulation chain (such as adding equalization, adding pilot, switching constellation), will require bandwidth or computational resources, and would make the "integrated" name not fit the description. By concentrating the problem to "distortion of phase modulation in DAFT domain symbols," the receiver only needs to maintain the original DAFT + P4 demodulation process, with performance loss concentrated on BER, and can be adjusted with a single knob. Such a modification approach is very suitable for typical IUDC scenarios such as marine resource exploration, underwater robotic cluster communication, and submarine and unmanned surface vehicles navigation and communication—they all need the same acoustic resource to "measure accurately and transmit stably."
Second, the medical instrument: What should we look at when inspecting the performance of a broadband fuzzy function WAF in underwater echo sounder? It depends on the shape of the fuzzy function on the plane of "time delay × velocity." Underwater echoes are scale-extended, so a broadband version is needed:
Discreteized after sampling time delay (for time delay sampling, for scaling index, for signal length):
Note that subscripts are not always integers, and interpolation is essential in engineering—this is both a key point to achieve and the most common oversight for beginners. An ideal WAF is "pin-sharp": a single peak at the origin, cleanly edged like a floor, ensuring accurate timing (distance) and velocity measurements without any ghosting.
III. Key Observation: The encoding directly determines the WAFAFDM transmitter to pass through IDAFT with DAFT domain symbols.
(2) Result in
Among the phase terms
See? In equation (4), the double summation of symbols—the shape of the symbol sequence determines the WAF. Therefore, "poor waveform detection performance" can be translated as "inappropriate encoding selection," which opens up opportunities for optimization.
Additionally, clarify the positive and negative directions: formulas (1)–(5) are for "positive problems," whereby the WAF is derived from the emission waveform; ORP-P4 addresses "negative problems"—given a desired cleaner WAF target, it inverts how to adjust the symbol phase. Such negative problems have almost no closed-form solutions, necessitating numerical iteration to approximate the solution. This is precisely why PSO is introduced later in the text. By integrating these two approaches, the overall structure of the article becomes very clear.
IV. P4 coding: Good foundation, but not able to withstand underwater conditions. P4 is a classic radar low sidelobe multi-phase code, with phases being quadratic:
Its autocorrelation sidelobes are low and it is resistant to interference, making it naturally suited for ISAC. However, underwater scenarios have exposed two weaknesses:
1  
Multipath will raise the PSL (Peak Sidelobe Level) - ghost sidelobes directly reduce detection accuracy.
2  
Doppler distortions can distort phase structures—both perception and communication suffer.
The P4's inherent low crosstalk is due to its phase incrementing in quadratic manner, equivalent to adding a "linear frequency-hopping sweep" layer within the sequence. This allows for the self-correlation energy of different code chips to cancel each other out in the sidelobe region, focusing the energy into the main lobe. However, this property relies on the "pure" phase structure: when multipath occurs, it effectively combines multiple P4 echoes with different delays, leading to a contamination of the second-order phase, rendering the cancellation mechanism ineffective, naturally raising the sidelobes. Understanding this physical reasoning, one realizes that "compensating for phase" is not a matter of superstition but rather targeted treatment.
V. ORP-P4: Adding perturbation but "knowing how to add"
The author's solution involves adding a bounded random phase perturbation on top of the P4 phase:
Carries communication data (such as 16-QAM), essentially achieving through "data × P4 × perturbation". The optimization goal is to suppress the peak of the sidelobe normalized by weight:
Weights are set as follows:
Here's the engineering intuition: giving "the sidelobes close to the main lobe" higher weights, prioritizing those most likely to be misjudged as targets; if you want to specifically optimize a certain delay-Doppler area, replace it with the corresponding window (the paper (15) is an equivalent formulation with an auxiliary variable). Why constrain the amplitude of the perturbation? Because the receiver doesn't know, and the perturbation is extra noise to the receiver, too much will definitely lead to a BER collapse. So, it's essentially a "detection—communication" dial.
VI. PSO Solving: Position vector is the problem of phase perturbation (12) highly nonlinear and locally very many, the author uses PSO. Particle position/velocity:
Velocity-position update:
Inertia weight linearly decreasing, exploring in the early stage and converging in the later stage:
Pseudocode (recommended to look at equations (12)–(14)):
Algorithm: ORP-P4 (PSO Search for bounded phase perturbation) input: Symbolic sequence x[0..N-1], Parameters c1, c2, Perturbation limit θmax,   Number of particles Np, Maximum iteration T, Learning factor η1, η2, Inertia weight boundary ωmax/ωmin1 for n = 0..N-1: φ[n] = π·n²/N − π·n     # P4 Phase, formula(9)2 for i = 1..Np:                # Initialization   θi(0) ← Uniformly random [−θmax,θmax] Evaluation of g(θi(0)) # Equation (13): Weighted sidelobe peak 3 pbest_i←Initial state; gbest ← Globally optimal particles4 for t = 0..T−1:   ω(t) ← ωmax − (ωmax−ωmin)·t/T       # Formula(19)   for i = 1..Np:    vi(t+1) ← ω·vi(t) + η1r1(pbest_i − di(t)) + η2r2(gbest − di(t)) # Formula(18)    di(t+1) ← di(t) + vi(t+1)    θi[n](t+1) ← clamp To [−θmax,θmax] # Constraints enforced
Evaluate g(θi(t+1));Updating pbest_i, updating gbest, satisfying convergence criterion then exiting 5θopt←gbest corresponding phase perturbation6x'[n]←x[n]·exp(j(φ[n]+θopt[n]))#equation (11)7s[k]←IDAFT(x', c1, c2, N) # Eq. (3) Core code snippet (MATLAB, (1) P4 Sequence Generation (Equation 9–10):
function code = p4_code(N)  n = (0:N-1).';  phi = pi*n.^2/N - pi*n;  % Equation (9) code = exp(1j*phi); % Equation (10) end\n② ORP-P4 Coding (form) 11)——Note: disturbances should only be applied once, do not apply P4 Phase multiply twice:   
function xPrime = orp_p4_encode(x, theta)  x  = x(:); theta = theta(:);  assert(numel(theta) == numel(x));  n  = (0:numel(x)-1).';  phi = pi*n.2/numel(x) - pi*n; % Equation (9) xPrime = x .* exp(1j*(phi + theta)); % Equation (11): x[n]e{j(φ[n]+θ[n])}end\n③ AFDM Modulation IDAFT (Formula 3)——Use chirp Premultiplication + IFFT Fast realization:   
function s = afdm_idaft(x, c1, c2)  x = x(:); N = numel(x); n = (0:N-1).'; k = (0:N-1).'; pre = exp(1j*2*pi*c2*n.2) .* x; % Multiply e{j2πc2n²} s = exp(1j*2*pi*c1*k.2) .* (sqrt(N)*ifft(pre)); Multiply by e{j2πc1k²}end\n④ Weighted sidelobe target function (Equation 12–14) - Excluding the main lobe, take the maximum:
function score = waf_psl_objective(theta, dataSymbols, cfg)  xPrime = orp_p4_encode(dataSymbols, theta);   % Formula(11)  s   = afdm_idaft(xPrime, cfg.c1, cfg.c2);   % Formula(3)  psi  = wideband_ambiguity(s, cfg.psoDelays, cfg.psoVelocities, cfg);  [delayGrid, velocityGrid] = ndgrid(cfg.psoDelays, cfg.psoVelocities);  alphaGrid = (cfg.soundSpeed - velocityGrid)./(cfg.soundSpeed + velocityGrid);  betaGrid = cfg.N*(alphaGrid - 1);        % α_β = 1 + β/N  weight  = 1./(1 + delayGrid.2 + betaGrid.2); % Formula(14)  valid = (delayGrid ~= 0) & (abs(betaGrid) > 10*eps); % Formula(13): Exclude m=0, β=0  weightedWaf = weight .* abs(psi);  score = max(weightedWaf(valid)); % Only the weighted maximum value is taken for sidelobe samples.end\n (wideband_ambiguity The core of the algorithm is to perform a linear interpolation and then sum it with the corresponding terms. (2)。)  
VIII. How to View Simulation Results:
1. Paper Parameters:,,, Signal Duration 10 ms, Bandwidth 12.8 kHz, kHz, Radial Velocity m/s; PSO: 50 particles, 100 iterations,; False Alarm Probability. The conclusions drawn from the three key figures are as follows:
Comparison items
P4
ORP-P4 (the larger the better)
WAF shape
Outer side of origin has sidelobe energy
Closer to "pin" shape, rapidly decaying in all directions
Autocorrelation sidelobes
Higher
Lower, main lobe narrower
Detection probability (Peak-to-Average Power Ratio detection)
Medium
Higher, and better than the baseline for 16-QAM
BER (Received End Unknown Disturbance)
≈ 16-QAM
Approximatedly increased, worsening monotonically
In a single sentence, summarize the performance rules: the higher the disturbance limit is opened, the cleaner the WAF is, and the better the detection is, but the BER is worse. The choice is to choose the system operating point - this is precisely one of the most practical aspects of this method.
It should be emphasized that this rule does not hold under any specific set of parameters: in the setting of a single symbol 10 ms, uncoded Gray 16-QAM, the BER monotonically worsens at SNR = 0, 5, 10, 20 dB across four levels; this shows that "using some communication margin to enhance detection capability" is a trend, not a coincidence under specific parameter settings.
Calibration note: To compare two easily misaligned switches, align them first; otherwise, the curves will not be on the same comparison basis:
Detection aperture: The paper uses "Peak-to-Average Power Ratio detection" - performing mutual correlation between the received and transmitted waveforms, taking the ratio of peak to mean value as the test statistic; the threshold is not determined by guesswork but by running Monte Carlo simulations under the assumption of only noise, determining the quantile based on false alarm probability; each SNR point then repeats thousands of trials with "target echo + noise" to statistically calculate the detection probability, corresponding to a single symbol 10 ms time window.
BER Angular Field of View: Uncoded Gray 16-QAM; Interference Dominance is unknown at the receiver end; the paper evaluates BER-SNR separately on five levels, and draws a curve of "BER vs. SNR under fixed conditions". In engineering terms, this corresponds to the configurations in `config_paper.m` with the `thetaGrid` and `berSnrDb` groups.
P4 phase is only multiplied once at the encoding end. Equation (11) already includes P4 phase, and if it's multiplied again, it equals doubling the phase, which completely changes the sequence properties. The most common error in code is this one.
2  
The objective function must exclude the main lobe. Equation (13) explicitly limits and, including the peak value, will pollute the objective function into a constant.
3  
Normalize the distortion: Equation (13) uses normalization of WAF (divided by), wherein the WAF is normalized first by peak value and then weighted, otherwise, the weights will clash with the sidelobes.
4  
Non-linear subscript interpolation is required: The equation (2) does not have an integer grid, and direct indexing will result in errors. Linear interpolation is the most cost-effective default approach.
5  
Mesh is integral: Sidelobe optimization is only effective on the time-delay-Doppler mesh you define. The closer the mesh to the actual working area, the better. Do not expect a single mesh to cover all scenarios.
6  
To reproduce the entire link chain "16-QAM modulation → P4/ORP-P4 encoding → IDAFT → WAF/autocorrelation/detection/BER" in one's own project, it is recommended to first perform three minor tasks: ① verify that the QAM modulation and demodulation processes are consistent; ② ensure that DAFT/IDAFT can perfectly reconstruct; ③ verify that when there is no disturbance, ORP-P4 is equivalent to P4. Once these three checks pass, then it makes sense to adjust the PSO later on. Tenth, what other paths can this approach take? Looking at the chain "encoding → WAF → detection", ORP-P4 is actually just a starting point:
Optimization Transformer: Switch from PSO to Differential Evolution, Grey Wolf or Projection-based Interior Point Methods for more robust convergence when dealing with larger problems and more rugged objective functions.
Weight Surface Change: By adapting the weights in equation (14) to a window that can be updated adaptively, we can implement a two-stage strategy of "first detecting the target, then focusing on suppressing sidelobes around it."
Modulation Change: When switching from 16-QAM to PSK, higher-order QAM, or even sparse code multiple access superposition, reoptimizing the perturbation vector is all that's needed, with the method framework fully reusable.
Reference Paper: Q. Niu, D. Ramírez, W. Shi, and Q. Zhang, "Underwater Detection and Communication Integrated Waveform Design Based on P4 Encoding," 2025 IEEE Global Communications Conference (GLOBECOM), pp. 2607–2612, DOI: 10.1109/GLOBECOM59602.2025.11431764.
Next Upcoming Post: The "relative" AFDM cousin, OTFS, who really shines underwater? We'll pull out the waveforms in the delay-Doppler domain for a head-to-head comparison.

## Images

![Disassembly | Underwater detection is often "blurred" by multipath interference? AFDM with P4 + PSO phase perturbation significantly improves detection probability [Code link at the end]](01.jpg)
![Disassembly | Underwater detection is often "blurred" by multipath interference? AFDM with P4 + PSO phase perturbation significantly improves detection probability [Code link at the end]](02.jpg)
![Disassembly | Underwater detection is often "blurred" by multipath interference? AFDM with P4 + PSO phase perturbation significantly improves detection probability [Code link at the end]](03.jpg)
![Disassembly | Underwater detection is often "blurred" by multipath interference? AFDM with P4 + PSO phase perturbation significantly improves detection probability [Code link at the end]](04.jpg)
![Disassembly | Underwater detection is often "blurred" by multipath interference? AFDM with P4 + PSO phase perturbation significantly improves detection probability [Code link at the end]](05.jpg)
![Disassembly | Underwater detection is often "blurred" by multipath interference? AFDM with P4 + PSO phase perturbation significantly improves detection probability [Code link at the end]](06.jpg)
![Disassembly | Underwater detection is often "blurred" by multipath interference? AFDM with P4 + PSO phase perturbation significantly improves detection probability [Code link at the end]](07.jpg)

<!-- PAYMENT_BLOCK -->

## Get this dataset

- Payment link (Stripe): [Buy now](https://buy.stripe.com/3cs8yP7sY87d0vu9AB)
- Payment QR code: ![Payment QR code](pay.png)

<!-- /PAYMENT_BLOCK -->
