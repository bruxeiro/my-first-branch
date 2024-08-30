# my-first-branch


#include <AccelStepper.h>
#include <ezButton.h>

const int PASSOS_POR_REVOLUCAO = 200;
const int GRAUS_POR_PASSO = 1.8;
const int PASSOS_POR_INCREMENTO = 30 / GRAUS_POR_PASSO;

const int PINO_PASSO = 2;
const int PINO_DIRECAO = 3;
const int PINO_ATIVAR = 4;
const int PINO_BOTAO = 5;
const int PINO_CHAVE = 6;
const int PINO_POTENCIOMETRO = A0;

AccelStepper motor(AccelStepper::DRIVER, PINO_PASSO, PINO_DIRECAO);
ezButton botao(PINO_BOTAO);

int velocidade = 60;
bool zeroEncontrado = false;

void setup() {
  motor.setMaxSpeed(1000);
  motor.setAcceleration(500);
  
  pinMode(PINO_ATIVAR, OUTPUT);
  digitalWrite(PINO_ATIVAR, LOW);
  
  pinMode(PINO_CHAVE, INPUT_PULLUP);
  
  botao.setDebounceTime(50);
  
  Serial.begin(9600);
}

void loop() {
  botao.loop();
  atualizarVelocidade();
  
  if (botao.isPressed()) {
    if (zeroEncontrado) {
      moverIncremento();
    } else {
      encontrarZero();
    }
  }
  
  if (digitalRead(PINO_CHAVE) == LOW && !zeroEncontrado) {
    zeroEncontrado = true;
    motor.setCurrentPosition(0);
    Serial.println("Posição zero encontrada!");
  }
  
  motor.run();
}

void atualizarVelocidade() {
  velocidade = map(analogRead(PINO_POTENCIOMETRO), 0, 1023, 60, 1000);
  motor.setMaxSpeed(velocidade);
}

void moverIncremento() {
  motor.move(PASSOS_POR_INCREMENTO);
  Serial.println("Movendo 30 graus");
}

void encontrarZero() {
  Serial.println("Procurando posição zero...");
  while (digitalRead(PINO_CHAVE) == HIGH) {
    motor.moveTo(motor.currentPosition() - 1);
    motor.run();
  }
  zeroEncontrado = true;
  motor.setCurrentPosition(0);
  Serial.println("Posição zero encontrada!");
}
