# LIGHT INTENSITY MONITORING SYSTEM
Using PIC18F452 Microcontroller
Embedded Systems / Microcontrollers Project Report
## 1. Introduction
A Light Intensity Monitoring System is an embedded application that senses the surrounding light level and provides a real-time, human-readable indication of how bright or dark the environment is. This project implements such a system using the PIC18F452 microcontroller, a Light Dependent Resistor (LDR) as the sensing element, a 16x2 LCD for numerical display, three LEDs for quick visual status indication, and a buzzer for audible low-light alerts.
Light intensity monitoring has practical importance in numerous real-world systems, including automatic street lighting, indoor lighting automation, greenhouse monitoring, solar panel positioning, camera exposure control, and safety systems that require a minimum ambient light level. By building this system around the PIC18F452 — a widely used 8-bit microcontroller in academic and industrial embedded projects — this project demonstrates core embedded systems concepts such as analog signal acquisition, analog-to-digital conversion (ADC), LCD interfacing, digital output control, and threshold-based decision making.
The system continuously samples the analog voltage produced by the LDR voltage-divider circuit, converts it into a digital value using the microcontroller's built-in 10-bit ADC module, scales that value into a light intensity percentage, and then uses this percentage to drive three categories of output: a numeric LCD readout, a three-level LED indicator (low / medium / high), and a buzzer alarm that activates when the surroundings become too dark.
## 2. Aims and Objectives
## Aims
•	Design and implement a real-time light intensity monitoring system using the PIC18F452 microcontroller.
•	Provide both a numeric (LCD) and a visual/audible (LED + buzzer) indication of ambient light level.
•	Demonstrate practical use of the PIC18F452's ADC module and general-purpose I/O ports.
## Objectives
•	Interface an LDR sensor with the PIC18F452 ADC channel (AN0) through a voltage-divider circuit.
•	Configure and use the internal 10-bit ADC module to digitize the analog light signal.
•	Convert the raw ADC reading into a meaningful light intensity percentage (0% – 100%).
•	Interface and drive a 16x2 LCD in 4-bit mode to display the live intensity reading.
•	Implement threshold logic to classify the intensity into Low, Medium, and High categories.
•	Drive status LEDs and a buzzer based on the classified intensity level.
•	Test the system under varying lighting conditions and verify correct behavior.
## 3. Literature Review
Several approaches exist for monitoring ambient light intensity in embedded systems. The most common sensing elements are the Light Dependent Resistor (LDR), photodiodes, and phototransistors.
LDR (Light Dependent Resistor): An LDR is a passive component whose resistance decreases as the incident light intensity increases (a property known as photoconductivity). LDRs are inexpensive, simple to interface using a basic voltage-divider circuit, and widely available — making them the most common choice for educational and low-cost embedded light-sensing projects. Their main limitations are a relatively slow response time and some non-linearity, which are acceptable for ambient-light-level monitoring applications such as this one.
Photodiodes and Phototransistors: These offer a faster response time and better linearity than LDRs, but typically require additional signal-conditioning (amplification) circuitry, increasing design complexity and cost. They are more commonly used in applications requiring fast optical switching, such as fiber-optic communication or precision light meters.
Based on this comparison, the LDR was selected as the sensing element for this project due to its simplicity, low cost, ease of interfacing with the PIC18F452's analog input pins, and sufficient accuracy for general ambient light classification (Low / Medium / High).
On the controller side, the PIC18F452 was chosen because it provides a built-in 10-bit ADC module (sufficient resolution for this application), enough general-purpose I/O pins to simultaneously drive an LCD, three LEDs, and a buzzer, and is a standard teaching platform supported by the MPLAB IDE and XC8/C18 compilers.
## 4. Proposed Solution / Methodology
The system follows a straightforward sense-process-act methodology, summarized in the steps below:
1.	Sense: The LDR, arranged in a voltage-divider configuration with a fixed 10kΩ resistor, produces an analog voltage proportional to ambient light intensity.
2.	Acquire: The PIC18F452's ADC module samples this analog voltage on channel AN0 (RA0) and converts it into a 10-bit digital value (0 - 1023).
3.	Process: The firmware scales the raw ADC value into a percentage (0% - 100%) representing the light intensity, then compares it against two configurable thresholds (20% and 60%).
4.	Display: The calculated percentage is written to a 16x2 LCD in real time, refreshed every 500 ms.
5.	Act: Depending on which threshold band the intensity falls into, the system lights the corresponding LED (Red = Low, Yellow = Medium, Green = High) and activates the buzzer if the intensity drops below the dark threshold.
System Block Diagram

<img width="750" height="583" alt="image" src="https://github.com/user-attachments/assets/5d915f28-67ab-4d1c-b778-318ecd9aeb18" />

5. Circuit Diagram and Hardware Design
The detailed schematic below shows the complete pin-level connections between the PIC18F452 and all peripheral components, including the LDR voltage-divider input, the crystal oscillator circuit, the LCD data lines, and the LED/buzzer output stage.

<img width="750" height="592" alt="image" src="https://github.com/user-attachments/assets/2b26da0c-140c-4812-ad60-0d7694ff1a30" />

## Pin Connection Summary

<img width="630" height="327" alt="image" src="https://github.com/user-attachments/assets/3e4401d1-fabc-4f45-872f-050e2a2d7d56" />

## Component List

<img width="627" height="392" alt="image" src="https://github.com/user-attachments/assets/05cd9245-21cc-490f-9478-41747236a764" />

## 7. Source Code
The complete firmware was developed in Embedded C for the PIC18F452, targeting the MPLAB XC8 / C18 compiler. The full annotated source file (main.c) is provided alongside this report. Key sections of the code are explained below; the complete listing follows.
## 7.1 Configuration and Pin Definitions
#pragma config OSC   = HS        // 20 MHz High Speed Crystal
#pragma config WDT   = OFF       // Watchdog Timer Disabled
#pragma config LVP   = OFF       // Low Voltage Programming Disabled

#define LCD_RS   PORTBbits.RB0
#define LCD_EN   PORTBbits.RB1
#define LCD_D4   PORTBbits.RB2
#define LCD_D5   PORTBbits.RB3
#define LCD_D6   PORTBbits.RB4
#define LCD_D7   PORTBbits.RB5

#define LED_LOW   PORTCbits.RC0
#define LED_MED   PORTCbits.RC1
#define LED_HIGH  PORTCbits.RC2
#define BUZZER    PORTCbits.RC3

#define DARK_THRESHOLD     20
#define MEDIUM_THRESHOLD   60
## 7.2 ADC Reading Function
This function selects the desired ADC channel, allows acquisition time, starts the conversion, and waits until it completes before returning the 10-bit result.
unsigned int ADC_Read(unsigned char channel)
{
    ADCON0 &= 0xC5;
    ADCON0 |= (channel << 3);
    Delay_ms(2);

    GO_DONE = 1;
    while (GO_DONE);

    return ((unsigned int)((ADRESH << 8) + ADRESL));
}
## 7.3 Converting ADC Value to Light Intensity Percentage
unsigned int Get_Light_Percent(void)
{
    unsigned long adcValue;
    unsigned int percent;

    adcValue = ADC_Read(0);
    percent  = (unsigned int)((adcValue * 100UL) / 1023UL);

    if (percent > 100) percent = 100;
    return percent;
}
## 7.4 Threshold-Based Output Control
void Update_Outputs(unsigned int lightPercent)
{
    if (lightPercent < DARK_THRESHOLD)
    {
        LED_LOW = 1; LED_MED = 0; LED_HIGH = 0; BUZZER = 1;
    }
    else if (lightPercent < MEDIUM_THRESHOLD)
    {
        LED_LOW = 0; LED_MED = 1; LED_HIGH = 0; BUZZER = 0;
    }
    else
    {
        LED_LOW = 0; LED_MED = 0; LED_HIGH = 1; BUZZER = 0;
    }
}
## 7.5 Main Program Loop
while (1)
{
    lightPercent = Get_Light_Percent();

    LCD_SetCursor(0, 0);
    LCD_String("Light Intensity");

    LCD_SetCursor(1, 0);
    sprintf(buffer, "%3u %%          ", lightPercent);
    LCD_String(buffer);

    Update_Outputs(lightPercent);

    Delay_ms(500);
}
## 8. Working / Algorithm Explanation
The system operates in a continuous polling loop, executing the following sequence approximately every 500 milliseconds:
1.	System Initialization: On power-up, the microcontroller configures PORTA as analog input (for the LDR), PORTB as output (for the LCD), and PORTC as output (for the LEDs and buzzer). The ADC module is configured with AN0 selected as the active channel.
2.	LCD Initialization: The LCD is initialized in 4-bit mode, the display is cleared, and a startup message is shown briefly.
3.	ADC Sampling: The ADC module samples the analog voltage at the LDR voltage-divider junction and produces a 10-bit digital value ranging from 0 (minimum voltage / darkest) to 1023 (maximum voltage / brightest).
4.	Percentage Conversion: The raw ADC value is scaled to a 0-100% range using the formula: percent = (ADC value × 100) / 1023.
5.	Display Update: The calculated percentage is formatted as a string and written to the second row of the LCD, while the first row continues to display a static label.
6.	Threshold Comparison: The percentage is compared against two thresholds: below 20% is classified as Low (dark), between 20% and 60% as Medium, and above 60% as High (bright).
7.	Output Actuation: Based on the classification, the corresponding LED is switched on (and the other two switched off), and the buzzer is activated only in the Low (dark) condition to alert the user.
## Threshold Logic Summary

<img width="624" height="120" alt="image" src="https://github.com/user-attachments/assets/fbb83e64-016c-4396-be3b-a93924f0d77e" />

## 11. Conclusion and Future Work
## Conclusion
This project successfully demonstrates a complete light intensity monitoring system built around the PIC18F452 microcontroller. By interfacing an LDR sensor through the microcontroller's ADC module, the system accurately measures ambient light levels and converts them into an easily interpretable percentage value. The combination of an LCD numeric display, a three-level LED indicator, and a buzzer alarm provides a comprehensive, multi-modal output that is both informative and immediately actionable. The project reinforces key embedded systems concepts including analog signal conditioning, ADC configuration and usage, LCD interfacing in 4-bit mode, and threshold-based digital output control.
## Future Work
•	Add wireless connectivity (e.g., Bluetooth or Wi-Fi module) to remotely monitor light intensity data.
•	Log historical light intensity data to an SD card or EEPROM for trend analysis.
•	Add automatic relay control to switch lighting fixtures on/off based on intensity.
•	Implement user-adjustable thresholds via push buttons and a menu system on the LCD.
•	Replace the LDR with a digital light sensor (e.g., BH1750) for improved accuracy and linearity.







