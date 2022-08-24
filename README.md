# World-class-Quantum-chemistry-with-TKET---Quantinuum
Womanium Quantum Hackathon 2022

## Team Information: UCLA Bruinium
| Member | Email | Discord Tag | Github Username |
| ----------- | ----------- | ----------- | ----------- |
| Will Wang | qinghe1230@gmail.com | #4440 | Qinghe-Wang |
| Changling Zhao | changling@physics.ucla.edu | #4426 | changlingzhao |
| Ashley Shin | ajshin@ucla.edu | #0089 | shinj |
| Lambert Kong | lambertkk@ucla.edu | #5925 | lambertkk |
| Lajoyce Mboning | lajoycemboning@ucla.edu | #1601 | lajoycemboning |
| Amanda Younes | amandajyounes@ucla.edu | #9428 | amandajyounes |

Pitch Presenter: Lajoyce Mboning

### The Challenge
One of the direct applications of a near term implementation of quantum computing is in quantum chemistry simulation. A cost-effective way to simulate the potential energy of molecules would be monumental to theoretical chemistry research, with direct applications to real-life problems in environmental and energy storage problems. The immediate challenge lies with error correction and optimization to actually create QED circuits that are as accurate as Hartree Fock (or higher) level of theory while being cost-effective, in terms of computing and energy cost. The first step to this eventual technological goal is to optimize the solution for small molecules, come up with innovative ways to design QED circuits, and expand to larger molecular systems.

We addressed all three levels of the challenge, including running our VQE on two different backends (IBM and IonQ) for beginner, shortening the depth of the circuit using the Jordan-Wigner Mapper for knowledgeable, and calculating the ground state energy and potential energy surface of the LiH molecule. We calculated the potential energy of LiH, a molecule that can be used to store hydrogen and reacted with water to extract hydrogen molecules for fuel (LiH+H2O->H2+LiOH). The goal was to use a variational quantum eigensolver (VQE) to find the ground state energy by relying on the variational principle of quantum mechanics and simulating a Hamiltonian on a quantum computer. We researched how to map the second quantization Hamiltonian to a VQE Hamiltonian, looking at papers from UCSB/Google and IBM, from which the one provided by Kandala et al is what we used for our LiH simulation (1). We got the expected energy for the valence configuration of LiH as was confirmed by their results.

### Methods
We first familiarized ourselves with the packages and backends by running a VQE on the hydrogen molecule, which has a much simpler Hamiltonian than LiH. We used functions provided from the pytket manual (4), and used a classical dual annealing optimizer to find the ground state of H2 using a noisy simulator and a Hamiltonian from Utkarsh et al (2). Using this method, we achieved a value that agrees with the number given in the referenced paper on hardware-efficient VQE. We also diagonalized the Hamiltonian and confirmed our result, then went through the same process for LiH. We also generated a Hamiltonian inspired from qiskit (3) and calculated a potential energy surface for various bond lengths, using our VQE and ansatz through TKET. It should be noted that the final ground state energies for the potential energy surface of LiH is calculated by summing the computed energy, energies extracted by applied transformers, and nuclear repulsion energy. 

### Hamiltonian
We used a Hamiltonian in Kandala et al (1) to simulate the valence configuration of the LiH molecule, written for four qubits in terms of Pauli operators. In our code, we converted this to tensor operators and diagonalized to get the lowest eigenvalue of our Hamiltonian (-0.88 Hartree) to use as a reference for our VQE result. Since the Hamiltonian from Kandala et al only accounted for the valence configuration of LiH, essentially excluding the energetic contribution from the filled orbitals, the total ground state energy was different from the expected value (around -7.8). We researched a separate Hamiltonian (2) to account for the offset due to the closed orbital contribution (the contribution from Li+), which combined with the valence Hamiltonian should give a close-to-accurate total ground state energy (Refer to the pitch deck for more information on this approach).

Additionally, we wrote a Hamiltonian using the qiskit nature package for various bond lengths, with some code taken from (3). The results agreed with our other value for the ground state energy at the equilibrium geometry. This method allowed us to map out a potential energy surface for different bond lengths, implemented in TKET with our own hardware-efficient ansatz.

### Ansatz Circuit
We used a one layer hardware-efficient ansatz circuit with 24 single-qubit gates and 3 two-qubit gates (1). Each single-qubit gate had a single optimizable parameter, for a total of 24 parameters. 

We used a classical dual annealing optimizer with our one layer ansatz circuit as the cost function, taking some code from (4). We also ran it with a second layer, which added 3 two-qubit gates and 12 single-qubit gates but gave no noticeable improvement in performance.

Although we did not use the provided ansatz circuit for our VQE, we did simplify it using the optimization passes provided by TKET. We were able to reduce the circuit depth for Hydrogen from 83 to 52 and the CNOT depth from 65 to 44. For the much more complex LiH circuit, we reduced the circuit depth from 8573 to 7330 and the CNOT depth from 9834 to 7964. At these circuit depths and with this number of qubits, we determined that a hardware-efficient ansatz would be more feasible given the time constraint and current hardware noise levels. For future implementation, we could switch to a chemistry inspired ansatz using a method such as a unitary coupled cluster (UCC) ansatz.

### Backends
Since we used a hardware-efficient ansatz, we did not have to do much optimization for different backends. The qubit geometry we assumed is a linear array with connections only between adjacent qubits, and only a small number of two-qubit gates which would contribute to noise. 

This geometry is common and easy to achieve, so our code can run on many backends without significant modifications. Since we did not assume much connectivity between qubits, compiling for different backends would not require adding two-qubit gates and thus more noise.

We chose a noisy simulator for most of our work, but our code is general to many different backends. With minor modifications, we could run our code on almost any hardware and achieve accurate results even with a moderate amount of noise. Since our gate and connectivity requirements are so simple, we could use pytket to modify our circuits and optimize for almost any standard available backend with good success.

We also ran our code for different backends, including AerBackend and ProjectQBackend to find the ground state energy with comparable results. So, we were able to use TKET to run VQE on multiple backends.

### Results
Our VQE results were -7.23 Hartree for Li+ and -0.84 Hartree for LiH valence configuration, giving a total energy of -8.07 Hartree for LiH. This compares very well to the exact solution obtained from diagonalizing the two Hamiltonians, which gives a ground state energy of -7.23 Hartree for Li+ and -0.88 Hartree for LiH valence configuration, giving a net ground state energy of -8.11 Hartree for Li-H. Both values (VQE -8.07 and exact -8.11 Hartree) are close to the published number of -7.8 Hartree for the full VQE simulation of the LiH molecule. The difference in value is unfortunately bigger than the chemical accuracy (less than 0.0016 Hartree) that we were aiming for, but we have ideas to improve it with more time, as outlined in the next section. It should be noted that our VQE value is within about 0.2 Hartree or about 2% of the actual LiH ground state energy of approximately -7.8 Hartree (1). Our simulated potential energy surface also agreed with these values and is in a graph in the pitch deck.

### Future Work
Our approach was different from the approaches we found in papers in that we used VQE to calculate the energy of the core electrons by simulating a Li+ ion rather than simply adding a theoretical offset to our valence value. The energy difference can be improved by adding more layers or allowing more optimization iterations. We could also account for the difference in energy for the (Li+(1s2) + Li-H(2sigma1)) configuration by running Li+ at a larger radius (presumably the 1s2 configuration of the bound lithium would have a larger radial wavefunction than in the cation). This method will improve in accuracy with bigger atoms, where the difference between the filled orbitals of the molecule do not differ too much from the linear combination of the atomic contributions individually. However, it is possible that the simulator noise limits our result to a similar level. We could also attempt a chemistry inspired ansatz, which could improve the error between our simulated value and the result given by diagonalizing the Hamiltonian.

To improve the speed of our calculation, we could simplify the Hamiltonian by removing terms that have little effect on the end result, and as a result get rid of redundant or less important gates. This would extend well to lowering the amount of computation required for simulating larger molecules. However, this would also likely lower the accuracy of our simulation if not done carefully, so any modifications would have to be motivated by an understanding of the origin and effect of each term involved. This plays well into the interests and skills of our combined team of chemists and physicists.

### References
(1) Kandala, Abhinav, et al. "Hardware-efficient variational quantum eigensolver for small molecules and quantum magnets." Nature 549.7671 (2017): 242-246.

(2) Utkarsh Singh, Shubham Kumar, Bikash K. Behera, and Prasanta K. Panigrahi, “Efficient Simulation of Charged Systems Using Variational Quantum Eigensolver”

(3) “Qiskit Nature Tutorials.” Qiskit Nature Tutorials - Qiskit Nature 0.4.4 Documentation, https://qiskit.org/documentation/nature/tutorials/index.html. 

(4) “Running on Backends.” Running on Backends - Pytket-Manual Documentation, https://cqcl.github.io/pytket/manual/manual_backend.html#expectation-value-calculations.
### Supplementary Information
In addition to this file, we have uploaded a pitch deck which summarizes the components of our solution and results. This file focuses on the specifics of our implementation and process.
