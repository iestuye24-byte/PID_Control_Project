**PID Control Project**



1\. Objectives



The goal of this laboratory is to get familiar with basic PID control and to gain experience

in implementing a control algorithm using MATLAB App Designer. The activity also aims to

connect what is happening in code with what is happening in an actual physical system.



2\. Introduction



Control is based on the idea that we have a desired value, called the setpoint, and an

actual value that is measured from the system. The objective is to make the actual value

follow the setpoint in a way that is stable and acceptable. In real systems, disturbances will

always be present, so the controller must be able to compensate for them. If there is no.

feedback, the system is simply driven based on expectation. This is called feedforward

control. It may work if conditions do not change, but once disturbances occur, the system

will deviate and there is no correction. This is not acceptable for most engineering

applications. With feedback, the actual value is measured and compared to the setpoint.

The difference is the error. The controller uses this error to determine how to adjust the

system input.



A proportional controller responds directly to the error. When the error is large, the

control action is large. When the error becomes small, the control action reduces.

However, proportional control alone cannot eliminate steady-state error. Increasing the

proportional gain reduces the error but can lead to instability. To solve this, an integral

term is added. The integral action accumulates error over time. Because of this

accumulation, the controller continues to act until the error is driven to zero. The

drawback is that if the error persists for a long time, the accumulated value becomes

large, leading to overshoot. This is called integral windup.



A derivative term can also be added. This term looks at how fast the error is changing and

helps predict future behavior: The combination of proportional, integral, and derivative

terms forms the PID controller.



3\. Building the Hardware



The system uses an Arduino Uno R3, an LED, and a Light Dependent Resistor (LDR). The

LED is used as the actuator, and the LDR is used as the sensor.



The LED is driven using a Pulse Width Modulation (PWM) signal from the Arduino. In this

setup, the PWM output is connected to digital pin D3 (since this pin supports PWM on the

Uno). Make sure to connect the anode of the LED (+ side, long lead) to the PWM output

port D3, and the cathode (- side, short lead, also the LED has a flat spot on the cathode

side) to the series resistor. If you reverse the cathode and the anode, the LED will not light

up, but it will not cause any damage. \[WARNING] The LED must be connected in series

with a resistor of about 220 ohms to limit the current. If the resistor is not used, the LED may

be damaged.



The LDR is used to detect light intensity. Its resistance changes depending on how much

light it receives. Since resistance cannot be measured directly by the Arduino, the LDR is

placed in a voltage divider with a fixed resistor of 5.1 ki. The voltage at the midpoint of

this divider is measured using analog input 0 (A0). In the app, we will call this the Actual

Value. In the program, we will also calculate the instantaneous value of the LDR (it

changes, because the light intensity from the LED projected onto it changes), so we need a

formula for this. Let's call the series resistor Rs the resistance of the LDR R;, the

measured actual voltage V and the power supply voltage from the Arduino VCC.



The relationship between the measured voltage and the resistance of the LDR is given by:



RL = Rs ( V / VCC - V )



where (RL) is the resistance of the LDR, (Rs) is the series resistor (5.1 k-ohms), (V) is the

measured voltage at A0, and (Vcc) is the supply voltage, which is 5V.



The circuit behaves in an inverted manner. When the LED becomes brighter, the LDR

resistance decreases, which changes the voltage reading in the opposite direction. This

inversion must be considered in the control equation.



You may refer to the following figures (Figure 1 \& 2) for the hardware setup. When operating the board,

make sure you place some folded tissue between the LED and the LDR.

\*\*Figure 1

\*\*Figure 2



Question 1: Derive the calculation of the instantaneous resistance value of the LDR based on the schematic drawn in Figure 2.



4\. Building the Software



Before running the program, MATLAB must be installed along with the MATLAB Support

Package for Arduino Hardware. MATLAB communicates with the Arduino by uploading a

server program that allows commands such as reading voltages and writing PWM outputs.

In this setup, write PWM Voltage will be used to generate PWM signal with specified voltage

on the digital pin..



4.1 Graphical User Interface



The graphical user interface is created using MATLAB App Designer. The interface

includes gauges and numeric indicators for the actual value, error, integrated error,

derivative error, and output voltage. These values are all expressed in volts. Figure 3

shows an example of the program's GUI All units are in Volt (V), except for the Actual

Value gage, you can set the limits of the gages to what is shown in Figure 3. The limits of

the Actual Value gages are calculated and assigned at the start of the program by turning

the LED off completely, which gives the upper limit, and then setting it to +5V (100%

PWM), which gives the lower limit.



\*\*Figure 3. Example Graphical User Interface of the PID controller app



At the bottom right of the GUI, you see a gage named R,;, which shows the instantaneous

value of the Light Dependent Resistor. The value of this will be calculated using an

equation derived from the voltage divider (see Question 1 answer).



The setpoint can be adjusted using either a slider or a numeric input field. These two

inputs must remain synchronized so that changing one updates the other. The limits of the

setpoint slider and the setpoint Edit Field (numeric) component are identical to the limits

of the Actual Value gage that are set at the start of the program. When the user moves the

slider to a new setpoint, this number needs to be copied to the setpoint Edit Field and vice

versa.



Sliders are also used to adjust the controller gains (Kp), (Ki), and (Kd). These values directly

affect how the system responds. You can keep the limits as shown in the figure

(recommended), these are chosen to allow demonstration of instability.



A spinner is used to define the number of samples used when averaging sensor readings.

Set the limits to \[10,30] and the step to 10. A STOP button is included to terminate the

control loop. Figure 4 shows the Component Browser, where names are chosen for the

objects to be used for the project. You may use EasyEDA to make the schematic diagram of

the project setup and name the components as shown in Figure 2.



\*\*Figure 4. Names to be used for the components in the PID controller design



4.2 Properties



The program defines properties that are accessible throughout the application. These

include the Arduino object and a flag used to stop the control loop.



In this program, we only need the Arduino component a (technically Matlab calls this an

object, which adds to the confusion), and the StopFlag property. Note that the lines

properties (Access = private) and end are generated automatically when you add

properties.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



properties (Access = private)

a = arduino("COM6', Uno"); ###verify on your device the port being used###

StopFlag =0

end



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



4.3 Startup Function



The main control logic is implemented in the startup function (startupFcn). You may do

this callback by right clicking on the app itself and scroll down to "add callback’. The

program starts by clearing previous outputs and initializing variables. Limits are defined

for the integral term to prevent windup. System constants such as the series resistance

and supply voltage are also defined.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



close all

cle



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



At the top of any program you should always make an initialization section, although some

initialization can take place in the Properties section as well. Shown here is the

initialization section of the app, we set limits for the integrated error (IntErrorMax and

IntErrorMin) as well as the value of the series resistor of the voltage divider and the

power supply value of the Arduino.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



================= Initialization ===============



IntErrorMax =15;

IntErrorMin =-15;

R Series = 5100; % Series resistance

Vee =5;

DataVectorSizeMax = 10000; % Maximum nr of data points collected



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



Following the initialization, there is a series of step that need to be taken. These are given

in natural language, leave the commented line, and below insert your Matlab code that

accomplishes these tasks.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



1 % Get NrSamples from the value of the NrSamples spinner value

2 % Make zero filled container for AverageVoltage size (NrSamples,1)



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



Next, we need to measure the lower controllable setpoint limit SetPointMin; we do that by

turning the LED On while writing 5V to port D3 using the writtPWMVoltage function.

Then, to reduce noise, we collect a number (NrSamples) of voltage samples using a for

loop using the readVoltage function. After we drop out of the loop, we assign SetPointMin

to the average of the AverageVoltage vector (which now contains NrSamples samples). Do

the same for the upper controllable setpoint limit SetPointMax by writing OV to port D3.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



1 % Turn LED fully ON by writing a PWM voltage of 5 volt to port D3; ...

&#x09;Collect NrSample datapoints (see above);

&#x09;Assign SetPointMin to the ... mean of AverageVoltage

2 % Turn LED fully OFF by writing a PWM voltage of 0 volt to port D3; ...

&#x09;Collect NrSample datapoints (see above);

&#x09;Assign SetPointMax to the ... mean of AverageVoltage



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



We can now use the (rounded) SetPointMin and SetPointMax values to set the lower and

upper limit of the ActualValueGauge as well as the SetPointSlider.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



1 % Set ActualValueGauge limits to \[round(SetPointMin,1); ...

&#x09;round(SetPointMax,1)]. This rounds off the Min and Max setpoint to ..

&#x09;one decimal number.

2 % Same for the SetPointSlider limits



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



At the start of the program let's set the SetPoint to the mean of the SetPointMin and

SetPointMax, this is at the 12 o'clock position of the Actual Value gage. We'll do the same

thing for the SetPointSlider.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



1 % Set SetPointEditField value to (round(SetPointMin,1) + ...

&#x09;round(SetPointMax,1))/2. This sets the initial SetPoint to the ...

&#x09;middle of the SetPoint Limits (12 o'clock position on you setpoint gage)

2 % Same for the SetPointSlider value



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



Before we write the control loop itself, we need to create some containers (vectors) to

collect data over time so later we can make a graph and see how the app performs as

follows:



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



1% ActualValueVec = zeros(DataVectorSizeMax,1);

2 % Do the same for SetPointVec, ErrorVec, ErrorintVec, ErrorDifVec, ...



OutputVoltageVec

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



1 % Initialize Errorint and ErrorOld to 0. The latter is a variable that...

&#x09;we need to calculate the derivative error ErrorDif, you'll see later.

2 % nis an index that is incremented every time the control loop ...

&#x09;executes, initialize to 1



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



Now it is time to write the control loop itself. Make a while loop that is executed until the

user clicks the STOP button, the latter which executes the STOPButton callback, which in

turn, sets the StopFlag property (remember that properties are written as app.variable

name) to 1.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

1

2 % ================= Control loop ===============

3 while app StopFlag == 0;

4

5 % Assign SetPoint to the SetPointSlider value

6 % Set the value of SetPointEditField to SetPoint

7 % Record SetPoint in the vector SetPointVec(n), where n is an index

8

9 % Collect NrSample datapoints as we did when we measured the ..

SetPointMax value (see above);

Assign ActualValue to the mean of ... the AverageVoltage vector

10 % Set the value of the ActualValueGauge to ActualValue

11 % Set the value of the ActualValueEditField to ActualValue

12 % Record ActualValue in the vector ActualValueVec(n), where n is an ... index

13

14 % Calculate the Error as the difference between SetPoint and ...

ActualValue. Note that Error can be positive and negative!

15 % Set the value of the ErrorGauge to Error

16 % Set the value of the ErrorEditField to Error

17 % Record Error in the vector ErrorVec(n), where n is an index

18

19 % Calculate the Errorint as the sum of Errorint and Error.

The ... integrated error continuously adds to itself making a running sum.

20

21 % Prevent integral windup by limiting the value that Errorint can ...

attain. Define IntErrorMax to 15 and IntErrorMin to-15 in the initialization section.

22 % If Errorint > IntErrorMax, then Errorint = IntErrorMax

23 % If Errorint < IntErrorMin, then Errorint = IntErrorMin

24

25 % Set the value of the ErrorIntGauge to Errorint

26 % Set the value of the ErrorintEditField to Errorint

27 % Record Errorlnt in the vector ErrorIntVec(n), where n is an index

28

29 % Calculate the ErrorDif as the difference between ErrorOld- ...

Error. The derivative error subtracts the current from the ...

previous error (ErrorOld).

30 % Set the value of the ErrorDifGauge to ErrorDif

31 % Set the value of the ErrorDifEditField to ErrorDif

32 9% Record ErrorDif in the vector ErrorDifVec(n), where n is an index

33

34 9% Assign Error0Old to Error to save it for the following loop execution;

35

36 9% Grab the Kp value from the KpValueSlider value. Repeat for the Ki ...

value and Kd value.

37

38 % Generate an output voltage using the PID rule as follows. The ...

negative sign is there because an increase in the LED intensity ...

causes a decrease in the voltage across the LDR. Code is shown here:

39

40 OutputVoltage =-(Kp\*Error + Ki\*Errorint + Kd\*ErrorDif);

41

42 % Limit the OutputVoltage to \[0, 5] V: if OutputVoltage > 5 then ...

set it to 5; if OutputVoltage <0 then set it to zero.

43

44 % Write OutputVoltage to pin D3 of the Arduino

45

46 % Set the value of the PWMoutputGauge to OutputVoltage

47 9% Set the value of the PWMoutputEditField to OutputVoltage

48 % Record OutputVoltage in the vector OutputVoltageVec(n), where n ...

is an index

49

50 % increment index n

51

52 9% pause for 10 milliseconds, to give the GUI time to check if the ..

user pushed any buttons.

53

54 end

55

56 0p =============== End Control loop ==============

57

58 % We will make 5 plots in portrait mode, the first is the most ...

complicated: We plot the ActualValueVec vector up to the nth entry ...

(whatever n is when you hit STOP). Then we hold the plot, to plot...

the SetPointVec vector to the same nth entry. To make the plot axis ...

values the same as those of the ActualValue gage, we copy the x-axis ...

values from the plot but change the y-axis values to ...

round(SetPointMin,1) round(SetPointMax,1)

59

60 subplot(5,1,1)

61 plot(ActualValueVec(1:n,1))

62 hold

63 ActualValueAxis = axis;

64 axis(\[ActualValueAxis(1) ActualValueAxis(2) round(SetPointMin,1) ...

round(SetPointMax,1)])

65 plot(SetPointVec(1:n,1))

66 title(‘Actual value, V')

67

68 % Now we plot the ErrorVec vector in the second plot

69 subplot(5,1,2)

70 plot(ErrorVec(1:n,1))

71 title("Error, V')

72

73 % Make three more plots where you plot ErrorIntVec, ErrorDifVec, and ...

OutputVoltageVec



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



When this program is run, the final plot (output graph) should resemble the one given in

Figure 5. These plots are used to analyze the performance of the controller and to

understand the effect of different gain settings.



\*\*Figure 5. Output graphs of the PID controller



4.4 STOP button callback



This is executed when the user clicks on the STOP button: we simply set the StopFlag to 1,

which makes the program drop out of the control loop.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

1 function STOPButtonPushed (app, event) |

2 app.StopFlag = 1; |

3 end

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



4.5 SetPoint slider callback



This callback function is executed when the user clicks on the SetPoint slider to seta new

setpoint. Update the SetPointEditField (numeric) value.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

1 function SetPointSliderValueChanged(app, event)

2 % Set SetPointEditField value to SetPointSlider value

3 end

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



4.6 SetPoint Edit Field (numeric) callback



This callback function is executed when the user enters a number in the SetPointEdit Field

(numeric). Update the SetPointSlider value.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

1 function SetPointEditFieldValueChanged(app, event)

2 % Set SetPointSlider value to SetPointEditField value

3 end

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



Question 2: Discuss how is the instability of the system is demonstrated and how it control

external disturbance.



7. Submission



Students are required to submit a video of the working MATLAB App Designer (no need

to show the code) with the report file in Google Classroom submission box.



Include the following in the video:



7.1. Set the Kp value to zero and the Ki value to 1. Demonstrate that the controller

responds to a change in setpoint, this is called tracking; use both the SetPointSlider and

the SetPointEditField to change the setpoint.



7.2. Set the Ki value to zero and the Kp value to 1. Demonstrate that the setpoint is not

reached when you do not have integral action, but you do have proportional action (give

Kp a stable value).



7.3. Demonstrate the working on the integral windup code, by setting Kp, Ki and Kd to

zero, which results in a prolonged sustained error that winds up the integral error. Then

return the Ki value to a reasonable value (0.4) and show that the integral error unwinds

and the setpoint is reached again.



7.4. Demonstrate how the output becomes unstable if you set the values of Kp, Ki and Kd

too high. Observe your gauges and your LED.



7.5. Demonstrate that your program compensates for external disturbances,

cover/uncover the LDR and show that the setpoint is reached in both cases.



The report should describe how the application works, starting from how the sensor

measurement is obtained, how the error is calculated, and how the PID controller

generates the output signal. The explanation should clearly connect the mathematical

formulation with the actual implementation. Answers to questions 1 and 2 mustalso be in

the final report. Section 7.1 to 7.5 must also be discussed in the report by discussing the

output graph generated.

