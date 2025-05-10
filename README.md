# 💧 Sistema de Irrigação Inteligente com ESP32 - Simulação Wokwi

Este projeto simula um sistema agrícola automatizado usando um microcontrolador ESP32 na plataforma [Wokwi](https://wokwi.com/). A irrigação é controlada com base na presença de nutrientes (Fósforo e Potássio), no pH e na umidade do solo. Quando detectadas condições inadequadas, a bomba é ativada automaticamente por 30 segundos.

## 🚀 Objetivo

Monitorar as condições do solo e realizar irrigação automática somente quando necessário, otimizando o uso de recursos hídricos na agricultura.

## 🧠 Sensores Simulados

- **Sensor de Fósforo (P):** Botão físico (TECLA 2)
  `Pressionado` → Fósforo presente 
  `Solto` → Fósforo ausente

- **Sensor de Potássio (K):** Botão físico (TECLA 1)
  `Pressionado` → Potássio presente  
  `Solto` → Potássio ausente

- **Sensor de pH:** Sensor LDR (luz como analogia ao valor de pH)  
  Variações na luminosidade simulam mudanças no pH do solo.

- **Sensor de Umidade:** Sensor DHT22  
  Mede a umidade relativa e simula a umidade do solo.

## ⚙️ Componentes Utilizados

- ESP32
- 2 Botões (P e K)
- LDR + Resistor
- Sensor DHT22
- Relé (representando a bomba)
- LED indicador da bomba
- Jumpers e resistores

## 🧪 Funcionamento

- O sistema verifica os sensores a cada ciclo.
- Se **algum nutriente estiver ausente** ou a umidade for **inferior a 40%**, a irrigação é ativada.
- A **bomba funciona por 30 segundos** e depois desliga automaticamente.
- Mensagens são exibidas no monitor serial indicando o status do sistema.

### 💬 Exemplo de Saída no Monitor Serial

Fósforo: Presente | Potássio: Presente | pH: 7.41 | Umidade: 88.5% -> Irrigação NÃO NECESSÁRIA

## Circuito

![image](https://github.com/user-attachments/assets/caefd3ef-d8ff-443e-bcc4-790b1308924d)

## Código

#include <DHT.h>  // Biblioteca para o sensor de umidade DHT22

// ========== DEFINIÇÃO DOS PINOS ==========
#define PINO_FOSFORO 32       // Botão verde (Fósforo presente se pressionado (NÚMERO 2))
#define PINO_POTASSIO 33      // Botão vermelho (Potássio presente se pressionado(NÚMERO 1))
#define PINO_PH 34            // Entrada analógica do sensor LDR (simula pH)
#define PINO_DHT 4            // Entrada digital do sensor DHT22 (umidade)
#define PINO_RELE 27          // Saída digital para acionar a bomba (relé)
#define PINO_LED 15           // LED indicador (acende quando a bomba estiver ligada)

// ========== INICIALIZAÇÃO DO SENSOR DHT22 ==========
#define DHTTYPE DHT22
DHT dht(PINO_DHT, DHTTYPE);   // Criação do objeto sensor DHT ligado ao pino 4

// ========== AUMATIZAÇÃO BOMBA ============
unsigned long tempoBombaLigada = 0;
bool bombaLigada = false;
const unsigned long DURACAO_IRRIGACAO = 30000; //30seg


void setup() {
  Serial.begin(115200);       // Inicializa o monitor serial
  dht.begin();                // Inicializa o sensor DHT22

  // Mensagem Boas-vindas
  Serial.println("=================================");
  Serial.println("  SISTEMA DE IRRIGAÇÃO AUTOMÁTICA");
  Serial.println("  Bem-vindo! ");
    Serial.println("=================================");
  delay(2000); // Espera 2 segundos antes de continuar


  // Configura os botões como entradas com resistor de pull-up interno
  // Isso faz com que o botão pressionado retorne LOW (0) e solto HIGH (1)
  pinMode(PINO_FOSFORO, INPUT_PULLUP);
  pinMode(PINO_POTASSIO, INPUT_PULLUP);

  // Define o pino do relé como saída (responsável por ligar/desligar a bomba)
  pinMode(PINO_RELE, OUTPUT);
  digitalWrite(PINO_RELE, LOW);  // Garante que a bomba inicie desligada
  pinMode(PINO_LED, OUTPUT);      //LED indicador
  digitalWrite(PINO_LED, LOW);   // Começa desligado 
}

void loop() {
  // ========== 1. LEITURA DOS BOTÕES ==========
  // Pressionado = LOW = Nutriente presente
  bool fosforoPresente = !digitalRead(PINO_FOSFORO);    // Inversão lógica
  bool potassioPresente = !digitalRead(PINO_POTASSIO);

  // ========== 2. LEITURA DO SENSOR DE pH (via LDR) ==========
  int valorLDR = analogRead(PINO_PH);  // Lê o valor do sensor LDR (0 a 4095)
  float ph = map(valorLDR, 0, 4095, 200, 1400) / 100.0;  
  // Converte o valor para uma faixa de pH entre 2.0 e 14.0

  // ========== 3. LEITURA DO SENSOR DE UMIDADE ==========
  float umidade = dht.readHumidity();  // Lê a umidade do sensor DHT22

  // Tratamento de falha na leitura do sensor DHT
  if (isnan(umidade)) {
    Serial.println("Erro: leitura inválida do DHT22. Usando valor padrão de 50%.");
    umidade = 50.0;  // Valor fictício para manter o funcionamento
  }

  // ========== 4. LÓGICA DA IRRIGAÇÃO ==========
  // A bomba será ligada se:
  // - o fósforo estiver ausente
  // - OU o potássio estiver ausente
  // - OU o pH estiver fora da faixa ideal (5.5 a 7.5)
  // - OU a umidade estiver abaixo de 50%

  // Verificação das condições
  bool umidadeBaixa = umidade < 50;
  bool fosforoAusente = !fosforoPresente;
  bool potassioAusente = !potassioPresente;
  bool phRuim = (ph < 5.5 || ph > 7.5);

  // Verifica se deve ligar a bomba
  bool ligarBomba = (umidadeBaixa || fosforoAusente || potassioAusente || phRuim);


  // ========== 5. CONTROLE DO RELÉ ==========
  digitalWrite(PINO_LED, ligarBomba ? HIGH : LOW);   // Acende o LED se a bomba estiver ligada

  // Se for necessário ligar a bomba e ela ainda não está ligada
  if (ligarBomba && !bombaLigada) {
  // Começa a irrigação agora
    digitalWrite(PINO_RELE, LOW);  // Liga bomba (nível baixo)
    tempoBombaLigada = millis();
    bombaLigada = true;
    Serial.println("⚠ Atenção: Problema(s) detectado(s)!");
    if (!fosforoPresente) Serial.println("Fósforo ausente");
    if (!potassioPresente) Serial.println("Potássio ausente");
    if (ph < 5.5 || ph > 7.5) Serial.println("pH fora do ideal");
    if (umidade < 50) Serial.println("Umidade baixa");
    Serial.println("---------------------");
    Serial.println("BOMBA LIGADA por 30 segundos...");
  }

  if (bombaLigada && millis() - tempoBombaLigada >= DURACAO_IRRIGACAO) {
    // Desliga após 30s
    digitalWrite(PINO_RELE, HIGH);
    bombaLigada = false;
    Serial.println("BOMBA DESLIGADA após 30 segundos.");
  }

  // ========== 6. IMPRESSÃO DOS DADOS NO MONITOR SERIAL ==========
  Serial.print("Fósforo: ");
  Serial.print(fosforoPresente ? "Presente" : "Ausente");

  Serial.print(" | Potássio: ");
  Serial.print(potassioPresente ? "Presente" : "Ausente");

  Serial.print(" | pH: ");
  Serial.print(ph, 2);  // Mostra 2 casas decimais

  Serial.print(" | Umidade: ");
  Serial.print(umidade, 1);  // Mostra 1 casa decimal
  Serial.print("%");

  Serial.print(" -> Irrigação ");
  Serial.println(ligarBomba ? "NECESSÁRIA" : "NÃO NECESSÁRIA");

  if (!ligarBomba) {
    Serial.println("✅ Condições ideais para a plantação. Irrigação desligada.");
  } else {
  Serial.println("⚠ Atenção: Problema(s) detectado(s)!");
  if (umidadeBaixa) Serial.println("Umidade baixa");
  if (fosforoAusente) Serial.println("Fósforo ausente");
  if (potassioAusente) Serial.println("Potássio ausente");
  if (phRuim) Serial.println("pH fora da faixa ideal (5.5 - 7.5)");
  }


  // Aguarda 3 segundos antes de repetir a leitura
  Serial.print("---------------------\n");
  delay(3000);
}



## 🖥️ Acesse o Projeto no Wokwi

🔗 [Clique aqui para ver no Wokwi](https://wokwi.com/projects/430519062599046145) 

