</p>
<h1 align="center">
    <img alt="NextLevelWeek" title="#NextLevelWeek" src="./assets/logo.jpg" />
</h1>

<h4 align="center"> 
	🚧 🙂 Boxpsycho  🚀 🚧
</h4>

<p align="center">
 Problema •
 Solução •
 Protótipo • 
 Configuração e execução • 
 Autores • 
 Código-fonte •
 Licença 
</p>


## 🏥 Problema de saúde abordado

O uso incorreto de remédios, especialmente os controlados, é um problema comum que representa riscos à saúde.  A estimativa da Organização Mundial da Saúde (OMS) é que cerca de 50% dos usuários de medicamentos não os utilizam corretamente, o que pode resultar em intoxicação, dependência, falência e até mesmo morte por overdose. A falta de monitoramento preciso e exato por parte dos médicos contribui para essa situação, pois eles têm dificuldade em garantir que os pacientes sigam corretamente as prescrições.


## 🦾 Solução proposta

Boxpsycho é uma caixa de remédios inteligente para o controle e monitoramento dos médicos e seus pacientes sobre suas medicações, a fim de melhorar a eficácia do tratamento e evitar uso incorreto ou ineficaz de farmacos. A caixa de remédios registra os medicamentos e horários de cada dose, emitindo alertas sonoros nos horários programados, além de registrar o uso do paciente permitindo o acompanhamento proposto.

Componentes da BoxPsycho:
- Um botão para abrir o slot de medicação;
- Uma tela de LCD para mensagens ao usuário;
- Um buzzer para sinalização do horário do medicamento;
- Um slot para armazenamento dos medicamentos;
- Uma pequena memória interna para registros;
- Um servo motor para controlar a abertura e fechamento do slot;
- Bateria para operação independente;
- RTC (Real Time Clock) para gerenciar horários.

Para pacientes:
  - Abertura e fechamento de tampa do slot;
  - Contagem de comprimidos disponíveis;
  - Lembrete de próximo da proxima dosagem.
  
Para o médicos:

- Configuração de disponibilide, horário das dosagens.



## 🚧 Protótipo

Essa trata-se de uma versão simplifica do projeto por conta de limitações de conhecimento e do Thinkercad em si, deixando alguma funcionalidades para uma etapa futura. Dessa maneira, a simulação permite:

#### Para médicos:
- Configuração de disponibilide e horário das dosagens.

#### Para pacientes:
  - Abertura e fechamento de tampa do slot;
  - Contagem de comprimidos disponíveis;
  - Lembrete de próximo da proxima dosagem;
  
#### Funcionalidades Futuras:

- Instalação de RTC e buzzer, para lembrete da medicação com buzzer na hora da medicação e restrição de doses antes do horario;
- Troca de dados com a Plataforma de Python;
- Registro detalhado de hora e data das doses tomada pelo paciente;

## ⚙️ Configuração e execução

#### Acesso ao Tinkercard:
- [Link para o Projeto]([https://www.tinkercad.com/things/jHI4InoDjWK-global-solution-edge-computing-and-computer-systems-20232?sharecode=O05syKseQvdTsvK07henlzyK9hOBdwnQZyQfaquQSi4](https://www.tinkercad.com/things/hos0b9wgmZh-global-solution-edge-computing-and-computer-systems-20232?sharecode=oz73z-uILemk9dt-PtvVSgkts3tkVyKIeZIEH7BPf2E))

#### Carregar dados de dosagem:
- Sem a conexão com Python, as informações de dosagem precisam ser colocadas diretamente nas variaveis: proximoHorario, intervaloMedicacao e medicamentosDisponiveis

#### Utilização da Boxpsycho:
- É algo simples, o botão é a única interação que o paciente tem com a caixa ela serve para abrir e fechar o compartimento, com isso o sistemas por conta propria registra, bloqueia e permite o uso.

## ⚙️ Código fonte
```cpp
#include <Servo.h>
#include <LiquidCrystal.h>

const int buttonPin = 2;
const int servoPin = 3;
Servo servo;
LiquidCrystal lcd(9, 8, 7, 6, 5, 4);


//Variaveis para loop independente
float ultimoTempo = 0;
const int intervalo = 5000;

// Configurações que virão da plataforma em Python
// Elas guiaram o paciente e posterior o RTC e buzzer
int proximoHorario = 17;
int intervaloMedicacao = 8;
int medicamentosDisponiveis = 3;

//
int usoNormal = 0;
int tentativasDeUso = 0;

void setup() {
  
  Serial.begin(9600);
  pinMode(buttonPin, INPUT);
  servo.attach(servoPin);
  servo.write(0); 
  lcd.begin(16, 2);

  // Inicia o lcd com a mensagem da próximo medicação 
  lcd.clear();
  lcd.print("Proxima dose");
  lcd.setCursor(0, 1);
  lcd.print("as ");
  lcd.print(proximoHorario);
  lcd.print(":00");
}

void loop() {
  
  
  // Loop independente para mostrar os uso e bloqueios do sistema
  if (millis() - ultimoTempo >= intervalo) {
    ultimoTempo = millis();
    
	Serial.print("Uso normal: ");
  	Serial.println(usoNormal);
  
  	Serial.print("Uso bloqueado: ");
  	Serial.println(tentativasDeUso);
  }
  
  // Variavel rastreia o estado do servo do motor(a tampa do slot)
  // O estado static evita o que incongruencia software/hardware ocorram entre loops
  static bool isOpen = false;
  
  // Verificação do clique do botão dupla com debounce evitando os possíveis ruídos elétricos, erros e falsas leituras
  // Como as boas práticas indicam
  if (digitalRead(buttonPin) == HIGH) {
    delay(75); 
    if (digitalRead(buttonPin) == HIGH) {
      
      // Verificação se o sistema está aberto ou fechado
      if (isOpen) {
        // Caso do sistema aberto
        
        // Fechar a tampa do slot
        servo.write(0); 
        isOpen = false;

        // Verificação se há medicamentos disponíveis para o próximo horário
        if (medicamentosDisponiveis > 0) {
          
          // Atualiza o horário para a próxima medicação
          proximoHorario = (proximoHorario + intervaloMedicacao) % 24;

          // Atualiza mensagem no LCD
          lcd.clear();
          lcd.print("Proxima dose");
          lcd.setCursor(0, 1);
          lcd.print("as ");
          lcd.print(proximoHorario);
          lcd.print(":00");
          
        } else {
          
          // Exibe mensagem no LCD se não houver medicamentos disponíveis
          lcd.clear();
          lcd.print("Sem comprimidos!");
          lcd.setCursor(0, 1);
          lcd.print("Consulte medico2.");
          
          
        }
      } else {
        // Caso do sistema fechado
        
        // Verificação se há medicamentos disponíveis
        if (medicamentosDisponiveis > 0) {
          
          // Abertura da tampa, atualização dos medicamentos, e estado da tampa
          servo.write(90);
          isOpen = true;
          medicamentosDisponiveis--;

          // Exibição mensagem no LCD de disponibilidade de compridos
          lcd.clear();
          lcd.print("Remedio tomado!");
          lcd.setCursor(0, 1);
          lcd.print("Disponiveis: ");
          lcd.print(medicamentosDisponiveis);
          usoNormal++;
          
        } else {
          
          // Exibição mensagem no LCD de falta de remédios
          lcd.clear();
          lcd.print("Sem comprimidos!");
          lcd.setCursor(0, 1);
          lcd.print("Consulte medico.");
          tentativasDeUso++;
        }
      }
      delay(75); // Debounce
    }
  }
}

```

## 👨🏽‍🏭👨🏽‍🔧 Autores
<p align="Center">
<a href="https://github.com/JorgeBooz00">
 <img style="border-radius: 50%;" src="https://avatars.githubusercontent.com/u/107008455?v=4" width="100px;" alt=""/>
 <br />
 <sub><b>Jorge Booz</b></sub> <sub><br><b>RM 552700</b></sub></a> <a href="https://github.com/JorgeBooz00" title="Booz">🚀</a>
 <br /> <a href="https://github.com/MateusTibao">
 <img style="border-radius: 50%;" src="https://media-gru1-1.cdn.whatsapp.net/v/t61.24694-24/358163362_6408788222509379_3686466471442376588_n.jpg?ccb=11-4&oh=01_AdRHAL-Mn925oHq0bWlu-bD6wtgKxPHinF3lPbH1xD7YyQ&oe=656A695F&_nc_sid=e6ed6c&_nc_cat=108" width="100px;" alt=""/>
 <br />
 <sub><b>Mateus Tibao</b></sub> <sub><br><b>RM 553267</b></sub></a> <a href="https://github.com/MateusTibao" title="Tibao">🚀</a>
 <br />



## 📝 Licença

Este projeto esta sobe a licença [MIT](./LICENSE).

Feito com muita vontade de aprender por [Jorge Booz](https://www.linkedin.com/in/jorge-booz-4038a2213/) e [Mateus Tibão](https://www.linkedin.com/in/mateustibao/) 👋🏽 

---
