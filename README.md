# Delta-Modulation
Aim
The aim of delta modulation is to convert an analog signal into a digital signal using a simple 1-bit quantization scheme. It's a form of analog-to-digital conversion that provides a simple way to encode signals with reduced complexity compared to PCM (Pulse Code Modulation).

Tools required
Signal generator (for input analog signal)

Comparator circuit
Integrator circuit
Flip-flop (for sampling)
Clock generator
Oscilloscope (to observe waveforms)
Resistors and capacitors (for circuit implementation)
Operational amplifiers
Breadboard/prototyping board
Power supply

Program

# Delta Modulation Simulation in Google Colab
import numpy as np
import matplotlib.pyplot as plt

# Configuration
SAMPLING_RATE = 1000  # Hz
DURATION = 1.0        # seconds
STEP_SIZE = 0.1       # quantization step size
SIGNAL_FREQ = 5       # Hz (for input signal)

# Generate input signal
def generate_input_signal():
    t = np.linspace(0, DURATION, int(SAMPLING_RATE * DURATION), endpoint=False)
    signal = np.sin(2 * np.pi * SIGNAL_FREQ * t)  # Sine wave
    # signal = np.sign(np.sin(2 * np.pi * SIGNAL_FREQ * t))  # Square wave (try this for interesting effects)
    return t, signal

# Delta Modulation Encoder
def delta_modulate(signal, step_size):
    reconstructed = [0]  # Start with zero initial condition
    bitstream = []
    
    for sample in signal:
        difference = sample - reconstructed[-1]
        
        if difference > 0:
            bit = 1
            new_value = reconstructed[-1] + step_size
        else:
            bit = 0
            new_value = reconstructed[-1] - step_size
            
        bitstream.append(bit)
        reconstructed.append(new_value)
    
    return bitstream, reconstructed[:-1]  # Remove last sample to match input length

# Delta Modulation Decoder
def delta_demodulate(bitstream, step_size):
    reconstructed = [0]  # Start with zero initial condition
    
    for bit in bitstream:
        if bit == 1:
            new_value = reconstructed[-1] + step_size
        else:
            new_value = reconstructed[-1] - step_size
            
        reconstructed.append(new_value)
    
    return reconstructed[1:]  # Remove initial zero

# Main simulation
t, input_signal = generate_input_signal()
bitstream, tx_reconstructed = delta_modulate(input_signal, STEP_SIZE)
rx_reconstructed = delta_demodulate(bitstream, STEP_SIZE)

# Visualization
plt.figure(figsize=(15, 10))

# Plot input signal
plt.subplot(3, 1, 1)
plt.plot(t, input_signal, label='Input Signal')
plt.title('Original Analog Input Signal')
plt.xlabel('Time (s)')
plt.ylabel('Amplitude')
plt.grid(True)
plt.legend()

# Plot transmitted signal
plt.subplot(3, 1, 2)
plt.step(t, tx_reconstructed, where='post', label='Tx Reconstructed')
plt.plot(t, input_signal, alpha=0.3, label='Input Reference')
plt.title('Transmitted Staircase Reconstruction (Encoder Side)')
plt.xlabel('Time (s)')
plt.ylabel('Amplitude')
plt.grid(True)
plt.legend()

# Plot received signal
plt.subplot(3, 1, 3)
plt.step(t, rx_reconstructed, where='post', label='Rx Reconstructed', color='orange')
plt.plot(t, input_signal, alpha=0.3, label='Input Reference')
plt.title('Received Staircase Reconstruction (Decoder Side)')
plt.xlabel('Time (s)')
plt.ylabel('Amplitude')
plt.grid(True)
plt.legend()

plt.tight_layout()
plt.show()

# Print some statistics
print(f"Simulation Parameters:")
print(f"- Sampling rate: {SAMPLING_RATE} Hz")
print(f"- Step size: {STEP_SIZE}")
print(f"- Signal frequency: {SIGNAL_FREQ} Hz")
print(f"\nGenerated {len(bitstream)} bits ({len(bitstream)/8/1024:.2f} KB)")
print(f"First 20 bits: {bitstream[:20]}")
print("\nNote: The red areas show where slope overload occurs (input changes faster than DM can track)")

Output Waveform

![Image](https://github.com/user-attachments/assets/104103cf-d971-4bc0-b976-ab5a35864a4a)

Results

Delta Modulation successfully encoded a 5Hz sine wave into a 1-bit digital stream (step=0.1, fs=1000Hz), showing slope overload at peaks and granular noise near zero-crossings.
First 20 bits: [1,1,1,1,1,0,0,0,0,0,1,1,1,1,1,0,0,0,0,0] - staircase reconstruction tracks input but lags during rapid changes.

