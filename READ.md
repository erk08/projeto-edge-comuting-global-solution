 Edge Computing - Global Solution
 
 Membros do grupo:
 Anna giazzi -RM: 569846
 Erick diego  - RM: 574064
 Giulia stella - RM: 568888
 Julia lopes - RM: 574128
 Rafael william - RM: 570738
 
Este repositório contém o desenvolvimento do circuito e da lógica de borda para o projeto da Global Solution.

##  Atalho para o Tinkercad
Para visualizar a simulação interativa do circuito, os componentes e o funcionamento em tempo real, acesse o link abaixo:

👉 **[Acessar Simulação no Tinkercad](https://www.tinkercad.com/things/857jPtZhVT0-edge-computing-satelite-de-monitoramento-?sharecode=5QSuTDLwHc4M8UGw5RM6-nZrQtkTIpuQCvirJIY2KBg)**

---

## 📝 Descrição do Projeeto

PROJETO GLOBAL SOLUTION 2026 - ENGENHARIA DE SOFTWARE FIAP
TEMA: Indústria Espacial - Edge Computing & Computer Systems
 
DESCRIÇÃO: Estação de Monitoramento Inteligente (Edge Node)
Este código realiza a leitura de sensores críticos e toma decisões locais 
antes de simular a transmissão de dados para um satélite.


## 🛠️ Hardware e Componentes (Simulação)
* 1x Arduino Uno R3
* 1x sensor tmp-36(sensor de temperatura)
* 1x sensor de umidade do solo
* 1x ldr(botão luminoso)
* 1x piezo(buzzer)
* 1x LCD crystal
* 3x leds
* 6x Resistores
  

## 💻 Código Fonte
O código carregado no microcontrolador está disponível na simulação do Tinkercad e também pode ser visualizado abaixo:
/**
* Membros do projeto:
* Anna Giazzi
* Erick Diego
* Giulia Stella
* Julia Lopes 
* Rafael Morales
*/


// biblioteca padrão do lcd crystal
#include <LiquidCrystal.h>

// -> configuração de pinos do lcd crystal(12,11,5,4,3,2)
const int rs = 12, en = 11, d4 = 5, d5 = 4, d6 = 3, d7 = 2;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

// -> configuração sensores de umidade,luminosidade e temperatura
const int tempPin = A0;    
const int umidadePin = A1; 
const int ldrPin = A2;     
const int ledTxPin = 13;   // led de simulacao de envio dados para o satelite de monitoramento

// -> NOVOS COMPONENTES DE ALERTA
const int piezoPin = 9;       // Buzzer/Piezo para alertas sonoros
const int ledCalorPin = 7;    // LED Amarelo para alerta de alta temperatura
const int ledSecoPin = 6;     // LED para alerta de solo seco (ex: Vermelho ou Laranja)

// -> variaveis globais
unsigned long lastTransmissionTime = 0;
const unsigned long transmissionInterval = 10000; // Transmite a cada 10 segundos

void setup() {
  // começo de simulação de projeto 
  lcd.begin(16, 2);
  
  // Configuração das saídas
  pinMode(ledTxPin, OUTPUT);
  pinMode(piezoPin, OUTPUT);
  pinMode(ledCalorPin, OUTPUT);
  pinMode(ledSecoPin, OUTPUT);
  
  Serial.begin(9600);
  
  // mensagem inicial para simulação 
  lcd.setCursor(0, 0);
  lcd.print("inicio projeto");
  lcd.setCursor(0, 1);
  lcd.print("Fiap satelite..");
  delay(2000);
  lcd.clear();
}

void loop() {
  // -> base de leitura dos sensores 
  
  // Temperatura (TMP36)
  int tempRead = analogRead(tempPin);
  float voltage = tempRead * (5.0 / 1024.0);
  float tempC = (voltage - 0.5) * 100;

  // Umidade (Mapeamento de 0 a 100%)
  int umidadeRead = analogRead(umidadePin);
  int umidadePercent = map(umidadeRead, 0, 1023, 0, 100);

  // Luminosidade (Mapeamento corrigido para o limite do simulador)
  int ldrRead = analogRead(ldrPin);
  int luzPercent = map(ldrRead, 0, 675, 0, 100);
  luzPercent = constrain(luzPercent, 0, 100); // Garante estabilidade entre 0% e 100%

  // 2. LÓGICA DE EDGE COMPUTING (Local Decision Making) & ATUADORES
  bool alertStatus = false;
  String alertMsg = "";

  if (tempC > 45.0) {
    alertStatus = true;
    alertMsg = "Alerta: Calor!";
    
    digitalWrite(ledCalorPin, HIGH); // Acende o LED Amarelo
    digitalWrite(ledSecoPin, LOW);
    tone(piezoPin, 1000);            // Apita o Piezo (frequência de 1000Hz)
  } 
  else if (umidadePercent < 20) {
    alertStatus = true;
    alertMsg = "Alerta: Seco!";
    
    digitalWrite(ledSecoPin, HIGH);  // Acende o LED de solo seco
    digitalWrite(ledCalorPin, LOW);
    tone(piezoPin, 1500);            // Apita o Piezo (frequência de 1500Hz)
  } 
  else {
    // Se tudo estiver OK, desliga os alertas de forma limpa (sem travar o loop)
    digitalWrite(ledCalorPin, LOW);
    digitalWrite(ledSecoPin, LOW);
    noTone(piezoPin); 
  }

  // -> configurar itens mostrados no lcd
  if (alertStatus) {
    lcd.setCursor(0, 0);
    lcd.print(alertMsg);
    // Espaços extras no final limpam caracteres residuais
    for (int i = alertMsg.length(); i < 16; i++) lcd.print(" "); 
    
    lcd.setCursor(0, 1);
    lcd.print("T:");
    lcd.print((int)tempC);
    lcd.print("C U:");
    lcd.print(umidadePercent);
    lcd.print("%   "); 
  } else {
    lcd.setCursor(0, 0);
    lcd.print("T:");
    lcd.print((int)tempC);
    lcd.print("C  U:");
    lcd.print(umidadePercent);
    lcd.print("%   "); 
    
    lcd.setCursor(0, 1);
    lcd.print("Luz:");
    lcd.print(luzPercent);
    lcd.print("%  Status:OK   "); 
  }

  // -> simulação da transmissão de dados orbital 
  if (millis() - lastTransmissionTime >= transmissionInterval) {
    noTone(piezoPin); // Garante silêncio no uplink
    transmitData(tempC, umidadePercent, luzPercent);
    lastTransmissionTime = millis();
  }

  delay(500); 
}

void transmitData(float t, int u, int l) {
  digitalWrite(ledTxPin, HIGH);
  
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(">>> Carregando <<<");
  lcd.setCursor(0, 1);
  lcd.print("Envio de Dados...");
  
  Serial.println("Novos dados de telemetria via satelite");
  Serial.print("{ \"temp\": "); Serial.print(t);
  Serial.print(", \"umid\": "); Serial.print(u);
  Serial.print(", \"luz\": "); Serial.print(l);
  Serial.println(" }");
  
  delay(1500); 
  digitalWrite(ledTxPin, LOW);
  lcd.clear();
}

