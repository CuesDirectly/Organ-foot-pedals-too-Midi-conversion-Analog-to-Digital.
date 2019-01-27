# Organ-foot-pedals-too-Midi-conversion-Analog-to-Digital.
Using the Arduino Mega, I will be converting the Analog foot pedals into Midi controllers/Bass pedals, at this time I could sure use some advice.   

If this were a flow chart, it would be a simple one.   

Using the Arudino Mega, I plan to use the Analog pins except for the final Midi output.

The foot pedals too start the project will be 25 pedals with a 20 pin set up.   I am able to use pins to send a signal thru a Pull Up application to activate all pedals thru 14 pins.  (There are 4 pins remaining and they all appear to go to ground on the PCB, I may be wrong and will address before final.) 

Simplified;

Send two analog signals --> Use Pull Up to add 5v --> send thru pedals --> pedals send return on 1 of 14 wires into Analog 2-16.

Start coding the Arduino Mega to achieve eventual Midi output thru Pin 1 and basic job is completed.  


Next up will be adding to the sounds and manipulating the pedals with effect like pedal sensitivity --> reverb --> delay --> second set of keys --> third set of keys and be happy for now.

Thanks for any and all who read this.





#define NUM_ROWS 2

#define NUM_COLS 14



#define NOTE_ON_CMD 0x90

#define NOTE_OFF_CMD 0x80

#define NOTE_VELOCITY 127



//MIDI baud rate

#define SERIAL_RATE 31250



// Pin Definitions


pinMode(A0, OUTPUT);

pinMode(A1, OUTPUT);

pinMode(A2, INPUT);

pinMode(A3, INPUT);

pinMode(A4, INPUT);

pinMode(A5, INPUT);

pinMode(A6, INPUT);

pinMode(A7, INPUT);

pinMode(A8, INPUT);

pinMode(A9, INPUT);

pinMode(A10, INPUT);

pinMode(A11, INPUT);

pinMode(A12, INPUT);

pinMode(A13, INPUT);

pinMode(A14, INPUT);

pinMode(A15, INPUT);

pinMode(A16, INPUT);












boolean keyPressed[NUM_ROWS][NUM_COLS];

uint8_t keyToMidiMap[NUM_ROWS][NUM_COLS];



// bitmasks for scanning columns

int bits[] =

{ 

  B11111110,

  B11111101,

  B11111011,

  B11110111,

  B11101111,

  B11011111,

  B10111111,

  B01111111

};



void setup()

{

  int note = 24;



  for(int colCtr = 0; colCtr < NUM_COLS; ++colCtr)

  {

    for(int rowCtr = 0; rowCtr < NUM_ROWS; ++rowCtr)

    {

      keyPressed[rowCtr][colCtr] = false;

      keyToMidiMap[rowCtr][colCtr] = note;

      note++;

    }

  }



  // setup pins output/input mode
  



 



  Serial.begin(SERIAL_RATE);

}



void loop()

{

  for (int colCtr = 0; colCtr < NUM_COLS; ++colCtr)

  {

    //scan next column

    scanColumn(colCtr);



    //get row values at this column

    int rowValue[NUM_ROWS];

    rowValue[0] = !digitalRead(row1Pin);

    rowValue[2] = !analogRead(A2Pin);

    rowValue[3] = !analogRead(A3Pin);

    rowValue[4] = !analogRead(A4Pin);

    rowValue[5] = !analogRead(A5Pin);

    rowValue[6] = !analogRead(A6Pin);

    rowValue[7] = !analogRead(A7Pin);

    rowValue[8] = !analogRead(A8Pin);

    rowValue[9] = !analogRead(A9Pin);

    rowValue[10] = !analogRead(A10Pin);

    rowValue[11] = !analogRead(A11Pin);

    rowValue[12] = !analogRead(A12Pin);

    rowValue[13] = !analogRead(A13Pin);

    rowValue[14] = !analogRead(A14Pin);

    rowValue[15] = !analogRead(A15Pin);

    rowValue[16] = !analogRead(A16Pin);




    // process keys pressed

    for(int rowCtr=0; rowCtr<NUM_ROWS; ++rowCtr)

    {

      if(rowValue[rowCtr] != 0 && !keyPressed[rowCtr][colCtr])

      {

        keyPressed[rowCtr][colCtr] = true;

        noteOn(rowCtr,colCtr);

      }

    }



    // process keys released

    for(int rowCtr=0; rowCtr<NUM_ROWS; ++rowCtr)

    {

      if(rowValue[rowCtr] == 0 && keyPressed[rowCtr][colCtr])

      {

        keyPressed[rowCtr][colCtr] = false;

        noteOff(rowCtr,colCtr);

      }

    }

  }

}



void scanColumn(int colNum)

{

  digitalWrite(latchPin, LOW);



  if(2 <= colNum && colNum <= 16)

  {

    shiftOut(dataPin, clockPin, MSBFIRST, B11111111); //right sr

    shiftOut(dataPin, clockPin, MSBFIRST, bits[colNum]); //left sr

  }

  else

  {

    shiftOut(dataPin, clockPin, MSBFIRST, bits[colNum-8]); //right sr

    shiftOut(dataPin, clockPin, MSBFIRST, B11111111); //left sr

  }

  digitalWrite(latchPin, HIGH);

}



void noteOn(int row, int col)

{

  Serial.write(NOTE_ON_CMD);

  Serial.write(keyToMidiMap[row][col]);

  Serial.write(NOTE_VELOCITY);

}



void noteOff(int row, int col)

{

  Serial.write(NOTE_OFF_CMD);

  Serial.write(keyToMidiMap[row][col]);

  Serial.write(NOTE_VELOCITY);

}


