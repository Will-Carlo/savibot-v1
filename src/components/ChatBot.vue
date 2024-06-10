<template>
  <div class="chatbot-container">
    <div class="chatbot-button-wrapper">
      <div class="chatbot-button" @mouseenter="openChat" @mouseleave="keepChatOpen" @click="toggleChat">
        <div class="chatbot-tooltip" :class="{ show: showTooltip }">
          ¿Necesitas ayuda? <br> Converse con nosotros
        </div>
      </div>
    </div>
    <div v-if="showChat" class="chatbot" @mouseleave="keepChatOpen">
      <div class="chatbot-header">
        <h3>Conversa con nosotros</h3>
        <button class="close-btn" @click="toggleChat">×</button>
      </div>
      <div ref="chatBody" class="chatbot-body">
        <div v-for="(message, index) in messages" :key="index" :class="['message-container', message.sender]">
          <div v-if="message.sender === 'typing'" class="typing-indicator">
            <span></span><span></span><span></span>
          </div>
          <div v-else-if="message.sender === 'twoOptions'" class="two-options-container">
            <button v-for="(option, idx) in message.options" :key="idx" class="two-option-button" @click="sendMessage(option)">
              <div class="two-option-content">{{ option }}</div>
            </button>
          </div>
          <div v-else class="message-bubble" v-html="message.text" @click="message.sender === 'option' && !message.disabled ? sendMessage(message.text) : null"></div>
        </div>
      </div>
      <div class="chatbot-footer">
        <input ref="userInput" v-model="userMessage" @keyup.enter="sendMessage(userMessage)" :disabled="!isInputEnabled" :placeholder="inputPlaceholder" />
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
      showTooltip: true,
      botName: 'Conversa con nosotros',
      isInputEnabled: false,
      inputPlaceholder: 'Buscar producto...',
      // idCiudad: parseInt(sessionStorage.getItem("coddepto")) || 0,
      idCiudad: 1,
      listCity: ["La Paz", "El Alto", "Cochabamba", "Santa Cruz", "Tarija", "Sucre", "Oruro", "Potosí"],
      listArea: ["Impresoras 3D", "Fotocopiadoras", "Sublimación", "Cortadora láser", "Computadoras", "Bioseguridad", "Impresoras", "Papel", "Novedades", "Otros"],
      listMenu: ["1. Ver tiendas en la ciudad", "2. Horarios de atención", "3. Buscar un Item", "4. Área de computación", "5. Área 3D", "6. Liquidaciones"],
      listAreaSupport: ["5. Área 3D", "🙋🏻‍♂️ Máquinas láser", "4. Área de computación", "🙋🏻‍♂️ Sublimación", "🙋🏻‍♂️ Atención general"],
      listCitySupport: ["➡️ La Paz", "➡️ El Alto", "➡️ Cochabamba", "➡️ Santa Cruz", "➡️ Tarija", "➡️ Sucre", "➡️ Oruro", "➡️ Potosí"],
      listAreaCatalog: ["📰 Impresoras 3D", "📰 Fotocopiadoras", "📰 Sublimación", "📰 Cortadora láser", "📰 Computadoras", "📰 Bioseguridad", "📰 Impresoras", "📰 Papel", "📰 Novedades", "📰 Otros"],
      listaContactos: [
        { id_ciudad: 1, title: 'Tienda La Paz' },
        { id_ciudad: 2, title: 'Tienda el Alto' },
        { id_ciudad: 5, title: 'Tienda Cochabamba' },
        { id_ciudad: 3, title: 'Tienda Santa Cruz' },
        { id_ciudad: 8, title: 'Tienda Sucre' },
        { id_ciudad: 7, title: 'Tienda Tarija' },
        { id_ciudad: 6, title: 'Tienda Oruro' },
        { id_ciudad: 9, title: 'Tienda Potosi' }
      ]
    };
  },
  methods: {
    toggleChat() {
      if (this.showChat) {
        this.clearMessages(); // Limpiar mensajes al cerrar el chat
      }
      this.showChat = !this.showChat;
      this.showTooltip = !this.showChat;
      if (this.showChat) {
        this.startConversation();
      }
    },
    openChat() {
      if (!this.showChat) {
        this.showChat = true;
        this.showTooltip = false;
        this.startConversation();
      }
    },
    keepChatOpen() {
      this.showChat = true;
    },
    startVibrationLoop() {
      this.showTooltip = true;

      setInterval(() => {
        const button = this.$el.querySelector('.chatbot-button');
        button.classList.add('vibrate');
        this.showTooltip == true ? true : false; // Mostrar el tooltip
        setTimeout(() => {
          button.classList.remove('vibrate');
          // this.showTooltip = false; // Ocultar el tooltip después de la vibración
        }, 200); // Duración de la vibración
      }, 15000); // Intervalo de 15 segundos
    },
    handleOutsideClick(event) {
      if (!this.$el.contains(event.target) && this.showChat) {
        this.clearMessages();
        this.showChat = false;
        this.showTooltip = true;
      }
    },
    clearMessages() {
      this.messages = []; // Vaciar el array de mensajes
    },
    startConversation() {
      if (this.messages.length === 0) {
        console.log("Valor de idCiudad:", this.idCiudad);
        if (this.idCiudad === 0) {
          this.welcome();
          this.registerCity();
        } else {
          this.welcome();
          this.menu();
        }
      }
      this.scrollToBottom();
    },
    sendMessage(userMessage) {
      if (userMessage.trim() !== '') {
        this.messages.push({ text: userMessage, sender: 'user' });
        this.disablePreviousOptions();
        this.handleBotResponse(userMessage);
        this.userMessage = '';
        this.isInputEnabled = false;
        this.scrollToBottom();
      }
    },
    handleBotResponse(userMessage) {
      this.showTypingIndicator();
      setTimeout(() => {
        this.hideTypingIndicator();
        switch (userMessage.toLowerCase()) {
          case '🤖 empezar':
            this.welcome();
            break;
          case 'hola':
          case 'buen':
          case 'buenos días':
          case 'buenas tardes':
          case 'buenas noches':
            this.welcome();
            break;
          case '✅ menú':
            this.menu();
            break;
          case '2. horarios de atención':
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
          case '1. ver tiendas en la ciudad':
            this.address();
            break;
          case '6. Liquidaciones':
            this.openPromotionsAndOffers();
            break;
          case '📖 ver catálogos':
            this.seeCatalogs();
            break;
          case '🙋🏻‍♂️ chatear con un asesor':
            this.chatWithAnAdvisor();
            break;
          case '3. buscar un item':
            this.enableProductSearch();
            break;
          default:
            if (this.listArea.includes(userMessage)) {
              this.searchForArea(userMessage);
            } else if (this.listaContactos.map(contact => contact.title).includes(userMessage)) {
              this.setCity(userMessage);
            } else if (this.listAreaCatalog.includes(userMessage)) {
              this.catalog();
            } else if (this.listAreaSupport.includes(userMessage)) {
              this.whatsAppLinks(userMessage);
            } else if (this.listCitySupport.includes(userMessage)) {
              this.customerSupportByCity(userMessage);
            } else if (this.listCity.includes(userMessage)) {
              this.addressForCity(userMessage);
            } else {
              this.errorMessage();
            }
            break;
        }
      }, this.getRandomResponseTime());
    },
    showTypingIndicator() {
      this.messages.push({ text: '...', sender: 'typing' });
      this.scrollToBottom();
    },

    hideTypingIndicator() {
      this.messages = this.messages.filter(message => message.sender !== 'typing');
    },

    getRandomResponseTime() {
      return Math.floor(Math.random() * 2000) + 1000; // Entre 1 y 3 segundos
    },
    getRandomResponseTimeChat() {
      return Math.floor(Math.random() * 300) + 300; // Entre 0.3 y 0.6 segundos
    },
    getRandomResponseTimeChatBotMessage() {
      return Math.floor(Math.random() * 300) + 700; // Entre 0.7 y 1.0 segundos
    },
    getRandomResponseTimeChatBotOptions() {
      return Math.floor(Math.random() * 300) + 1100; // Entre 1.1 y 1.4 segundos
    },
    async sendBotMessage(message) {
      await this.delay(this.getRandomResponseTimeChat());
      this.messages.push({ text: message, sender: 'bot' });
      this.scrollToBottom();
    },
    async sendBotMessageOptions(message) {
      await this.delay(this.getRandomResponseTimeChatBotMessage());
      this.messages.push({ text: message, sender: 'bot' });
      this.scrollToBottom();
    },
    async sendBotOptions(options) {
      await this.delay(this.getRandomResponseTimeChatBotOptions());
      for (const option of options) {
        this.messages.push({ text: option, sender: 'option', disabled: false });
        this.scrollToBottom();
      }
    },
    async sendBotOption(option) {
      await this.delay(this.getRandomResponseTimeChatBotOptions());
      this.messages.push({ text: option, sender: 'option', disabled: false });
      this.scrollToBottom();
    },
    async sendTwoOptions(options) {
      if (options.length !== 2) {
        console.error("sendTwoOptions requires exactly two options.");
        return;
      }
      await this.delay(this.getRandomResponseTimeChatBotOptions());
      this.messages.push({ options: options, sender: 'twoOptions' });
      this.scrollToBottom();
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    disablePreviousOptions() {
      this.messages.forEach(message => {
        if (message.sender === 'option') {
          message.disabled = true;
        }
      });
    },
    enableProductSearch() {
      this.sendBotMessage("Por favor, escribe el nombre del Item que estás buscando 👇🏻");

      this.isInputEnabled = true;
      this.inputPlaceholder = 'Escriba el nombre del producto...';
      this.$nextTick(() => {
        this.$refs.userInput.focus();
        this.scrollToBottom();
      });      
    },
    welcome() {
      this.sendBotMessage("🤖 ¡Hola! Soy tu asistente 👋🏻");
      // this.sendBotMessage("Tú asistente virtual");
    },
    registerCity(){
      this.sendBotMessageOptions("¿En qué ciudad te encuentras? 👇🏻");
      this.sendBotOptions(this.listaContactos.map(contact => contact.title));
    },
    menu() {
      this.sendBotMessageOptions("¿En qué podemos ayudarte? 👇🏻");
      this.sendBotOptions(this.listMenu);
    },
    preMenu(){
      this.sendBotMessage("¿En qué puedo ayudarte hoy? 😃");
      this.menu();
    },
    attentionSchedule() {
      this.sendBotMessage(
        `Nuestros horarios de atención en tiendas son:<br>
        <div style="display: flex; align-items: center;">
          <span style="color: #4CAF50;">✅</span> Lunes a Viernes
        </div>
        <div style="padding-left: 1em;">
          <div style="display: flex; align-items: center;">
            <span style="color: #4CAF50;">➡</span> de 8:30am a 12:30pm
          </div>
          <div style="display: flex; align-items: center;">
            <span style="color: #4CAF50;">➡</span> de 2:30pm a 7:00pm
          </div>
        </div>
        <div style="display: flex; align-items: center;">
          <span style="color: #4CAF50;">✅</span> Sábados
        </div>
        <div style="padding-left: 1em;">
          <div style="display: flex; align-items: center;">
            <span style="color: #4CAF50;">➡</span> de 9:00am a 1:00pm
          </div>
        </div>`
      );
      this.menuPreEnd();
    },
    menuPreEnd() {
      this.sendBotMessageOptions("¿Te puedo ayudar con algo más? 🤔");
      this.sendTwoOptions(["✅ Sí", "⛔ No"]);
    },
    goodbye() {
      this.sendBotMessage("🤖 Ha sido un placer atenderte");
      this.sendBotMessage("No olvides revisar nuestra página web y nuestros productos aquí 👋🏻");
      // this.sendBotMessage("https://savin.com.bo/");
    },
    errorMessage() {
      this.sendBotMessage("Lo siento, no entendí lo que dijiste 👀");
      // this.sendBotOption("🤖 Empezar");
      this.menu();
    },
    productsAndPrices() {
      this.sendBotOptions(this.listArea);
    },
    address() {
      this.sendBotMessageOptions("¿En qué ciudad te encuentras? 👇🏻");
      this.sendBotOptions(this.listCity);
    },
    openPromotionsAndOffers() {
      window.location.href = 'https://www.savin.com.bo/liquidacion/';
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
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/tNsAqrArK2NfGnM47' target='_blank'>Calle Loayza # 349, local 3 (Frente a la facultad de Derecho UMSA)</a>",
            phone: "72030101"
          },
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/vnP2W9hk2oJZMSwx5' target='_blank'>Calle Zapata # 141 (frente Monoblock UMSA)</a>",
            phone: "72030107"
          }
        ],
        'El Alto': [
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/vTUrQCpyNQmC24hH6' target='_blank'>Av. Juan Pablo II Edif. EI Ceibo Local A-15 (Final Autopista casi esq. Rene Dorado)</a>",
            phone: "72029023"
          },
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/bagfMNGR4GSmpom9A' target='_blank'>Avenida Satélite # 668 (Cerca al Banco Mercantil Santa Cruz)</a>",
            phone: "71543980"
          }
        ],
        'Cochabamba': [
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/6MfeLnrtaiAk9p6y9' target='_blank'>Calle Sucre # 882 (Casi esquina Oquendo)</a>",
            phone: "72030102"
          }
        ],
        'Santa Cruz': [
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/1xw1r9zfBwv1pQJK6' target='_blank'>Avenida Centenario # 113 casi esquina Palermo (entre primer y segundo anillo)</a>",
            phone: "72030103"
          }
        ],
        'Tarija': [
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/rHxKVwKALUQev44QA' target='_blank'>Calle Alejandro del Carpio # 258 entre Suipacha y Méndez (Zona Las Panosas)</a>",
            phone: "72030105"
          }
        ],
        'Sucre': [
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/bcK8XhSmjCk9daXt7' target='_blank'>Calle Regimiento Campos # 174 Esquina Ricardo Andrade (Frente a la Facultad Técnica)</a>",
            phone: "72030104"
          }
        ],
        'Oruro': [
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/5ARt9qRxZoRzadc89' target='_blank'>Calle Potosí # 5507 Esquina Montecinos (Diagonal al Col. Juan Misael Saracho)</a>",
            phone: "72030106"
          }
        ],
        'Potosí': [
          {
            address: "<span class='address-icon'>🏢</span><a href='https://maps.app.goo.gl/mzG5tcuqNpD9NcLDA' target='_blank'>Avenida Prado San Clemente # 29 entre las calles Camargo y 13 de Mayo</a>",
            phone: "68868684"
          }
        ]
      };

      const whatsappBaseUrl = "https://api.whatsapp.com/send/?phone=591";

      addresses[city].forEach(contact => {
        this.sendBotMessage(`
          <div class='address-container'>
            ${contact.address}<br>
            <span class='phone-icon'>📲</span>
            <a href='${whatsappBaseUrl}${contact.phone}&text&type=phone_number&app_absent=0' target='_blank'>${contact.phone}</a>
          </div>
        `);
      });

      this.menuPreEnd();
    },



    async whatsAppLinks(area) {
      const linksWhatsApp = {
        '4. Área de computación': [
          "Iván", "<a href='https://api.whatsapp.com/send/?phone=59174040348&text&type=phone_number&app_absent=0' target='_blank'>74040348</a>"
        ],
        '5. Área 3D': [
          "Rodri", "<a href='https://api.whatsapp.com/send/?phone=59168068883&text&type=phone_number&app_absent=0' target='_blank'>68068883</a>"
        ]
      };

      const advisor = linksWhatsApp[area];
  
      // Enviar el nombre del asesor
      await this.sendBotMessage(`Nuestro especialista te atenderá con gusto 😊`);

      if (advisor) {
        // Enviar el enlace de WhatsApp
        await this.sendBotMessage(`
          <div class='address-container'>
            <span class='phone-icon'>📲</span> ${advisor[1]}
          </div>
        `);
      }

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
    },
    setCity(city) {
      const selectedContact = this.listaContactos.find(contact => contact.title === city);
      this.idCiudad = selectedContact.id_ciudad;
      sessionStorage.setItem("coddepto", this.idCiudad);
      this.menu();
      this.scrollToBottom();
    }
  },
  mounted() {
    document.addEventListener('click', this.handleOutsideClick);
    this.startVibrationLoop();
  },
  beforeDestroy() {
    document.removeEventListener('click', this.handleOutsideClick);
  }
};
</script>

<style>

@keyframes vibrate {
  0%, 100% { transform: translateX(0); }
  20%, 60% { transform: translateX(-1px); }
  40%, 80% { transform: translateX(1px); }
}

@keyframes expand {
  0%, 100% {
    box-shadow: 0 0 0 0 rgba(37, 211, 102, 0.7);
  }
  50% {
    box-shadow: 0 0 15px 15px rgba(37, 211, 102, 0);
  }
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
  background-image: url('https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/bot-head.png?v=1717688111538');
  background-size: 69%;
  background-repeat: no-repeat;
  background-position: center;
  animation: expand 2s infinite;
}

.chatbot-button:hover {
  animation: expand 2s infinite, vibrate 0.2s;
}
.vibrate {
  animation: vibrate 0.2s;
}



.bot-head {
  position: relative;
  width: 60%;
  height: 60%;
  background: white;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.bot-eyes {
  position: absolute;
  top: 25%;
  left: 50%;
  transform: translateX(-50%);
  width: 60%;
  display: flex;
  justify-content: space-between;
}

.eye {
  width: 20%;
  height: 20%;
  background: black;
  border-radius: 50%;
}

.bot-mouth {
  position: absolute;
  bottom: 20%;
  left: 50%;
  transform: translateX(-50%);
  width: 20%;
  height: 10%;
  background: black;
  border-radius: 50%;
}

.bot-antenna {
  position: absolute;
  top: -10%;
  left: 50%;
  transform: translateX(-50%);
  width: 10%;
  height: 10%;
  background: black;
  border-radius: 50%;
}


.chatbot {
  position: fixed;
  bottom: 80px;
  right: 20px;
  width: 320px;
  height: 500px;
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
  background-color: #fff;
}

.message-container {
  display: flex;
  margin: 3px 0;
  max-width: 100%;
}

.message-container.user {
  justify-content: flex-end;
}

.message-container.bot,
.message-container.option {
  justify-content: flex-start;
}

.message-bubble {
  display: inline-block;
  max-width: 80%;
  word-wrap: break-word;
  padding: 5px 8px;
  border-radius: 5px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  opacity: 0; /* Oculto inicialmente */
  transform: translateY(20px); /* Desplazado hacia abajo inicialmente */
  animation: slideUp 0.5s forwards; /* Animación que lo hace visible y lo desplaza hacia arriba */
}

@keyframes slideUp {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.message-bubble p {
  margin: 0;
  /* width: 100%; */
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
  border-radius: 5px;
  text-align: left;
  
  cursor: pointer;
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

  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.address-container {
  font-size: 14px;
  line-height: 1.5;
}

.address-icon, .phone-icon {
  font-size: 18px;
}

.address-container a {
  color: inherit;
  text-decoration: none;
}

.address-container a:hover {
  text-decoration: underline;
}

.close-btn {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  color: white;
}


.typing-indicator {
  display: flex;
  align-items: center;
  justify-content: center;
}

.typing-indicator span {
  background-color: #34e47a;
  height: 10px;
  width: 10px;
  border-radius: 50%;
  display: inline-block;
  margin: 0 2px;
  animation: typing 1.2s infinite;
}

.typing-indicator span:nth-child(2) {
  animation-delay: 0.2s;
}

.typing-indicator span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes typing {
  0% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
  100% { transform: translateY(0); }
}

/* dos opciones  */
.two-options-container {
  display: flex;
  justify-content: center;
  margin-top: 0px;
}

.two-option-button {
  flex: 0 1 auto;
  margin: 5px;
  padding: 5px 15px;
  background-color: #25D366;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.two-option-button:disabled {
  background-color: #a4d3a2;
  cursor: not-allowed;
}

.two-option-content {
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  white-space: nowrap; /* Para evitar que el texto se divida en varias líneas */
}


/* tool tip */

.chatbot-container {
  position: relative;
}

.chatbot-button-wrapper {
  position: fixed;
  bottom: 20px;
  right: 20px;
}

.chatbot-button {
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
  background-image: url('https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/bot-head.png?v=1717688111538');
  background-size: 69%;
  background-repeat: no-repeat;
  background-position: center;
  animation: expand 2s infinite;
}

.chatbot-button:hover {
  animation: expand 2s infinite, vibrate 0.2s;
}

.vibrate {
  animation: vibrate 0.2s;
}

.chatbot-tooltip {
  position: absolute;
  top: 10%;
  left: calc(100% - 230px); /* Ajustar para que el tooltip comience justo al lado derecho del botón */
  transform: translateY(-50%);
  background-color: #002f5d;
  color: white;
  padding: 5px 10px;
  border-radius: 5px; 
  opacity: 0;
  transform: translateX(20px); /* Desplazamiento inicial para el efecto de transición */
  transition: opacity 0.5s ease, transform 0.5s ease;
  white-space: nowrap; /* Para evitar que el texto se divida en varias líneas */
  z-index: 999; /* Asegurarse de que el tooltip esté por encima de otros elementos */
}

.chatbot-tooltip.show {
  opacity: 1;
  transform: translateX(0); /* Desplazamiento final para el efecto de transición */
}


</style>
