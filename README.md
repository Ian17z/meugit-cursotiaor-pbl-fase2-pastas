# Cultura: Soja
**Dados:** Embrapa

## Parâmetros de Solo e Nutrientes

* **Faixa adequada de pH do solo:** A faixa ideal de pH em água para a soja situa-se entre 5,5 e 6,5. Manter o pH próximo a 6,0 maximiza a produtividade, pois é neste intervalo que as plantas encontram a melhor condição para assimilação de macronutrientes (fósforo, potássio, nitrogênio).
* **Nitrogênio (N):** A soja obtém o nitrogênio de que precisa quase que exclusivamente através da Fixação Biológica de Nitrogênio (FBN), realizada dentro do ciclo do nitrogênio. Porém, caso seja necessário, a aplicação é recomendada no máximo 20 kg/ha. **Vamos utilizar 10 kg/ha.**
* **Fósforo (P):** A adubação de manutenção exige de 60 a 80 kg/ha de P₂O₅ para sustentar produtividades entre 3 e 4 toneladas de grãos por hectare. Em visão disso, **vamos utilizar a média de 70 kg/ha** para o cultivo.
* **Potássio (K):** Para cada tonelada de grãos produzida, a soja demanda aproximadamente 48 a 50 kg/ha de K₂O. Em visão disso, **vamos utilizar a média de 49 kg/ha.**

## Necessidade de Água e Umidade

Para atingir o seu máximo potencial produtivo, a cultura da soja necessita de 450 a 800 mm de água por ciclo. O limite inferior ou superior dependerá diretamente das condições climáticas locais (temperatura, umidade relativa do ar) e do manejo do solo, como o uso de plantio direto na palhada, que ajuda a reter a umidade. Para nossa cultura **utilizaremos a média de 700 mm**, mas ficando atentos às mudanças climáticas e outras variações.

### Condições que indicam a necessidade de irrigação:

* **Fases Críticas:** O déficit hídrico é mais prejudicial durante a floração e o enchimento de grãos. A falta de água nessas fases aborta flores e reduz severamente o peso do grão.
* **Capacidade de Água Disponível (CAD):** Em solos mais arenosos ou com baixa CAD (ex: CAD de 70 mm), períodos sem chuva (veranicos) de 10 a 15 dias já sinalizam necessidade urgente de irrigação.
* **Monitoramento:** A decisão exata deve ser baseada no monitoramento do teor de umidade do solo por meio de tensiômetros ou sensores instalados na profundidade efetiva do sistema radicular (20 a 40 cm).

---

## Etapa 2 - Montando o circuito (WOKWI)

1. **ESP32:** Controla o circuito e executa o código.
2. **LDR:** Representa o pH por meio de uma leitura analógica simulada.
3. **Três botões:** Representam os nutrientes N, P e K.
4. **Sensor de umidade DHT22:** É a simulação da irrigação analisando a umidade do solo.
5. **LED vermelho:** Representa a bomba ligada quando a irrigação estiver ativa.

### Programa (Pseudocódigo)

```text
Ler N, P e K # Os botões
Ler o valor do pH do solo # LDR	
Ler a umidade do solo # DHT22

Converter o valor do LDR para uma escala que represente o pH simulado

Se a umidade estiver abaixo do limite:
  Ligar a irrigação
Caso contrário:
  Desligar a irrigação

Exibir as leituras e o estado da irrigação
Repetir

### Código completo

'''text
#include "DHT.h"

#define DHTPIN 15    // Fio azul do DHT22 no pino 15
#define DHTTYPE DHT22
#define LDRPIN 34    // LDR no pino 34
#define RELAYPIN 23  // LED/Relé transferido para o pino 23
#define BTN_N 22     // Botão superior (N) transferido para o pino 21
#define BTN_P 19     // Botão do meio (P) transferido para o pino 19
#define BTN_K 18      // Botão inferior (K) transferido para o pino 18

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
  
  // Configuração dos pinos
  pinMode(RELAYPIN, OUTPUT);
  pinMode(BTN_N, INPUT_PULLUP); // Usando pullup interno (botão fecha no GND)
  pinMode(BTN_P, INPUT_PULLUP);
  pinMode(BTN_K, INPUT_PULLUP);
  
  // Garante que a bomba inicie desligada
  digitalWrite(RELAYPIN, LOW);
  
  Serial.println("FarmTech Solutions - Iniciando Sistema de Monitoramento de Soja");
}

void loop() {
  // 1. Leitura de Umidade (DHT22)
  float umidade = dht.readHumidity();
  
  // 2. Leitura e Conversão do pH (LDR)
  // O LDR no ESP32 gera valores de 0 a 4095. Mapeamos isso para 0.0 a 14.0 -> 0 Ácido || 7 Neutro || 14 Básico
  int valorLDR = analogRead(LDRPIN);
  float pH = (valorLDR / 4095.0) * 14.0;
  
  // 3. Leitura dos Nutrientes (Botões em PULLUP: LOW significa pressionado/presente)
  bool nivelN = !digitalRead(BTN_N); 
  bool nivelP = !digitalRead(BTN_P);
  bool nivelK = !digitalRead(BTN_K);

  // Verificação de falha de leitura do DHT
  if (isnan(umidade)) {
    Serial.println("Erro ao ler o sensor de humidade!");
    delay(2000);
    return;
  }

  // 4. Exibição do Dashboard no Serial Monitor
  Serial.println("-----------------------------------------------------------");
  Serial.print("Umidade do Solo: "); Serial.print(umidade); Serial.println("%");
  Serial.print("pH Atual: "); Serial.println(pH);
  Serial.print("Status NPK -> N: "); Serial.print(nivelN ? "ALTO (Alerta)" : "OK");
  Serial.print(" | P: "); Serial.print(nivelP ? "OK" : "O nível de fósforo está baixo");
  Serial.print(" | K: "); Serial.println(nivelK ? "OK" : "O nível de potássio está baixo");

  // 5. Lógica de Decisão da Irrigação
  bool phCritico = (pH < 5.5 || pH > 6.5);
  bool soloSeco = (umidade < 65.0);

  if (phCritico) {
    Serial.println("ALERTA: pH em nível crítico! Irrigação bloqueada para evitar toxidez.");
    digitalWrite(RELAYPIN, LOW); // Desliga/Mantém desligada a bomba
  } 
  else if (soloSeco) {
    Serial.println("AÇÃO: Solo seco detectado. Ligando irrigação (Relé ON).");// RELÉ ON
    digitalWrite(RELAYPIN, HIGH);
  } 
  else {
    Serial.println("AÇÃO: Umidade adequada. Irrigação desativada (Relé OFF).");//Relé OFF
    digitalWrite(RELAYPIN, LOW);
  }
  delay(3000); // Aguarda 3 segundos antes da próxima leitura
}
