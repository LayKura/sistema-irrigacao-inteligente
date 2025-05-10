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

## 🖥️ Acesse o Projeto no Wokwi

🔗 [Clique aqui para ver no Wokwi](https://wokwi.com/projects/430519062599046145) 
