# Autonomous HVAC Energy Optimization via EnergyPlus & Llama 3.1

An end-to-end, closed-loop AI control architecture for commercial HVAC systems. This project integrates the C++ **EnergyPlus** physics engine with a local **Llama 3.1** LLM agent via function calling to optimize dynamic temperature setpoints in real time.

---

## System Architecture

The framework operates on a synchronous runtime exchange loop using the Python bindings for the EnergyPlus C++ API (`pyenergyplus`).

1. **State Engine:** EnergyPlus steps through the timeline hour-by-hour, evaluating heat balance and thermal loads.
2. **Callback Listener:** Python intercepts every zone timestep and extracts telemetry (zone mean air temperature, power draw).
3. **Agent Inference:** Telemetry is structured as contextual messages and passed to Llama 3.1.
4. **Tool Execution:** If temperature boundaries are violated or optimization is required, Llama 3.1 invokes the `inject_setpoint` tool, triggering an actuator override (`Zone Temperature Control`) directly inside the active C++ state memory.

---

## Prompt Engineering & Tool-Calling Strategy

* **System Prompt Constraints:** The LLM is initialized with strict domain boundaries:
  > "You are an autonomous building agent. Optimize energy while keeping temp between 20C-24C. Only use the tool if the temp is outside this range."
* **JSON Schema Enforcement:** Native tool definitions force Llama 3.1 to generate structured tool calls adhering to the parameters expected by `inject_setpoint(target_temp: float)`.

---

## Latency Management & Telemetry Strategies

* **Local Inference Acceleration:** Running Llama 3.1 locally via Ollama on a GPU instance eliminates cloud HTTP network overhead, dropping token inference latency.
* **Log Aggregation & Memory Management:** Rather than feeding verbose raw standard output or multi-megabyte C++ `.err` logs into the context window, telemetry is filtered at the callback layer into lightweight Python dictionary payloads.
* **Selective Querying:** LLM calls are limited to zone state transitions, dramatically reducing total model queries without sacrificing thermal control precision.
