# Arduino-Punk-Console
Pulse width audio for the Arduino Uno R3
/*******************************************************************\
*   ARDUINO PUNK CONSOLE (Pulse Width Audio)                        *
*             Version:  B.2.3.1                                     *
*          Target:  Arduino Uno R3                                  *
*             Sketch by:  Ryan Liston                               *
*           Date: January 2,2023                                    *
*                                                                   *
*      "Fpot" @ A1 controls note frequency                          *
*       "WPot"  @ A0 comtrols pulse width                           *
*                                                                   *
*   Parts:                                                          *
*       1 Aurduino Uno R3                                           *
*       2 Potentiometers (100k prefered;10k recommended minimum.)   *
*       1  Piezo Spkrer                                             *
*                               ...Wires (as needed)                *
*                                                                   *
\*******************************************************************/

//PROGRAM CONSTANTS

//pin settings

#define WPot A0         //input pulse width control on potrntiometer @ pin A0
#define Fpot A1         //input note frequency control on potrntiometer @ pin A1
#define Mask B00010000  //bitmask for digital pin audio output

//.....................................................................

//note table settings

#define Root 16.35      //root tone in calculation (16.35 = C0)
#define Base 2.0000217  //exponential base for calculating note table
#define Intv 12         //note interval (12 = semitones or 12 note chromaric scale)
#define Keys 120        //how many tones to be produced (120 tones)
#define TRef 10000      //timing refrence for note table (10000= 1 percent of 1000000) for coulculating duty cycle percentage

//....................................................................

#define LCnt 50   //percentage at the low end of the WPot potentiometer
#define HCnt 1    //percentage at the high end of the WPot potentiometer
#define HNot 119  //tone at the high end of the FPot potentiometer
#define LNot 0  //tone at the low end of the FPot potentiometer

//....................................................................

//isr settings

#define CRef 4          //clock refference for isr
#define Scal B00001011  //mode and scaler bits for isr

//====================================================================

//initialize global variables

bool Cycl;                    //duty cycle flag (true/false(high/low))
double Clok;                  //duty cycle clock register
double Note[Keys];            //declare array for the note table
volatile unsigned char Freq;  //input value for use as a pointer to the note table array
volatile unsigned char CyHi;  //Duty cycle high % (0-100)
volatile unsigned char CyLo;  //Duty cycle low % (0-100)

//=====================================================================

//Pulse width interrupt service routine

ISR(TIMER1_COMPA_vect) {  //set up isr

  if (micros() > Clok) {                              //checks to see if micros() is greater than the duty cycle (Clok) timer
    switch (Cycl) {                                   //compares cycl (Duty cycle high/low (true/false))..
      case true:                                      // when Cycl=true (high)
        Clok = micros() + Note[Freq] * double(CyHi);  //sets duty cycle clock for high period
        break;                                        //exit case
      case false:                                     // when Cycl=false (low)
        Clok = micros() + Note[Freq] * double(CyLo);  //sets duty cycle clock for low period
        break;                                        //exit case
    }                                                 //end switch
    PORTB = PORTB ^ Mask;                             //toggle Spkr (pin 12)
    Cycl = !Cycl;                                     //toggle Cyle true/false (high/low)
  }                                                   //end if
}  //end of isr

//=====================================================================

void setup() {  //setup

  //.....................................................................

  //Generate None Table

  for (double x = 0; x < Keys; x++) {                      //loop for Keys
    Note[int(x)] = (TRef / (Root * pow(Base, x / Intv)));  //set tones to note table
  }                                                        //end loop

  //.....................................................................

  //initialize pins
  DDRB |= Mask;  //set port b pin 4 (D12) to output

  //.....................................................................

  //Initialize ISR for Timer 1
  TCCR1A = 0;            // set operation
  TCCR1B |= Scal;        // set mode and pre-scaling
  OCR1A = CRef;          // set compare register clock value
  TIMSK1 = bit(OCIE1A);  // set as interrupt on compare/match

  //.....................................................................


}  //end setup

//=====================================================================

void loop() {                                         //start control loop
  Freq = map(analogRead(Fpot), 0, 1023, LNot, HNot);  //read FPot (analog input @ pin A1 ) for note frequency pointer
  CyHi = map(analogRead(WPot), 0, 1023, LCnt, HCnt);  //read WPot (analog input @ pin A0 ) for pulse width high ratio %
  CyLo = 100 - CyHi;                                  // calculate for pulse width low ratio %
}  //end control loop

//______________________________________________________________________
Footer
© 2023 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
