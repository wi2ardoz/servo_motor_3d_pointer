;#============================================================================================================#;
;# Title:         Servo Motor 3D Pointer (ver2) - MSP430xG461x                                                #;
;# Date:          21.01.20                                                                                    #;
;# Author:        Liad Oz                                                                                     #;
;#                Gal Vaknin                                                                                  #;
;#                                                                                                            #;
;# Description:   This program uses 2 servo motors controlled by PWM signals done by TimerB clock.            #;
;#                2 PWM signals simultaneously - phi (TBCCR1) & theta (TBCCR2), with frequency (TBCCR0).      #;  
;#                For more hardware info about dutycycle-angle relation check the final report.               #;
;#                Main menu:                                                                                  #;
;#                                                                                                            #;
;#                (1) Instruction - show on LCD general info about the project.                               #;
;#                (2) 3D pointer (phi,theta) - servo stick will move to spatial position according to phi and #;
;#                    theta inputs. Sampling of angles is by potentiometer (0v-3v) done by ADC12 clock.       #;   
;#                    In this option PWM frequency is 40 Hz.                                                  #;  
;#                (3) 3D pointer (x,y,z) -  servo stick will move to spatial position according to elementary #;   
;#                    vectors. Inputs are via keypad. In this function there are 7 inputs possible:           #;  
;#                    (0,0,1) => (phi 0,theta 90) , (0,1,0) => (phi 90,theta 0) ,                             #;                                                            
;#                    (0,1,1) => (phi 90,theta 45) , (1,0,0) => (phi 0,theta 0) ,                             #;                                                           
;#                    (1,0,1) => (phi 0,theta 45) , (1,1,0) => (phi 45,theta 0) ,                             #;
;#                    (1,1,1) => (phi 45,theta 45)                                                            #;
;#                    In this option PWM frequency is 40 Hz.                                                  #;    
;#                (4) Scanner mode - (input) PWM frequency (5Hz - 20Hz), fixed angle and its value.           #;  
;#                    servo stick will move to the chosen fixed angle and in the other angle will scan area   #;
;#                    according to it spatial range (from min to max) repeatedly with 100msec delay.          #;
;#                    Sampling of angle and frequency is by potentiometer (0v-3v) done by ADC12 clock.        #;
;#                (5) Quit - go to sleep and wait for wake from push button PB0.                              #;
;#                                                                                                            #;
;# Size:           2.83 kb                                                                                    #;
;#============================================================================================================#;  
  
#include  <msp430xG46x.h>
;=============================================================================================================
          ; macro definitions 
          
          ; delay
delay     MACRO   time                                                                                                               
          local   L3      
          mov.w   time,R15               
L3        dec.w   R15                     
          jnz     L3                                          
          ENDM
          
          ; multiplier macro
mult      MACRO   op1,op2,res                                                   ; op1,op2 16 bit and res is 32 bit
          mov.w   op1,&MPY                                                      ; load first operand-unsigned mult
          mov.w   op2,&OP2                                                      ; load second operand 
          mov.w   &RESLO,res
          mov.w   #0x02,R13
          mov.w   &RESHI,res(R13)
          ENDM
          
          ; divider macro
div       MACRO   Divided,Divisor,Quotient,Remainder
          LOCAL   L1,L2,L3
          mov     #17,R9
          clr     R10                                                           ; quotient register
          mov.w   Divided,R14                                                   ; devided registers is R13,R14
          clr     R13
          mov.w   Divisor,R11                                                   ; divisor registers is R11          
L3        rla     R10
L1        dec     R9
          jz      L2
          rla     R14
          rlc     R13
          cmp     R11,R13
          jlo     L3
          sub     R11,R13
          setc
          rlc     R10
          jmp     L1           
L2        mov.w   R10,Quotient
          mov.w   R13,Remainder             
          ENDM 
;=============================================================================================================         
          NAME    Main
          PUBLIC  Main
          EXTERN  Print,Config                                                  ; extern routines         
          EXTERN  PORT1_ISR,PORT2_ISR,ADC12_ISR,TIMERB_ISR                      ; extern interrupts
          PUBLIC  PullUp,PullDown,PB_opt,KP_opt,flagPB0,potmode                 ; var & flags                 
          PUBLIC  temp,sum,quotient,remainder,a,b,c                             ; calculation vars           
          PUBLIC  phi,theta,angle                                               ; option 2 vars
          PUBLIC  xyz,flagK,flagxyz,str3_0,str3_1                               ; option 3 vars
          PUBLIC  freq,flagFixAngle,angle4,modeUpDown                           ; option 4 vars   
;============================================================================================================= 
          ORG     0x1100                                                        ; Begins a RAM segment
;=============================================================================================================
              ; variables  
PullUp        EQU  280                                                          ; wait of 0.8msec = (3op*280)/2^20
PullDown      EQU  70                                                           ; wait of 0.2msec = (3op*70)/2^20
PB_opt        DW   1                                                            ; PB-check which place in menu
KP_opt        DW   0                                                            ; which option has been chosen from the menu with keypad
angle         DW   "000"
a             DW   0          
b             DW   0
c             DW   0
divided       DW   0
divisor       DW   0
quotient      DW   0
remainder     DW   0
temp          DW   0 
sum           DL   0
flagPB0       DW   0
phi           DW   0
theta         DW   0
angle4        DW   0
flagK         DW   0
xyz           DW   0
flagxyz       DW   0
potmode       DW   0
freq          DW   0
flagFixAngle  DW   0
modeUpDown    DW   1

              ; strings
str1          DB   "1. Instructions"                                            ; menu options
str2          DB   "2. 3D pointer      (phi,theta)"                             
str3          DB   "3. 3D pointer      (x,y,z)"                                 
str4          DB   "4. Scanner"                                                 
str5          DB   "5. Quit"                                                    
str11         DB   "Welcome to 3D   Pointer"                                    ; menu1 options
str12         DB   "Based Servo     Motor"                                      
str13         DB   "By Liad and Gal      "                                      
str14         DB   "Connect POT1 to P6.3 "                                      
str15         DB   "PB0 To EXIT"                                                 
str32         DB   "Insert 3 binary digits"                                     ; menu3 options
str3_0        DB   "0"
str3_1        DB   "1"
str41         DB   "Choose angle    phi-1 / theta-2"                            ; menu4 options
str51         DB   "Going to sleep  PB0 to wake-up"                             ; menu5 options

;=============================================================================================================
             RSEG     CSTACK                                                    ; define stack segment
;=============================================================================================================
             RSEG     CODE                                                      ; assemble to Flash memory
;=============================================================================================================                                                                                  
Main         mov.w   #SFE(CSTACK),SP                                            ; initialize stack pointer
StopWDT      mov.w   #WDTPW+WDTHOLD,&WDTCTL                                     ; stoping watch dog      

SetupFLL     bis.b   #XCAP14PF,&FLL_CTL0                                        ; configure load caps
OFIFGcheck   bic.b   #OFIFG,&IFG1                                               ; clear OFIFG
             mov.w   #047FFh,R15                                                ; wait for OFIFG to set again if
OFIFGwait    dec.w   R15                                                        ; not stable yet
             jnz     OFIFGwait
             bit.b   #OFIFG,&IFG1                                               ; has it set again?
             jnz     OFIFGcheck                                                 ; if so, wait some more
             		 
             call    #Config                                                    ; config ports and periphrials
             mov     #0,KP_opt                                                  ; keypad state
             mov     #1,PB_opt                                                  ; pull button state
                      
;-------------------------------------------------------------------------------------------------------------
             ; main code section section
             jmp     start                                                      
       
sleep        bis.w   #LPM0+GIE,SR                                               ; Enter LPM0, Global interrupts enabled
             jmp     start
              
             ; memory issue for div macro
divLabel     div    divided,divisor,quotient,remainder                          
             ret
             
             ; FSM_loop
start        cmp     #1,KP_opt                                                  ; check option number 1 was pressed    
             jz      Instruct                     
             
             cmp     #2,KP_opt                                                  ; check option number 2 was pressed    
             jz      PointerAngle

             cmp     #3,KP_opt                                                  ; check option number 3 was pressed    
             jz      PointerSpace
             
             cmp     #4,KP_opt                                                  ; check option number 4 was pressed    
             jz      Scanner
              
             cmp     #5,KP_opt                                                  ; check option number 5 was pressed    
             jz      quitLabel         
;-------------------------------------------------------------------------------------------------------------                                                                               
             ; main menu - KP_opt=0
             cmp     #0,KP_opt                                                   
             jnz     Instruct
             
First_Menu   cmp     #1,PB_opt                 			                ; check if PB_opt=1 - fisrt line of menu           
             jnz     Second_Menu               			                ; if not jump to second line of menu
             mov     #str1,R6                    		                ; str1 address to R6 
             call    #Print                      		                ; Print function
             jmp     sleep                       		                ; jump to sleep

Second_Menu  cmp     #2,PB_opt                   		                ; check if PB_opt=2 - second line of menu 
             jnz     Third_Menu                  		                ; if not jump to third line of menu
             mov     #str2,R6                    		                ; str2 address to R6 
             call    #Print                     		                ; print function
             jmp     sleep                      		                ; jump to sleep
            
Third_Menu   cmp     #3,PB_opt                		                        ; check if PB_opt=3 - third line of menu 
             jnz     Fourth_Menu               			                ; if not jump to fourth line of menu
             mov     #str3,R6                  			                ; str3 address to R6 
             call    #Print                    			                ; print function
             jmp     sleep                    		                        ; jump sleep
             
Fourth_Menu  cmp     #4,PB_opt                		                        ; check if PB_opt=4 - third line of menu 
             jnz     Fifth_Menu               			                ; if not jump to fourth line of menu
             mov     #str4,R6                  			                ; str4 address to R6 
             call    #Print                    			                ; print function
             jmp     sleep                    		                        ; jump sleep

Fifth_Menu   mov     #str5,R6                                                   ; str5 address to R6 
             call    #Print                                                     ; print function
             jmp     sleep                                                      ; jump to sleep

;-------------------------------------------------------------------------------------------------------------  
             ; KEY_PAD OPTIONS 
             ; "instructions" KP_opt=1                                                 
Instruct    
First_Inst   cmp     #1,PB_opt			                                ; checks if PB_opt=1           
             jnz     Second_Inst                		                ; if not jump -> Second_Inst - second line of instructions
             mov     #str11,R6                 		                        ; str11 address to R6 
             call    #Print                      		                ; print function
             jmp     sleep                       		                ; jump to sleep

Second_Inst  cmp     #2,PB_opt                   		                ; check if PB_opt=2
             jnz     Third_Inst                  		                ; if not jump -> Third_Inst - third line of instructions
             mov     #str12,R6                 			                ; str12 address to R6 
             call    #Print                      		                ; print function
             jmp     sleep                       		                ; jump to sleep
            
Third_Inst   cmp     #3,PB_opt                   		                ; check if PB_opt=3
             jnz     Fourth_Inst            		                        ; if not jump -> Fourth_Inst - fourth line of instructions
             mov     #str13,R6                 			                ; str13 address to R6 
             call    #Print                      		                ; print function
             jmp     sleep                       		                ; jump to sleep
             
Fourth_Inst  cmp     #4,PB_opt                   		                ; check if PB_opt=4
             jnz     Fifth_Inst            		                        ; if not jump -> Fourth_Inst - fourth line of instructions
             mov     #str14,R6                 			                ; str14 address to R6 
             call    #Print                      		                ; print function
             jmp     sleep                       		                ; jump to sleep             
             
Fifth_Inst   mov     #str15,R6                 	                                ; str15 address to R6 
             call    #Print                      		                ; print function
             jmp     sleep                       		                ; jump to sleep
         
;-------------------------------------------------------------------------------------------------------------             
             ; "PointerAngle" KP_opt=2
PointerAngle cmp     #2,KP_opt                		                        ; checks KP_opt=2 -> option number 3 was pressed  
             jnz     PointerSpace                                               ; if not, jump to sconds meter
             
             delay   #3000
             mov     #2,flagPB0                                                 ; set flagPB0 to count 2 inputs
             mov     #0,potmode                                                 ; set potmode to angle mode
             delay   #3000

             ; buttons interrupts masking
             bic.b   #0xFF,&P1IE                                                ; enable only interrupts fromPB0
             bis.b   #0x10,&P1IE
             bic.b   #2,&P2IE                                                   ; irq interrupt disable
             
             ;enable ADC12 moudle
             bic     #ENC,&ADC12CTL0
             bis     #ADC12ON,&ADC12CTL0
             bis     #ENC,&ADC12CTL0
             
             ; turn on sampling
             bis.w   #ADC12SC,&ADC12CTL0                                        ; sample pot1 to understand how many degrees to move          
             
             ; (1,2 inputs) wait for phi and theta (acts as barrier, not full delay!)
delayAgain   delay   #14000                           
             cmp     #0,flagPB0                                                 ; if flagPB0 is 0 => end of inputs, continue
             jnz     delayAgain                                                 ; else flagPB0 is 1 => continue smapling + delay barrier
 
             ; start Timer_B
             mov     #TBSSEL_1+MC_1,&TBCTL                                      ; set ACLk 2^15, up mode
             mov     #819,&TBCCR0                                               ; set 40hz
             
             ; calculate duty of phi and update TBCCR1                 
             mov     #OUTMOD_7,&TBCCTL1                                         ; Reset/Set mode
             mult    phi,#61,sum                                                 
             mov     sum,divided
             mov     #180,divisor
             call    #divLabel
             add     #18,quotient
             mov     quotient,&TBCCR1                                           ; update PWM for phi
             
             ; calculate duty of theta and update TBCCR2  
             mov     #OUTMOD_7,&TBCCTL2                                         ; Reset/Set mode
             mult    theta,#5,sum
             mov     sum,divided
             mov     #18,divisor
             call    #divLabel
             add     #49,quotient
             mov     quotient,&TBCCR2                                           ; update PWM for theta
              
             ; print in LCD PB0 to EXIT
             mov     #str15,R6                  	                        ; str15 address to R6 
             call    #Print                    		                        ; Print function
             
             ; wait for servo
             delay   #3000

             ; go to sleep wait for intturapt from PB0
             ; when PB0 is on turn on timer create PWM for angles 0             
             jmp     sleep                    		    
;-------------------------------------------------------------------------------------------------------------              
               ; "Pointer Space" KP_opt=3 
PointerSpace   cmp     #3,KP_opt                                                ; checks KP_opt=3 -> option number 3 was pressed       
               jnz     Scanner                                                  ; if not, jump to Scanner
               
               ; enable intreupt from keypad (input)
               clr.b   &P10OUT
               bis.b   #0x02,&P2IE
               
               ; check if first time and all inputs ready
               ; (1,2,3 inputs) wait for vector
               cmp     #0,flagK                                                 ; if flag is 0 => update flags, print, sleep 
               jnz     inputMode                                                ; else flag is 1 => already in input mode
               clr     xyz                                                      ; clean xyz 3 binary string
               mov     #1,flagK                                                 ; set flagK for input mode
               mov     #3,flagxyz                                               ; set flagxyz for 3 inputs
               mov     #str32,R6                                                ; print to LCD
               call    #Print 
               jmp     sleep
inputMode      cmp     #0,flagxyz                                               ; if flag is 0 => inputs ready, begin option 3
               jz      begin
               jmp     sleep                                                    ; else wait for another input, sleep
                         
begin          
               ; disable intreupt from keypad (input) 
               bic.b   #0x02,&P2IE
               
               rra     xyz                                                      ; fix the final vector xyz                                                    
                             
               ; check which xyz vector is it (respectively) and jump to update
               ; (0,0,1),(0,1,0),(0,1,1),(1,0,0),(1,0,1),(1,1,0),(1,1,1) 
opt1           cmp     #1,xyz
               jnz     opt2
               mov     #0,phi
               mov     #90,theta
               jmp     update
opt2           cmp     #2,xyz
               jnz     opt3
               mov     #90,phi
               mov     #0,theta
               jmp     update
opt3           cmp     #3,xyz
               jnz     opt4 
               mov     #90,phi
               mov     #45,theta
               jmp     update
opt4           cmp     #4,xyz
               jnz     opt5
               mov     #0,phi
               mov     #0,theta
               jmp     update
opt5           cmp     #5,xyz
               jnz     opt6
               mov     #0,phi
               mov     #45,theta
               jmp     update
opt6           cmp     #6,xyz
               jnz     opt7
               mov     #45,phi
               mov     #0,theta
               jmp     update
opt7           cmp     #7,xyz
               mov     #45,phi
               mov     #45,theta
               
update         ; start Timer_B
               mov     #TBSSEL_1+MC_1,&TBCTL                                    ; set ACLk 2^15, up mode
               mov     #819,&TBCCR0                                             ; set 40hz
               
               ; calculate duty of phi and update TBCCR1                         
               mov     #OUTMOD_7,&TBCCTL1                                       ; reset/set mode
               mult    phi,#61,sum
               mov     sum,divided
               mov     #180,divisor
               call    #divLabel
               add     #18,quotient
               mov     quotient,&TBCCR1
               
               ; calculate duty of phi and update TBCCR2 
               mov     #OUTMOD_7,&TBCCTL2             
               mult    theta,#5,sum        
               mov     sum,divided
               mov     #18,divisor
               call    #divLabel
               add     #49,quotient
               mov     quotient,&TBCCR2
               
               ; print in LCD PB0 to EXIT
               mov     #str15,R6                  	                        ; str15 address to R6 
               call    #Print                                                   ; Print function 
               
               ; wait for servo
               delay   #50000
               delay   #50000
               delay   #50000
               
               jmp     sleep


;------------------------------------------------------------------------------------------------------------- 
              ; memory issue for jmp labels
sleepLabel    jmp      sleep
instructLable jmp      Instruct
quitLabel     jmp      Quit_prog
;-------------------------------------------------------------------------------------------------------------

;-------------------------------------------------------------------------------------------------------------
             ; "Scanner" KP_opt=4
Scanner      cmp     #4,KP_opt                                                  ; checks KP_opt=4 -> option number 4 was pressed                       
             jnz     Quit_prog                                                  ; if not, jump to quit            
                         
             delay   #3000                                                      
             mov     #2,flagPB0                                                 ; set flagPB0 to count 2 inputs
             mov     #1,potmode                                                 ; set potmode to freq mode
             delay   #3000
             
             ; buttons interrupts masking
             bic.b   #0xFF,&P1IE                                                ; enable only interrupts fromPB0
             bis.b   #0x10,&P1IE
             bic.b   #2,&P2IE                                                   ; irq interrupt disable
             
             ; enable ADC12 moudle
             bic     #ENC,&ADC12CTL0                                            ; set edit mode on
             bis     #ADC12ON,&ADC12CTL0                                        ; turn on ADC12 module
             bis     #ENC,&ADC12CTL0                                            ; set edit mode off
             
             ; start sampling
             bis.w   #ADC12SC,&ADC12CTL0                                        ; sample pot1 to understand how many degrees to move    

             ; (1) wait for freq input (acts as barrier, not full delay!)
delayAgain2  delay   #14000                 
             cmp   #1,flagPB0                                                   ; if flagPB0 is 1 => end of first pot input, continue 
             jnz   delayAgain2                                                  ; else flagPB0 is 2 => continue smapling + delay barrier
             
             ; keypad config            
             mov     #1,flagK                                                   ; set keypad for input mode  
             clr.b   &P10OUT                                                    ; set keypad for interrupts
             bis.b   #0x02,&P2IE
             
             ; choose fixed angle
             ; flagfixangle - phi 0 , theta 1 (keypad press - phi 1 , theta 2)
             mov     #str41,R6
             call    #Print  
             delay   #3000 
             
             ; (2) wait for freq input - sleep here
             bis.w   #LPM0+GIE,SR                                               ; enter sleep wait for angle option     

             bic.b   #0x02,&P2IE                                                ; disable intreupt from keypad (input)
             mov     #0,potmode                                                 ; set potmode to angle mode
                           
             ; enable ADC12 moudle  
             bic     #ENC,&ADC12CTL0                                            ; set edit mode on
             bis     #ADC12ON,&ADC12CTL0                                        ; turn on ADC12 module
             bis     #ENC,&ADC12CTL0                                            ; set edit mode off
             bis.w   #ADC12SC,&ADC12CTL0                                        ; sample pot1 to understand how many degrees to move    
             
             ; (3) wait for freq input (acts as barrier, not full delay!)         
delayAgain3  delay   #14000
             cmp   #0,flagPB0                                                   ; if flagPB0 is 0 => end of second pot input, continue 
             jnz   delayAgain3                                                  ; else flagPB0 is 1 => continue smapling + delay barrier
                        
             ; calculate freq and update TBCCR0 (y= 2^15/x)
             mov     #32768,divided
             mov     freq,divisor
             call    #divLabel
             mov     quotient,&TBCCR0
                      
             ; check which angle is fixed
             cmp   #0,flagFixAngle
             jz    phiFixed
             
             ; timer B fixed angles config        
thetaFixed   ; calculate duty of fixed theta and update TBCCR2, update non-fixed angle TBCCR1
             mult    angle4,#5,sum        
             mov     sum,divided
             mov     #18,divisor
             call    #divLabel
             add     #49,quotient
             mov     quotient,&TBCCR2
             mov     #4,&TBCCR1                                                 ; set 0 angle to phi
             mov     #4,angle4                                                  ; update phi index (ready for intturpts)
             jmp     continue                                                   
phiFixed     ; calculate duty of fixed phi, update TBCCR1, update non-fixed angle TBCCR2
             mult    angle4,#61,sum        
             mov     sum,divided
             mov     #180,divisor
             call    #divLabel
             add     #18,quotient
             mov     quotient,&TBCCR1
             mov     #49,&TBCCR2                                                ; set 0 angle to theta
             mov     #49,angle4                                                 ; update theta index (ready for intturpts)
             
             ; TimerB config 
continue     mov     &ADC12MEM3,R12                                             ; clean ADC12 IFG          
             mov     #OUTMOD_7,&TBCCTL1                                         ; reset/set mode
             mov     #OUTMOD_7,&TBCCTL2                                         ; reset/set mode
             mov     #CCIE,&TBCCTL3                                             ; enable IE from TBCCTL3  
             mov     #1280,&TBCCR3                                              ; set 100msec
             mov     #TBSSEL_1+MC_1+TBIE,&TBCTL                                 ; set ACLk 2^15, up mode, enable clock intturpts
             
             ; wait untill servo moves to fixed angle 
             delay   #50000                         
             delay   #50000
             delay   #50000
             
             ; print in LCD PB0 to EXIT
             mov     #str15,R6                  	                        ; str15 address to R6 
             call    #Print                    		                        ; print function 

             delay   #50000
             
             jmp    sleepLabel
;-------------------------------------------------------------------------------------------------------------  
             ; "quit" KP_opt=5 
Quit_prog    cmp     #5,KP_opt                                                  ; checks KP_opt=5 -> option number 5 was pressed       
             jnz     instructLable                                              ; if not, jump to Intruct
             mov     #str51,R6                 	                                ; program finish message
             call    #Print                                                     ; Print function
             bic.b   #0xFF,&P1IE                                                ; enable interrupt only fromPB0
             bis.b   #0x10,&P1IE
             bic.b   #2,&P2IE                                                   ; irq interrupt disable
             jmp     sleepLabel                                                 ; jump to sleep 
;-------------------------------------------------------------------------------------------------------------              
             nop                                                                ;remove warnnigs
             
;=============================================================================================================
            COMMON  INTVEC                                                      ; Interrupt Vectors- common segment with name of INTVEC 
;=============================================================================================================
            ORG     RESET_VECTOR                                                ; MSP430 RESET Vector
            DW      Main          
            ORG     PORT1_VECTOR                                                ; PORT1 Interrupt Vector
            DW      PORT1_ISR
            ORG     PORT2_VECTOR                                                ; PORT2 Interrupt Vector
            DW      PORT2_ISR
            ORG     TIMERB1_VECTOR                                              ; TIMER_B Interrupt Vector
            DW      TIMERB_ISR
            ORG     ADC12_VECTOR                                                ; ADC12 Interrupt Vector
            DW      ADC12_ISR
         
            END
