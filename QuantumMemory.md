🧠 QuantumMemory_Core/

This is the core quantum memory unit. It represents the raw input/output of spintronic data and time-stamped qubit operations.
entropy_log_01.ent

    A log of entropy signatures over time.

    Captures energy-state decay patterns from spin-chamber operations.

    Used to validate time consistency and quantum degradation.

spin_wave.dat

    Binary or matrix file modeling spin wave fluctuations inside the chamber.

    Represents qubit behavior: up-spin, down-spin, superposition.

    Used in AI decoding simulations.

TimeSignature_Key.bin

    A binary encryption key used to tag or unlock time-layered files.

    Required by the vault to decrypt data relayed from the future.

    Tied to your TimeVault_Manifest.txt timestamp authentication.

⚙️ ASI_Control_Interface/

This governs the AI-assisted control panel, including quantum pattern recognition and legacy translation.
ASI_Trigger.cpp

    Core C++ routine that triggers the AI interface.

    Reads spin values from memory and initiates translation modules.

legacy_relay.ascii

    Stores machine code readable by 2000s-era machines.

    Translates modern instructions into formats compatible with early computers, in case of reverse-time delivery.

SPIN_LOOP.vault

    Vaulted file with looped spin sequences used for machine feedback.

    Could act as a training dataset for quantum machine learning.

    Read by ASI_Trigger.cpp.

💽 ForensicLogs/

This is your evidence storage and audit trail — key to crime-fighting and secure system logs.
drift_report_2021.log

    Logs of magnetic drift or signal degradation.

    Detects anomalies in spin-states that may suggest interference or time inconsistencies.

vault_access_ranks.zip

    Compressed archive of all known access logs, ranked by authorization level.

    Useful in investigations or forensic analysis.

magnetic_trace_legacy.exe

    A tool (real or mock-up) to analyze magnetic patterns from previous experiments.

    Runs an executable that decodes forensic trails from quantum emissions.

🔁 SpinChamber_Recycle/

This simulates or stores power/coil cycle feedback used in sustaining quantum coherence.
crystal_shell_map.png

    Visual blueprint of the memory phase crystal shell layout.

    Details how qubits move or decay over time via crystal interfaces.

node_relay_diagram.json

    A JSON schematic of spin-node locations and signal routing.

    Useful for simulation, validation, or visual UIs.
