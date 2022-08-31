# Servo Motor 3D Pointer 

## Introduction

This program uses 2 servo motors controlled by PWM signals done by TimerB clock.            
2 PWM signals simultaneously - phi (TBCCR1) & theta (TBCCR2), with frequency (TBCCR0).        
For more hardware info about dutycycle-angle ratio check the final report.

## HW

TI MSP430FG4619

## Functionality

1. Instruction - show on LCD general info about the project.              
2. 3D pointer (phi,theta) - servo stick will move to spatial position according to phi and 
    theta inputs. Sampling of angles is by potentiometer (0v-3v) done by ADC12 clock.          
    In this option PWM frequency is 40 Hz.                                                    
3. 3D pointer (x,y,z) -  servo stick will move to spatial position according to elementary    
    vectors. Inputs are via keypad. In this function there are 7 inputs possible:             
    (0,0,1) => (phi 0,theta 90) , (0,1,0) => (phi 90,theta 0) ,           
    (0,1,1) => (phi 90,theta 45) , (1,0,0) => (phi 0,theta 0) ,                                                                                       
    (1,0,1) => (phi 0,theta 45) , (1,1,0) => (phi 45,theta 0) ,                             
    (1,1,1) => (phi 45,theta 45)                                                            
    In this option PWM frequency is 40 Hz.                                                      
4. Scanner mode - (input) PWM frequency (5Hz - 20Hz), fixed angle and its value.             
    servo stick will move to the chosen fixed angle and in the other angle will scan area   
    according to it spatial range (from min to max) repeatedly with 100msec delay.          
    Sampling of angle and frequency is by potentiometer (0v-3v) done by ADC12 clock.        
5. Quit - go to sleep and wait for wake from push button PB0.                              

## Size

2.83 kb                                                                                  
