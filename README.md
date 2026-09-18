#include <LiquidCrystal.h>

// ======================================================
// JOYSTICK VARIABLES - PLAYER 1
// PLAYER 1 JOYSTICK CONTROLS TANK #2
// ======================================================

int vrx1 = A0;
int vry1 = A1;
int sw1 = 8;

int sw_val1;
int x_val1;
int y_val1;


// ======================================================
// JOYSTICK VARIABLES - PLAYER 2
// PLAYER 2 JOYSTICK CONTROLS TANK #1
// JOYSTICK IS UPSIDE DOWN
// ======================================================

int vrx2 = A3;
int vry2 = A2;
int sw2 = 7;

int sw_val2;
int x_val2;
int y_val2;


// ======================================================
// LEDS
// ======================================================

// Tank #2 LEDs
int p2r = 3;
int p2g = 4;

// Tank #1 LEDs
int p1r = 6;
int p1g = 5;


// ======================================================
// BUZZER
// ======================================================

int buzzer = 2;


// ======================================================
// LCD
// ======================================================

int rs = 53;
int e = 51;
int d4 = 49;
int d5 = 9;
int d6 = 10;
int d7 = 47;

LiquidCrystal lcd(rs, e, d4, d5, d6, d7);


// ======================================================
// CUSTOM LCD TANK SPRITES
// ======================================================
//
// LCD allows 8 custom characters.
//
// 0 = Tank 1 facing RIGHT
// 1 = Tank 1 facing LEFT
// 2 = Tank 2 facing RIGHT
// 3 = Tank 2 facing LEFT
// 4 = Tank 1 bullet
// 5 = Tank 2 bullet
//
// ======================================================


// ======================================================
// TANK #1 - HEAVY TANK
// ======================================================

byte tank1Right[8] = {

  B00000,
  B00100,
  B01110,
  B11111,
  B10101,
  B11111,
  B01010,
  B00000
};


byte tank1Left[8] = {

  B00000,
  B00100,
  B01110,
  B11111,
  B10101,
  B11111,
  B01010,
  B00000
};


// ======================================================
// TANK #2 - DIFFERENT LIGHT TANK
// ======================================================

byte tank2Right[8] = {

  B00000,
  B00010,
  B00110,
  B01111,
  B11101,
  B11111,
  B01010,
  B00000
};


byte tank2Left[8] = {

  B00000,
  B01000,
  B01100,
  B11110,
  B10111,
  B11111,
  B01010,
  B00000
};


// ======================================================
// BULLET SPRITES
// ======================================================

byte bullet1[8] = {

  B00000,
  B00000,
  B00000,
  B00100,
  B01110,
  B00100,
  B00000,
  B00000
};


byte bullet2[8] = {

  B00000,
  B00000,
  B00100,
  B01110,
  B11111,
  B01110,
  B00100,
  B00000
};


// ======================================================
// TANK POSITIONS
// ======================================================

// Tank #1 = LEFT
// Controlled by Joystick #2

int tank1x = 2;
int tank1y = 1;


// Tank #2 = RIGHT
// Controlled by Joystick #1

int tank2x = 13;
int tank2y = 1;


// ======================================================
// LIVES
// ======================================================

int tank1Lives = 3;
int tank2Lives = 3;


// ======================================================
// TANK DIRECTIONS
// ======================================================

// 1 = right
// -1 = left

int tank1Direction = 1;
int tank2Direction = -1;


// ======================================================
// BULLETS
// ======================================================

bool tank1BulletActive = false;
int tank1BulletX;
int tank1BulletY;
int tank1BulletDirection;


bool tank2BulletActive = false;
int tank2BulletX;
int tank2BulletY;
int tank2BulletDirection;


// ======================================================
// GAME STATES
// ======================================================

bool gameStarted = false;
bool gameOver = false;


// ======================================================
// TIMERS
// ======================================================

unsigned long lastMoveTime = 0;
unsigned long lastBulletTime = 0;

unsigned long startHoldTime = 0;
bool holdingToStart = false;

unsigned long gameOverTime = 0;
unsigned long lastGameOverBeep = 0;

unsigned long hitTime = 0;
bool hitEffect = false;


// ======================================================
// GAME SETTINGS
// ======================================================

int moveDelay = 200;
int bulletDelay = 150;


// ======================================================
// SETUP
// ======================================================

void setup() {

  Serial.begin(9600);


  // ====================================================
  // JOYSTICKS
  // ====================================================

  pinMode(vrx1, INPUT);
  pinMode(vry1, INPUT);
  pinMode(sw1, INPUT_PULLUP);

  pinMode(vrx2, INPUT);
  pinMode(vry2, INPUT);
  pinMode(sw2, INPUT_PULLUP);


  // ====================================================
  // LEDS
  // ====================================================

  pinMode(p2r, OUTPUT);
  pinMode(p2g, OUTPUT);

  pinMode(p1r, OUTPUT);
  pinMode(p1g, OUTPUT);


  // ====================================================
  // BUZZER
  // ====================================================

  pinMode(buzzer, OUTPUT);


  // Turn everything off

  digitalWrite(p1r, LOW);
  digitalWrite(p1g, LOW);

  digitalWrite(p2r, LOW);
  digitalWrite(p2g, LOW);

  digitalWrite(buzzer, LOW);


  // ====================================================
  // LCD
  // ====================================================

  lcd.begin(16, 2);


  // Load custom characters

  lcd.createChar(0, tank1Right);
  lcd.createChar(1, tank1Left);

  lcd.createChar(2, tank2Right);
  lcd.createChar(3, tank2Left);

  lcd.createChar(4, bullet1);
  lcd.createChar(5, bullet2);


  // ====================================================
  // START SCREEN
  // ====================================================

  lcd.clear();

  lcd.setCursor(0, 0);
  lcd.print("TANK BATTLE");

  lcd.setCursor(0, 1);
  lcd.print("HOLD TO START");
}


// ======================================================
// MAIN LOOP
// ======================================================

void loop() {

  // Waiting for players

  if (!gameStarted && !gameOver) {

    checkStart();

    return;
  }


  // Game over

  if (gameOver) {

    gameOverAnimation();

    return;
  }


  // Normal game

  readJoysticks();

  moveTanks();

  shoot();

  moveBullets();

  checkHits();

  drawGame();
}


// ======================================================
// READ JOYSTICKS
// ======================================================

void readJoysticks() {

  // ====================================================
  // JOYSTICK 1
  // CONTROLS TANK #2
  // ====================================================

  x_val1 = analogRead(vrx1);
  y_val1 = analogRead(vry1);
  sw_val1 = digitalRead(sw1);


  // ====================================================
  // JOYSTICK 2
  // CONTROLS TANK #1
  // UPSIDE DOWN
  // ====================================================

  x_val2 = analogRead(vrx2);
  y_val2 = analogRead(vry2);
  sw_val2 = digitalRead(sw2);


  // ====================================================
  // SERIAL MONITOR
  // ====================================================

  Serial.print("Joystick 1 X: ");
  Serial.print(x_val1);

  Serial.print("  Y: ");
  Serial.print(y_val1);

  Serial.print("  Button: ");
  Serial.print(sw_val1);

  Serial.print("     Joystick 2 X: ");
  Serial.print(x_val2);

  Serial.print("  Y: ");
  Serial.print(y_val2);

  Serial.print("  Button: ");
  Serial.println(sw_val2);
}


// ======================================================
// START GAME
// ======================================================

void checkStart() {

  bool player1Holding = (digitalRead(sw1) == LOW);
  bool player2Holding = (digitalRead(sw2) == LOW);


  if (player1Holding && player2Holding) {

    if (!holdingToStart) {

      holdingToStart = true;

      startHoldTime = millis();
    }


    unsigned long holdTime = millis() - startHoldTime;


    lcd.clear();

    lcd.setCursor(0, 0);
    lcd.print("BOTH READY!");


    lcd.setCursor(0, 1);


    if (holdTime < 1000) {

      lcd.print("3...");
    }

    else if (holdTime < 2000) {

      lcd.print("2...");
    }

    else if (holdTime < 3000) {

      lcd.print("1...");
    }


    if (holdTime >= 3000) {

      startGame();
    }

  }

  else {

    holdingToStart = false;


    lcd.clear();

    lcd.setCursor(0, 0);
    lcd.print("TANK BATTLE");

    lcd.setCursor(0, 1);
    lcd.print("HOLD TO START");
  }
}


// ======================================================
// START GAME
// ======================================================

void startGame() {

  gameStarted = true;
  gameOver = false;

  holdingToStart = false;


  // Reset lives

  tank1Lives = 3;
  tank2Lives = 3;


  // Tank #1 starts LEFT

  tank1x = 2;
  tank1y = 1;


  // Tank #2 starts RIGHT

  tank2x = 13;
  tank2y = 1;


  // Reset directions

  tank1Direction = 1;
  tank2Direction = -1;


  // Reset bullets

  tank1BulletActive = false;
  tank2BulletActive = false;


  // LEDs off

  digitalWrite(p1r, LOW);
  digitalWrite(p1g, LOW);

  digitalWrite(p2r, LOW);
  digitalWrite(p2g, LOW);


  lcd.clear();


  // Starting beep

  tone(buzzer, 1000, 150);

  delay(200);
}


// ======================================================
// MOVE TANKS
// ======================================================

void moveTanks() {

  if (millis() - lastMoveTime < moveDelay) {

    return;
  }

  lastMoveTime = millis();


  // ====================================================
  // TANK #1
  // JOYSTICK #2
  // UPSIDE-DOWN JOYSTICK
  // ====================================================


  // Physical RIGHT
  // Raw X is LOW

  if (x_val2 < 350) {

    if (tank1x < 15) {

      tank1x++;
    }

    tank1Direction = 1;
  }


  // Physical LEFT
  // Raw X is HIGH

  if (x_val2 > 650) {

    if (tank1x > 0) {

      tank1x--;
    }

    tank1Direction = -1;
  }


  // Physical UP
  // Raw Y is HIGH

  if (y_val2 > 650) {

    if (tank1y > 0) {

      tank1y--;
    }
  }


  // Physical DOWN
  // Raw Y is LOW

  if (y_val2 < 350) {

    if (tank1y < 1) {

      tank1y++;
    }
  }


  // ====================================================
  // TANK #2
  // JOYSTICK #1
  // NORMAL
  // ====================================================


  // LEFT

  if (x_val1 < 350) {

    if (tank2x > 0) {

      tank2x--;
    }

    tank2Direction = -1;
  }


  // RIGHT

  if (x_val1 > 650) {

    if (tank2x < 15) {

      tank2x++;
    }

    tank2Direction = 1;
  }


  // UP

  if (y_val1 < 350) {

    if (tank2y > 0) {

      tank2y--;
    }
  }


  // DOWN

  if (y_val1 > 650) {

    if (tank2y < 1) {

      tank2y++;
    }
  }
}


// ======================================================
// SHOOTING
// ======================================================

void shoot() {

  static bool previousButton1 = HIGH;
  static bool previousButton2 = HIGH;


  bool currentButton1 = digitalRead(sw1);
  bool currentButton2 = digitalRead(sw2);


  // ====================================================
  // JOYSTICK #2 BUTTON → TANK #1
  // ====================================================

  if (previousButton2 == HIGH &&
      currentButton2 == LOW) {

    if (!tank1BulletActive) {

      tank1BulletActive = true;

      tank1BulletX = tank1x + tank1Direction;

      tank1BulletY = tank1y;

      tank1BulletDirection = tank1Direction;
    }
  }


  // ====================================================
  // JOYSTICK #1 BUTTON → TANK #2
  // ====================================================

  if (previousButton1 == HIGH &&
      currentButton1 == LOW) {

    if (!tank2BulletActive) {

      tank2BulletActive = true;

      tank2BulletX = tank2x + tank2Direction;

      tank2BulletY = tank2y;

      tank2BulletDirection = tank2Direction;
    }
  }


  previousButton1 = currentButton1;
  previousButton2 = currentButton2;
}


// ======================================================
// MOVE BULLETS
// ======================================================

void moveBullets() {

  if (millis() - lastBulletTime < bulletDelay) {

    return;
  }

  lastBulletTime = millis();


  // Tank #1 bullet

  if (tank1BulletActive) {

    tank1BulletX += tank1BulletDirection;


    if (tank1BulletX < 0 ||
        tank1BulletX > 15) {

      tank1BulletActive = false;
    }
  }


  // Tank #2 bullet

  if (tank2BulletActive) {

    tank2BulletX += tank2BulletDirection;


    if (tank2BulletX < 0 ||
        tank2BulletX > 15) {

      tank2BulletActive = false;
    }
  }
}


// ======================================================
// CHECK FOR HITS
// ======================================================

void checkHits() {

  // Tank #1 → Tank #2

  if (tank1BulletActive) {

    if (tank1BulletX == tank2x &&
        tank1BulletY == tank2y) {

      tank1BulletActive = false;

      tank2Lives--;

      tank2Hit();
    }
  }


  // Tank #2 → Tank #1

  if (tank2BulletActive) {

    if (tank2BulletX == tank1x &&
        tank2BulletY == tank1y) {

      tank2BulletActive = false;

      tank1Lives--;

      tank1Hit();
    }
  }
}


// ======================================================
// TANK #2 GETS HIT
// ======================================================

void tank2Hit() {

  // Tank #1 hit Tank #2

  digitalWrite(p1g, HIGH);

  digitalWrite(p2r, HIGH);

  tone(buzzer, 800, 250);


  hitTime = millis();

  hitEffect = true;


  if (tank2Lives <= 0) {

    endGame(1);
  }
}


// ======================================================
// TANK #1 GETS HIT
// ======================================================

void tank1Hit() {

  // Tank #2 hit Tank #1

  digitalWrite(p2g, HIGH);

  digitalWrite(p1r, HIGH);

  tone(buzzer, 800, 250);


  hitTime = millis();

  hitEffect = true;


  if (tank1Lives <= 0) {

    endGame(2);
  }
}


// ======================================================
// DRAW GAME
// ======================================================

void drawGame() {

  lcd.clear();


  // ====================================================
  // TANK #1 LIVES
  // LEFT SIDE
  // ====================================================

  lcd.setCursor(0, 0);

  if (tank1Lives >= 1) {
    lcd.print(".");
  }

  if (tank1Lives >= 2) {
    lcd.print(".");
  }

  if (tank1Lives >= 3) {
    lcd.print(".");
  }


  // ====================================================
  // TANK #2 LIVES
  // RIGHT SIDE
  // ====================================================

  lcd.setCursor(13, 0);

  if (tank2Lives >= 1) {
    lcd.print(".");
  }

  if (tank2Lives >= 2) {
    lcd.print(".");
  }

  if (tank2Lives >= 3) {
    lcd.print(".");
  }


  // ====================================================
  // DRAW TANK #1
  // ====================================================

  lcd.setCursor(tank1x, tank1y);

  if (tank1Direction == 1) {

    lcd.write(byte(0));
  }

  else {

    lcd.write(byte(1));
  }


  // ====================================================
  // DRAW TANK #2
  // ====================================================

  lcd.setCursor(tank2x, tank2y);

  if (tank2Direction == 1) {

    lcd.write(byte(2));
  }

  else {

    lcd.write(byte(3));
  }


  // ====================================================
  // DRAW TANK #1 BULLET
  // ====================================================

  if (tank1BulletActive) {

    if (!(tank1BulletX == tank2x &&
          tank1BulletY == tank2y)) {

      lcd.setCursor(tank1BulletX, tank1BulletY);

      lcd.write(byte(4));
    }
  }


  // ====================================================
  // DRAW TANK #2 BULLET
  // ====================================================

  if (tank2BulletActive) {

    if (!(tank2BulletX == tank1x &&
          tank2BulletY == tank1y)) {

      lcd.setCursor(tank2BulletX, tank2BulletY);

      lcd.write(byte(5));
    }
  }


  // ====================================================
  // TURN OFF HIT LIGHTS
  // ====================================================

  if (hitEffect &&
      millis() - hitTime > 400) {

    digitalWrite(p1g, LOW);
    digitalWrite(p2g, LOW);

    digitalWrite(p1r, LOW);
    digitalWrite(p2r, LOW);

    hitEffect = false;
  }
}


// ======================================================
// END GAME
// ======================================================

void endGame(int winner) {

  gameStarted = false;

  gameOver = true;

  gameOverTime = millis();

  lastGameOverBeep = 0;


  lcd.clear();

  lcd.setCursor(0, 0);


  // Tank #1 wins

  if (winner == 1) {

    lcd.print("PLAYER 1 WINS!");

    digitalWrite(p1g, HIGH);

    digitalWrite(p2r, HIGH);
  }


  // Tank #2 wins

  else {

    lcd.print("PLAYER 2 WINS!");

    digitalWrite(p2g, HIGH);

    digitalWrite(p1r, HIGH);
  }


  lcd.setCursor(0, 1);

  lcd.print("GAME OVER!");
}


// ======================================================
// GAME OVER ANIMATION
// ======================================================

void gameOverAnimation() {

  if (millis() - lastGameOverBeep > 600) {

    lastGameOverBeep = millis();

    tone(buzzer, 1000, 250);
  }


  // Shut down after 5 seconds

  if (millis() - gameOverTime > 5000) {

    shutDownGame();
  }
}


// ======================================================
// SHUT DOWN GAME
// ======================================================

void shutDownGame() {

  gameOver = false;

  gameStarted = false;


  // LEDs off

  digitalWrite(p1r, LOW);
  digitalWrite(p1g, LOW);

  digitalWrite(p2r, LOW);
  digitalWrite(p2g, LOW);


  noTone(buzzer);


  // Return to start screen

  lcd.clear();

  lcd.setCursor(0, 0);

  lcd.print("TANK BATTLE");

  lcd.setCursor(0, 1);

  lcd.print("HOLD TO START");
}
