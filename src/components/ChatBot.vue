<template>
  <div>
    <div class="chatbot-button" @click="toggleChat">
      <img src="../assets/whatsapp-icon.png" alt="Chat Icon" />
    </div>
    <div v-if="showChat" class="chatbot">
      <div class="chatbot-header">
        <h3>SaviBot</h3>
        <button class="close-btn" @click="toggleChat">×</button>
      </div>
      <div ref="chatBody" class="chatbot-body">
        <div v-for="(message, index) in messages" :key="index" :class="['message-container', message.sender]">
          <div class="message-bubble" @click="message.sender === 'option' && !message.disabled ? sendMessage(message.text) : null">
            <p>{{ message.text }}</p>
          </div>
        </div>
      </div>
      <div class="chatbot-footer">
        <input v-model="userMessage" @keyup.enter="sendMessage(userMessage)" :disabled="!isInputEnabled" placeholder="Type your message..." />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      messages: [],
      userMessage: '',
      showChat: false,
      botName: 'SaviBot',
      isInputEnabled: false,
      listCity: ["La Paz", "El Alto", "Cochabamba", "Santa Cruz", "Tarija", "Sucre", "Oruro", "Potosí"],
      listArea: ["Impresoras 3D", "Fotocopiadoras", "Sublimación", "Cortadora láser", "Computadoras", "Bioseguridad", "Impresoras", "Papel", "Novedades", "Otros"],
      listMenu: ["📦 Productos y precios", "🗺️ Dirección", "💰 Promociones y ofertas", "📖 Ver catálogos", "🙋🏻‍♂️ Chatear con un asesor"],
      listAreaSupport: ["🙋🏻‍♂️ Impresoras 3D", "🙋🏻‍♂️ Máquinas láser", "🙋🏻‍♂️ Computadoras", "🙋🏻‍♂️ Sublimación", "🙋🏻‍♂️ Atención general"],
      listCitySupport: ["➡️ La Paz", "➡️ El Alto", "➡️ Cochabamba", "➡️ Santa Cruz", "➡️ Tarija", "➡️ Sucre", "➡️ Oruro", "➡️ Potosí"],
      listAreaCatalog: ["📰 Impresoras 3D", "📰 Fotocopiadoras", "📰 Sublimación", "📰 Cortadora láser", "📰 Computadoras", "📰 Bioseguridad", "📰 Impresoras", "📰 Papel", "📰 Novedades", "📰 Otros"]
    };
  },
  methods: {
    toggleChat() {
      this.showChat = !this.showChat;
      if (this.showChat) {
        this.startConversation();
      }
    },
    startConversation() {
      this.sendBotOption("🤖 Empezar");
    },
    sendMessage(userMessage) {
      if (userMessage.trim() !== '') {
        this.messages.push({ text: userMessage, sender: 'user' });
        this.disablePreviousOptions();
        this.handleBotResponse(userMessage);
        this.userMessage = '';
      }
    },
    handleBotResponse(userMessage) {
      switch (userMessage.toLowerCase()) {
        case '🤖 empezar':
          this.welcome("🫡");
          break;
        case 'hola':
        case 'buen':
        case 'buenos días':
        case 'buenas tardes':
        case 'buenas noches':
          this.welcome("👋🏻");
          break;
        case '✅ menú':
          this.menu();
          break;
        case '📅 horarios':
          this.attentionSchedule();
          break;
        case '✅ sí':
          this.menu();
          break;
        case '⛔ no':
          this.goodbye();
          break;
        case '📦 productos y precios':
          this.productsAndPrices();
          break;
        case '🗺️ dirección':
          this.address();
          break;
        case '💰 promociones y ofertas':
          this.promotionsAndOffers();
          break;
        case '📖 ver catálogos':
          this.seeCatalogs();
          break;
        case '🙋🏻‍♂️ chatear con un asesor':
          this.chatWithAnAdvisor();
          break;
        default:
          if (this.listArea.includes(userMessage)) {
            this.searchForArea(userMessage);
          } else if (this.listCity.includes(userMessage)) {
            this.addressForCity(userMessage);
          } else if (this.listAreaCatalog.includes(userMessage)) {
            this.catalog();
          } else if (this.listAreaSupport.includes(userMessage)) {
            this.customerSupport(userMessage);
          } else if (this.listCitySupport.includes(userMessage)) {
            this.customerSupportByCity(userMessage);
          } else {
            this.errorMessage();
          }
          break;
      }
    },
    sendBotMessage(message) {
      this.messages.push({ text: message, sender: 'bot' });
      this.scrollToBottom();
    },
    sendBotOptions(options) {
      options.forEach(option => {
        this.messages.push({ text: option, sender: 'option', disabled: false });
      });
      this.scrollToBottom();
    },
    sendBotOption(option) {
      this.messages.push({ text: option, sender: 'option', disabled: false });
      this.scrollToBottom();
    },
    disablePreviousOptions() {
      this.messages.forEach(message => {
        if (message.sender === 'option') {
          message.disabled = true;
        }
      });
    },
    welcome(emoji) {
      this.sendBotMessage(`🤖 ¡Hola! Soy SaviBot ${emoji}`);
      this.sendBotMessage("Tú asistente virtual");
      this.sendBotOptions(["📅 Horarios de atención", "🔍 Buscar producto", "💻 Área de computación", "🧩 Área 3D", "💰 Liquidaciones"]);
    },
    menu() {
      this.sendBotMessage("¿En qué puedo ayudarte? 🤔");
      this.sendBotOptions(this.listMenu);
    },
    attentionSchedule() {
      this.sendBotMessage(
        "Nuestros horarios de atención en tiendas son:\n" +
        "✅ Lunes a Viernes\n" +
        "    ➡ de 8:30am a 12:30pm\n" +
        "    ➡ de 2:30pm a 7:00pm\n" +
        "✅ Sábados\n" +
        "    ➡ de 9:00am a 1:00pm"
      );
      this.menuPreEnd();
    },
    menuPreEnd() {
      this.sendBotOptions(["✅ Sí", "⛔ No"]);
    },
    goodbye() {
      this.sendBotMessage("🤖 Ha sido un placer atenderte");
      this.sendBotMessage("No olvides revisar nuestra página web y nuestros productos aquí 👇🏻");
      this.sendBotMessage("https://savin.com.bo/");
    },
    errorMessage() {
      this.sendBotMessage("Lo siento, no entendí lo que dijiste 👀");
      this.sendBotOption("🤖 Empezar");
    },
    productsAndPrices() {
      this.sendBotOptions(this.listArea);
    },
    address() {
      this.sendBotOptions(this.listCity);
    },
    promotionsAndOffers() {
      this.sendBotMessage("Mantente al pendiente de nuestras ofertas 😉");
      this.menuPreEnd();
    },
    seeCatalogs() {
      this.sendBotOptions(this.listAreaCatalog);
    },
    chatWithAnAdvisor() {
      this.sendBotMessage("Muy bien ¿En qué área necesitas la atención al cliente? 🤔");
      this.sendBotOptions(this.listAreaSupport);
    },
    searchForArea(area) {
      this.sendBotMessage(`¿Qué producto estás buscando en el área ${area}?`);
    },
    addressForCity(city) {
      const addresses = {
        'La Paz': [
          "🏢 Calle Loayza # 349, local 3 (Frente a la facultad de Derecho UMSA)\n📲 72030101\n📌 https://maps.app.goo.gl/tNsAqrArK2NfGnM47",
          "🏢 Calle Zapata # 141 (frente Monoblock UMSA)\n📲 72030107\n📌 https://maps.app.goo.gl/vnP2W9hk2oJZMSwx5",
          "🏢 Calle 2 de obrajes entre Av. Hernando Siles y Av. 14 de Septiembre (Frente Universidad Catolica)\n📲 71545171\n📌 https://maps.app.goo.gl/Vvw4BjAnpP6MFnwa8"
        ],
        'El Alto': [
          "🏢 Av. Juan Pablo II Edif. EI Ceibo Local A-15 (Final Autopista casi esq. Rene Dorado)\n📲 72029023\n📌 https://maps.app.goo.gl/vTUrQCpyNQmC24hH6",
          "🏢 Avenida Satélite # 668 (Cerca al Banco Mercantil Santa Cruz)\n📲 71543980\n📌 https://maps.app.goo.gl/bagfMNGR4GSmpom9A"
        ],
        'Cochabamba': [
          "🏢 Calle Sucre # 882 (Casi esquina Oquendo)\n📲 72030102\n📌 https://maps.app.goo.gl/6MfeLnrtaiAk9p6y9"
        ],
        'Santa Cruz': [
          "🏢 Avenida Centenario # 113 casi esquina Palermo (entre primer y segundo anillo)\n📲 72030103\n📌 https://maps.app.goo.gl/1xw1r9zfBwv1pQJK6"
        ],
        'Tarija': [
          "🏢 Calle Alejandro del Carpio # 258 entre Suipacha y Méndez (Zona Las Panosas)\n📲 72030105\n📌 https://maps.app.goo.gl/rHxKVwKALUQev44QA"
        ],
        'Sucre': [
          "🏢 Calle Regimiento Campos # 174 Esquina Ricardo Andrade (Frente a la Facultad Técnica)\n📲 72030104\n📌 https://maps.app.goo.gl/bcK8XhSmjCk9daXt7"
        ],
        'Oruro': [
          "🏢 Calle Potosí # 5507 Esquina Montecinos (Diagonal al Col. Juan Misael Saracho)\n📲 72030106\n📌 https://maps.app.goo.gl/5ARt9qRxZoRzadc89"
        ],
        'Potosí': [
          "🏢 Avenida Prado San Clemente # 29 entre las calles Camargo y 13 de Mayo\n📲 68868684\n📌 https://maps.app.goo.gl/mzG5tcuqNpD9NcLDA"
        ]
      };
      addresses[city].forEach(address => this.sendBotMessage(address));
      this.menuPreEnd();
    },
    catalog() {
      this.buildArea();
      this.menuPreEnd();
    },
    customerSupport(area) {
      const advisors = {
        "🙋🏻‍♂️ Impresoras 3D": ["Rodri", "📲 68068883"],
        "🙋🏻‍♂️ Máquinas láser": ["Rodri", "📲 68068883"],
        "🙋🏻‍♂️ Computadoras": ["Iván", "📲 74040348"],
        "🙋🏻‍♂️ Sublimación": ["Mafer", "📲 72507000"]
      };
      const advisor = advisors[area];
      if (advisor) {
        this.sendBotMessage(`Nuestro asesor ${advisor[0]} te atenderá con gusto 😊`);
        this.sendBotMessage(advisor[1]);
      } else if (area === "🙋🏻‍♂️ Atención general") {
        this.sendBotOptions(this.listCitySupport);
      }
      this.menuPreEnd();
    },
    customerSupportByCity(city) {
      const phones = {
        "➡️ La Paz": "📲 72030101 'Loayza' \n📲 72030107 'Zapata' \n📲 71545171 'Obrajes' ",
        "➡️ El Alto": "📲 72029023 'Ceibo' \n📲 71543980 'Satélite' ",
        "➡️ Cochabamba": "📲 72030102",
        "➡️ Santa Cruz": "📲 72030103",
        "➡️ Tarija": "📲 72030105",
        "➡️ Sucre": "📲 72030104",
        "➡️ Oruro": "📲 72030106",
        "➡️ Potosí": "📲 68868684"
      };
      this.sendBotMessage(`Nuestro asesor en la tienda de ${city} te atenderá con gusto. 😊`);
      this.sendBotMessage(phones[city]);
      this.menuPreEnd();
    },
    buildArea() {
      this.sendBotMessage("Lo siento, esta área está en construcción 🛠️");
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const chatBody = this.$refs.chatBody;
        chatBody.scrollTop = chatBody.scrollHeight;
      });
    }
  }
};
</script>

<style>
.chatbot {
  position: fixed;
  bottom: 80px;
  right: 20px;
  width: 300px;
  height: 400px;
  background: #f0f0f0;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  z-index: 1000;
  border-radius: 10px;
}

.chatbot-header {
  background-color: #25D366;
  color: white;
  padding: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
}

.chatbot-header h3 {
  margin: 0;
  font-size: 16px;
}

.chatbot-body {
  flex: 1;
  padding: 10px;
  overflow-y: auto;
}

.message-container {
  display: flex;
  margin: 3px 0;
  max-width: 70%;
}

.message-container.user {
  justify-content: flex-end;
}

.message-container.bot,
.message-container.option {
  justify-content: flex-start;
}

.message-bubble {
  display: flex;
  align-items: center;
  cursor: pointer;
  width: 100%;
  padding: 5px 8px;
  border-radius: 15px;
}

.message-bubble p {
  margin: 0;
  width: 100%;
}

.user .message-bubble {
  background: #DCF8C6;
  border-radius: 15px 15px 0 15px;
  text-align: right;
}

.bot .message-bubble {
  background: #ECECEC;
  border-radius: 15px 15px 15px 0;
  text-align: left;
}

.option .message-bubble {
  background: #25D366;
  color: white;
  border-radius: 15px;
  text-align: left;
}

.option:hover .message-bubble:not(:disabled) {
  background-color: #34e47a;
}

.option:disabled .message-bubble {
  background: #a4d3a2;
  cursor: not-allowed;
}

.chatbot-footer {
  padding: 10px;
  border-top: 1px solid #ccc;
  display: flex;
}

.chatbot-footer input {
  flex: 1;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 20px;
}

.chatbot-button {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 60px;
  height: 60px;
  background-color: #25D366;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  z-index: 1000;
}

.chatbot-button img {
  width: 35px;
  height: 35px;
}

.close-btn {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  color: white;
}
</style>
