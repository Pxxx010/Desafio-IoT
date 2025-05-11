# Sistema de Automação para Ventilação de Servidores

## 1. Problema/Oportunidade e Justificativa

### Problema

Os data centers e salas de servidores de provedores de internet enfrentam desafios significativos relacionados ao controle de temperatura. Servidores em funcionamento geram calor constantemente, e o superaquecimento pode causar:

* Degradação prematura de componentes eletrônicos
* Falhas inesperadas de hardware
* Perda de dados críticos
* Interrupção de serviços para os clientes
* Aumento de custos operacionais devido ao consumo energético elevado

### Justificativa

A utilização de sistemas IoT com sensores para monitoramento e controle automático da temperatura oferece:

* Monitoramento em tempo real 24/7 sem intervenção manual constante
* Resposta rápida a mudanças de temperatura para evitar danos
* Redução do consumo de energia ao otimizar o uso dos sistemas de ventilação
* Coleta de dados para análise de tendências e manutenção preventiva
* Alertas remotos que permitem intervenção antes que problemas críticos ocorram

## 2. Ideia da Solução

Desenvolvimento de um sistema de automação IoT para monitoramento e controle automático da ventilação em ambientes de servidores, utilizando:

**Componentes:**

* Sensor DHT22 para medição precisa de temperatura e umidade
* ESP32 como controlador principal, oferecendo Wi-Fi integrado e capacidade de processamento
* Dois relés para controle independente de dispositivos de refrigeração (ventiladores, ar-condicionado)
* Plataforma ThingSpeak para visualização remota de dados e análise de tendências

**Funcionamento:**

* O sistema monitora constantemente a temperatura e umidade do ambiente
* Com base em limiares predefinidos, ativa ou desativa automaticamente os sistemas de ventilação
* Os dados são enviados a cada 30 segundos para a nuvem, permitindo monitoramento remoto
* Diferentes níveis de resposta são programados com base na gravidade da situação térmica:
  * Temperatura acima de 35°C: ativação de todos os sistemas de refrigeração (modo de emergência)
  * Temperatura entre 30-35°C: ventilação moderada
  * Temperatura abaixo de 30°C: modo econômico de refrigeração

## 3. Esboço da Arquitetura

O sistema é composto por:

1. **Hardware:**
   * ESP32 (unidade central de processamento)
   * Sensor DHT22 (temperatura e umidade)
   * Dois módulos de relé (controle de dispositivos)
   * Breadboard para prototipagem
   * Conexões e cabos
2. **Software:**
   * Código em C++ para ESP32
   * Bibliotecas para DHT22, WiFi e ThingSpeak
   * Lógica de controle baseada em limiares de temperatura
3. **Nuvem:**
   * Plataforma ThingSpeak para armazenamento e visualização de dados
   * Gráficos de temperatura e umidade em tempo real
   * Registro histórico para análise de tendências

![Diagrama de Conexão](https://i.imgur.com/YkxTGKJ.png)
*Diagrama mostrando a conexão entre ESP32, sensor DHT22 e módulos de relé*

## 4. Integração com ThingSpeak

O ThingSpeak é utilizado como plataforma central para visualização e análise dos dados coletados:

* **Canal dedicado** (ID: 2943258) para receber e armazenar os dados
* **Campo 1:** Temperatura em °C
* **Campo 2:** Umidade em %
* **Visualizações personalizadas** incluindo:
  * Gráfico de temperatura em tempo real
  * Gráfico de umidade em tempo real
  * Histórico de temperatura nas últimas 24 horas
  * Indicadores visuais para alertas de temperatura

O sistema envia dados a cada 30 segundos para o ThingSpeak, garantindo um monitoramento quase em tempo real das condições da sala de servidores.

[Link para o canal ThingSpeak](https://thingspeak.com/channels/2943258)

## 5. Código do Projeto

```cpp
#include <WiFi.h>
#include "DHTesp.h"
#include "ThingSpeak.h"

// Pinos
const int DHT_PIN = 15;       // DHT22 no GPIO15
const int relay1Pin = 2;      // Relé 1 no GPIO2
const int relay2Pin = 4;      // Relé 2 no GPIO4

// Wi-Fi
const char* WIFI_NAME = "Wokwi-GUEST";
const char* WIFI_PASSWORD = "";

// ThingSpeak
const int myChannelNumber = 2943258;
const char* myApiKey = "P159Q7RTA3PLYUCU";
const char* server = "api.thingspeak.com";

// Objetos
DHTesp dhtSensor;
WiFiClient client;

void setup() {
  Serial.begin(115200);
  
  // Configura pinos
  pinMode(relay1Pin, OUTPUT);
  pinMode(relay2Pin, OUTPUT);
  digitalWrite(relay1Pin, LOW);
  digitalWrite(relay2Pin, LOW);

  // Inicializa DHT
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);

  // Conecta Wi-Fi
  WiFi.begin(WIFI_NAME, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Conectando ao WiFi...");
  }
  Serial.println("WiFi conectado!");
  Serial.println("IP local: " + String(WiFi.localIP()));
  WiFi.mode(WIFI_STA);

  // Inicializa ThingSpeak
  ThingSpeak.begin(client);
}

void loop() {
  TempAndHumidity data = dhtSensor.getTempAndHumidity();
  float tempC = data.temperature;
  float hum = data.humidity;

  Serial.println("Temperatura: " + String(tempC, 1) + " °C");
  Serial.println("Umidade: " + String(hum, 1) + " %");

  // Envia dados ao ThingSpeak
  ThingSpeak.setField(1, tempC);
  ThingSpeak.setField(2, hum);
  int x = ThingSpeak.writeFields(myChannelNumber, myApiKey);
  if (x == 200) {
    Serial.println("Dados enviados com sucesso");
  } else {
    Serial.println("Erro ao enviar: " + String(x));
  }

  // Controle dos relés com base na temperatura
  if (tempC > 35) {
    digitalWrite(relay1Pin, HIGH);
    digitalWrite(relay2Pin, HIGH);
  } else if (tempC < 30) {
    digitalWrite(relay1Pin, HIGH);
    digitalWrite(relay2Pin, LOW);
  } else {
    digitalWrite(relay1Pin, HIGH);
    digitalWrite(relay2Pin, LOW);
  }

  Serial.println("---");
  delay(30000);  // Espera 30 segundos
}
```

## 6. Implementação no Wokwi

O projeto foi implementado e testado na plataforma Wokwi, permitindo simular o funcionamento completo do sistema sem necessidade de hardware físico:

[Link para o projeto no Wokwi](https://wokwi.com/projects/430593361022688257)

## 7. Benefícios do Sistema

* **Prevenção de danos:** Evita falhas de hardware causadas por superaquecimento
* **Economia energética:** Otimiza o uso dos sistemas de refrigeração
* **Monitoramento remoto:** Permite acompanhamento das condições sem estar fisicamente presente
* **Dados para tomada de decisão:** Fornece insights sobre padrões térmicos e desempenho da infraestrutura
* **Aumento de disponibilidade:** Reduz o risco de interrupções de serviço causadas por problemas térmicos

## 8. Possíveis Expansões Futuras

* Adição de sensores de qualidade do ar para detecção de poeira ou gases nocivos
* Integração com sistemas de notificação para alertas via SMS/email
* Implementação de algoritmos de IA para predição de falhas térmicas
* Expansão para controle de múltiplas salas de servidores
* Adição de interface web personalizada para gerenciamento

## 9. Referências

* Documentação do ESP32: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/
* Biblioteca DHTesp: https://github.com/beegee-tokyo/DHTesp
* Documentação ThingSpeak: https://www.mathworks.com/help/thingspeak/
* Boas práticas para resfriamento de servidores: https://www.datacenterdynamics.com/
