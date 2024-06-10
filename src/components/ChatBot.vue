<template>
  <div class="chatbot-container">
    <div class="chatbot-button-wrapper">
      <div class="chatbot-button" @click="toggleChat">
        <div class="chatbot-tooltip" :class="{ show: showTooltip && !showChat }">
          ¿Necesitas ayuda? <br> <strong>Converse con nosotros</strong>
        </div>
      </div>
    </div>

    <div v-if="showChat" class="chatbot" @mouseleave="keepChatOpen">
      <div class="chatbot-header">
        <h3>Hola Bienvenido 👋🏻</h3>
        <p>Ciudad seleccionada: {{ nameCity }}</p>
        <button class="close-btn" @click="toggleChat">×</button>
      </div>
      <div ref="chatBody" class="chatbot-body">
        <div v-for="(message, index) in messages" :key="index" :class="['message-container', message.sender, { 'bot-message-with-options': message.hasOptions }]">
          <div v-if="message.sender === 'typing'" class="typing-indicator">
            <span></span><span></span><span></span>
          </div>
          <div v-else-if="message.sender === 'twoOptions'" class="two-options-container">
            <button v-for="(option, idx) in message.options" :key="idx" class="two-option-button" @click="sendMessage(option)">
              <div class="two-option-content">{{ option }}</div>
            </button>
          </div>
          <div v-else-if="message.sender === 'groupedOptions'" class="options-grouped-container">
            <button v-for="(option, idx) in message.options" :key="idx" class="option-grouped-button" @click="sendMessage(option)">
              {{ option }}
            </button>
          </div>
          <div v-else class="message-bubble" v-html="message.text" @click="message.sender === 'option' && !message.disabled ? sendMessage(message.text) : null"></div>
        </div>
      </div>
      <div class="chatbot-footer">
        <input ref="userInput" v-model="userMessage" @input="handleInput" @keyup.enter="checkEnter" :disabled="!isInputEnabled" :placeholder="inputPlaceholder" />
        <ul v-if="autocompleteResults.length > 0" class="autocomplete-results">
          <li v-for="(result, index) in autocompleteResults" :key="index" @click="selectAutocompleteResult(result)" v-html="result.nombreProducto"></li>
        </ul>
      </div>
      <div v-if="showTemporaryMessage" class="temporary-message">
        Debe seleccionar un producto
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
      idCiudad: parseInt(sessionStorage.getItem("coddepto")) || 3,
      // idCiudad: 1,
      listCity: ["La Paz", "El Alto", "Cochabamba", "Santa Cruz", "Tarija", "Sucre", "Oruro", "Potosí"],
      listArea: ["Impresoras 3D", "Fotocopiadoras", "Sublimación", "Cortadora láser", "Computadoras", "Bioseguridad", "Impresoras", "Papel", "Novedades", "Otros"],
      listMenu: ["1. Ver tiendas en tu ciudad", "2. Horarios de atención", "3. Buscar un producto", "4. Área de computación", "5. Área 3D", "6. Liquidaciones"],
      listAreaSupport: ["5. Área 3D", "🙋🏻‍♂️ Máquinas láser", "4. Área de computación", "🙋🏻‍♂️ Sublimación", "🙋🏻‍♂️ Atención general"],
      listCitySupport: ["➡️ La Paz", "➡️ El Alto", "➡️ Cochabamba", "➡️ Santa Cruz", "➡️ Tarija", "➡️ Sucre", "➡️ Oruro", "➡️ Potosí"],
      listAreaCatalog: ["📰 Impresoras 3D", "📰 Fotocopiadoras", "📰 Sublimación", "📰 Cortadora láser", "📰 Computadoras", "📰 Bioseguridad", "📰 Impresoras", "📰 Papel", "📰 Novedades", "📰 Otros"],
      listaContactos: [
        { id_ciudad: 1, title: 'La Paz' },
        { id_ciudad: 2, title: 'El Alto' },
        { id_ciudad: 5, title: 'Cochabamba' },
        { id_ciudad: 3, title: 'Santa Cruz' },
        { id_ciudad: 8, title: 'Sucre' },
        { id_ciudad: 7, title: 'Tarija' },
        { id_ciudad: 6, title: 'Oruro' },
        { id_ciudad: 9, title: 'Potosi' }
      ],
      autocompleteResults: [],
      showTemporaryMessage: false,
      nameCity: ''
    };
  },
  methods: {
    toggleChat() {
      if (this.showChat) {
        this.clearMessages();
        this.clearAutocomplete();
        this.disableInput();
      }else {
        this.userMessage = ''; 
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
        }, 200); // Duración de la vibración
      }, 15000); // Intervalo de 15 segundos
    },
    handleOutsideClick(event) {
      if (!this.$el.contains(event.target) && this.showChat) {
        this.clearMessages();
        this.clearAutocomplete();
        this.disableInput();
        this.showChat = false;
        this.showTooltip = true;
      }
    },
    clearMessages() {
      this.messages = []; // Vaciar el array de mensajes
    },
    clearAutocomplete() {
      this.autocompleteResults = [];
    },
    disableInput() {
      this.isInputEnabled = false;
    },
    startConversation() {
      if (this.messages.length === 0) {
        console.log("Valor de idCiudad:", this.idCiudad);
        if (this.idCiudad === 0) {
          this.welcome();
          this.registerCity();
        } else {
          this.nameCity= this.getCityNameById(this.idCiudad)
          this.welcome();
          this.menu();
        }
      }
      this.scrollToBottom();
    },
    sendMessage(userMessage) {
      if (this.isInputEnabled) {
        this.showTemporaryMessage = true;
        setTimeout(() => {
          this.showTemporaryMessage = false;
        }, 2000);
        return;
      }

      if (userMessage.trim() !== '') {
        this.messages.push({ text: userMessage, sender: 'user' });
        // this.disablePreviousOptions();
        this.handleBotResponse(userMessage);
        this.userMessage = '';
        this.isInputEnabled = false;
        this.scrollToBottom();
      }
    },
    checkEnter(event) {
      if (event.key === 'Enter' && this.isInputEnabled) {
        this.showTemporaryMessage = true;
        setTimeout(() => {
          this.showTemporaryMessage = false;
        }, 2000);
      }
    },
    redirectWithMessage(url) {
    this.sendBotMessage("Serás redirigido en breve 😊");
    const delay = Math.random() * 1000 + 500; 
    setTimeout(() => {
      window.location.href = url;
    }, delay);
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
          case '1. ver tiendas en tu ciudad':
            this.addressSelected();
            break;
          case '6. liquidaciones':
            this.openPromotionsAndOffers();
            break;
          case '📖 ver catálogos':
            this.seeCatalogs();
            break;
          case '🙋🏻‍♂️ chatear con un asesor':
            this.chatWithAnAdvisor();
            break;
          case '3. buscar un producto':
            this.enableProductSearch();
            break;
          default:
            if (this.listArea.includes(userMessage)) {
              this.searchForArea(userMessage);
            // } else if (this.listaContactos.map(contact => contact.title).includes(userMessage)) {
            //   this.setCity(userMessage);
            //   this.addressForCity(this.nameCity);
            } else if (this.listAreaCatalog.includes(userMessage)) {
              this.catalog();
            } else if (this.listAreaSupport.includes(userMessage)) {
              this.whatsAppLinks(userMessage);
            } else if (this.listCitySupport.includes(userMessage)) {
              this.customerSupportByCity(userMessage);
            } else if (this.listCity.includes(userMessage)) {
              // this.addressForCity(userMessage);
              this.nameCity = userMessage;
              this.addressForCity(this.nameCity);
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
      this.messages.push({ text: message, sender: 'bot', hasOptions: true });
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
    async sendBotOptionsGrouped(options) {
      await this.delay(this.getRandomResponseTimeChatBotOptions());
      this.messages.push({ options: options, sender: 'groupedOptions' });
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
      this.sendBotMessage("Por favor, escribe el nombre del producto que estás buscando 👇🏻");
      this.isInputEnabled = true;
      this.inputPlaceholder = 'Escriba el nombre del producto...';
      this.$nextTick(() => {
        this.$refs.userInput.focus();
        this.scrollToBottom();
      });
    },
    async handleInput(event) {
      const query = event.target.value;
      if (query) {
        const results = await this.getDataAutocomplete(query);
        this.autocompleteResults = results;
      } else {
        this.autocompleteResults = [];
      }
    },
    // DESARROLLO 
    // ..............
    // método de busqueda de productos
    async getDataAutocomplete() {
      return new Promise(resolve => {
        setTimeout(() => {
          resolve([
            {
              "idCategoria": "2",
              "ordenLista": 1,
              "nombreCategoria": "IMPRESORAS",
              "idRubro": "7",
              "nombreRubro": "INSUMOS IMPRESORAS",
              "nombreProducto": "<b>TONER</b> para IMPRESORAS INSUMOS IMPRESORAS",
              "idMarca": "37",
              "query": "TONER",
              "tipoBusqueda": 1
            },
            {
              "idCategoria": "1",
              "ordenLista": 1,
              "nombreCategoria": "FOTOCOPIADORAS",
              "idRubro": "2",
              "nombreRubro": "REPUESTOS",
              "nombreProducto": "<b>TONER</b> para FOTOCOPIADORAS REPUESTOS",
              "idMarca": "7",
              "query": "TONER",
              "tipoBusqueda": 1
            },
            {
              "idCategoria": "1",
              "ordenLista": 1,
              "nombreCategoria": "FOTOCOPIADORAS",
              "idRubro": "46",
              "nombreRubro": "CHIPS P/TONER",
              "nombreProducto": "<b>TONER</b> para FOTOCOPIADORAS CHIPS P/TONER",
              "idMarca": "105",
              "query": "TONER",
              "tipoBusqueda": 1
            },
            {
              "idCategoria": "1",
              "ordenLista": 1,
              "nombreCategoria": "FOTOCOPIADORAS",
              "idRubro": "1",
              "nombreRubro": "INSUMOS",
              "nombreProducto": "<b>TONER</b> para FOTOCOPIADORAS INSUMOS",
              "idMarca": "17",
              "query": "TONER",
              "tipoBusqueda": 1
            }
          ]);
        }, 500);
      });
    },

    // método de enviar producto seleccionado
    sendDataAutocomplete(param) {
      // Mostrar el objeto en la consola
      console.log('Objeto seleccionado:', param);

      // Esperar 5 segundos y actualizar la página
      setTimeout(() => {
        console.log('recarga');
        // window.location.reload();
      }, 5000);
    },

    // ..............
    // DESARROLLO
    selectAutocompleteResult(result) {
      this.sendDataAutocomplete(result);
      this.autocompleteResults = [];
    },

    welcome() {
      this.sendBotMessage("🤖 Seré tu asistente virtual");
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
      this.sendBotMessageOptions("¿Te puedo ayudar con algo más? 😃");
      this.sendTwoOptions(["✅ Sí", "⛔ No"]);
    },
    goodbye() {
      this.sendBotMessage("🤖 Ha sido un placer atenderte");
      this.sendBotMessage("No olvides revisar nuestra página web y nuestros productos aquí 👋🏻");
    },
    errorMessage() {
      this.sendBotMessage("Lo siento, no entendí lo que dijiste 👀");
      this.menu();
    },
    productsAndPrices() {
      this.sendBotOptions(this.listArea);
    },
    getCityNameById(id) {
      console.log("Valor de id:", id);
      const city = this.listaContactos.find(contact => contact.id_ciudad === id);
      console.log("Valor de city:", city);
      return city ? city.title : 'error';
    },
    address() {
      this.sendBotMessage("¿En qué ciudad te encuentras? 👇🏻");
      this.sendBotOptions(this.listCity);
    },
    addressSelected() {
      this.addressForCity(this.nameCity)
    },
    openPromotionsAndOffers() {
      this.redirectWithMessage('https://www.savin.com.bo/liquidacion/');
      // window.location.href = 'https://www.savin.com.bo/liquidacion/';
    },
    seeCatalogs() {
      this.sendBotOptions(this.listAreaCatalog);
    },
    chatWithAnAdvisor() {
      this.sendBotMessage(`Muy bien ¿En qué área necesitas la atención al cliente? 🤔`);
      this.sendBotOptions(this.listAreaSupport);
    },
    searchForArea(area) {
      this.sendBotMessage(`¿Qué producto estás buscando en el área ${area}?`);
    },
    filterCities(nameCity) {
      return this.listCity.filter(city => city !== nameCity);
    },
    async addressForCity(city) {
      const addresses = {
        'La Paz': [
          {
            address: "https://maps.app.goo.gl/tNsAqrArK2NfGnM47",
            description: "Calle Loayza # 349, local 3 (Frente a la facultad de Derecho UMSA)",
            phone: "72030101"
          },
          {
            address: "https://maps.app.goo.gl/vnP2W9hk2oJZMSwx5",
            description: "Calle Zapata # 141 (frente Monoblock UMSA)",
            phone: "72030107"
          }
        ],
        'El Alto': [
          {
            address: "https://maps.app.goo.gl/vTUrQCpyNQmC24hH6",
            description: "Av. Juan Pablo II Edif. EI Ceibo Local A-15 (Final Autopista casi esq. Rene Dorado)",
            phone: "72029023"
          },
          {
            address: "https://maps.app.goo.gl/bagfMNGR4GSmpom9A",
            description: "Avenida Satélite # 668 (Cerca al Banco Mercantil Santa Cruz)",
            phone: "71543980"
          }
        ],
        'Cochabamba': [
          {
            address: "https://maps.app.goo.gl/6MfeLnrtaiAk9p6y9",
            description: "Calle Sucre # 882 (Casi esquina Oquendo)",
            phone: "72030102"
          }
        ],
        'Santa Cruz': [
          {
            address: "https://maps.app.goo.gl/1xw1r9zfBwv1pQJK6",
            description: "Avenida Centenario # 113 casi esquina Palermo (entre primer y segundo anillo)",
            phone: "72030103"
          }
        ],
        'Tarija': [
          {
            address: "https://maps.app.goo.gl/rHxKVwKALUQev44QA",
            description: "Calle Alejandro del Carpio # 258 entre Suipacha y Méndez (Zona Las Panosas)",
            phone: "72030105"
          }
        ],
        'Sucre': [
          {
            address: "https://maps.app.goo.gl/bcK8XhSmjCk9daXt7",
            description: "Calle Regimiento Campos # 174 Esquina Ricardo Andrade (Frente a la Facultad Técnica)",
            phone: "72030104"
          }
        ],
        'Oruro': [
          {
            address: "https://maps.app.goo.gl/5ARt9qRxZoRzadc89",
            description: "Calle Potosí # 5507 Esquina Montecinos (Diagonal al Col. Juan Misael Saracho)",
            phone: "72030106"
          }
        ],
        'Potosí': [
          {
            address: "https://maps.app.goo.gl/mzG5tcuqNpD9NcLDA",
            description: "Avenida Prado San Clemente # 29 entre las calles Camargo y 13 de Mayo",
            phone: "68868684"
          }
        ]
      };
      // await this.sendBotMessage(`Nuestra ubicación y contacto 👇🏻`);

      await this.sendBotMessage(`Tiendas disponibles en la ciudad de ${this.nameCity}`);


      for (const contact of addresses[city]) {
        await this.sendBotMessage(`
          <div class='address-container'>
            <span class='address-icon'>📌</span>${contact.description}<br>
            ${this.generateMapsButton(contact.address)}<br>
            <span class='phone-icon'>📲</span>
            <a href='https://api.whatsapp.com/send/?phone=591${contact.phone}&text&type=phone_number&app_absent=0' target='_blank'>${contact.phone}</a>
            <br>
            ${this.generateWhatsAppButton(contact.phone)}
            <br>
            <span class='search-icon'>🧐</span>
            Requieres un producto
            <br>
            ${this.generateSearchButton()}
          </div>
        `);
      }


      
      await this.sendBotMessage("También estamos en estas ciudades");
      await this.sendBotOptionsGrouped(this.filterCities(this.nameCity));

      this.menuPreEnd();
    },
    generateWhatsAppButton(phoneNumber) {
      const whatsappBaseUrl = "https://api.whatsapp.com/send/?phone=591";
      return `
        <a href='${whatsappBaseUrl}${phoneNumber}&text&type=phone_number&app_absent=0' target='_blank' class='whatsapp-button'>
          <img src='https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/whatsapp_logo_icon_229310.png?v=1717776398635' alt='WhatsApp' class='whatsapp-icon' />
          <span>Iniciar conversación</span>
        </a>
      `;
    },
    generateMapsButton(googleMapsUrl) {
      return `
        <a href='${googleMapsUrl}' target='_blank' class='maps-button'>
          <img src='https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/google-maps-icon-free-png.webp?v=1717799314299' alt='Google Maps' class='maps-icon' />
          <span>Abrir ubicación</span>
        </a>
      `;
    },
    generateSearchButton() {
      return `
        <a href="#" class="search-button search-button-custom">
          <img src="https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/binoculars_78394.png?v=1718033434301" alt="Buscar" class="search-icon" />
          <span>Buscar un producto</span>
        </a>
      `;
    },
    async whatsAppLinks(area) {
      const linksWhatsApp = {
        '4. Área de computación': [
          "Iván", "74040348"
        ],
        '5. Área 3D': [
          "Rodri", "68068883"
        ]
      };

      const advisor = linksWhatsApp[area];

      await this.sendBotMessage(`Nuestro especialista te atenderá con gusto 😊`);

      if (advisor) {
        await this.sendBotMessage(`
          <div class='address-container'>
            <span class='phone-icon'>📲</span> ${advisor[0]}: ${advisor[1]}
            <br>
            ${this.generateWhatsAppButton(advisor[1])}
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
    document.addEventListener('click', (event) => {
      if (event.target.closest('.search-button-custom')) {
        event.preventDefault();
        this.enableProductSearch();
      }
    });
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
  position: relative;
  width: 60px;
  height: 60px;
  background-color: #25D366;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
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
  z-index: 9999; 
  border-radius: 10px;
}

/* saludo del bot */

.chatbot-header {
  background-color: #25D366;
  color: white;
  padding: 8px;
  display: flex;
  flex-direction: column;
  align-items: flex-start; /* Alinea los elementos al inicio (izquierda) */
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
  height: auto; /* Ajusta la altura automáticamente */
  position: relative; /* Necesario para el posicionamiento absoluto del botón de cerrar */
}

.chatbot-header h3 {
  margin: 0;
  font-size: 27px;
  padding: 10px 0 5px 10px;
  display: flex;
  align-items: center;
  justify-content: flex-start; /* Alinea el título al inicio (izquierda) */
  width: 100%;
}

.chatbot-header p {
  margin: 0;
  padding-left: 10px;
  padding-bottom: 10px;
  display: flex;
  align-items: center;
  justify-content: flex-start; /* Alinea el texto de la ciudad al inicio (izquierda) */
  width: 100%;
  font-weight: bold;
}

.chatbot-header .close-btn {
  margin-left: auto; 
}



.chatbot-body {
  flex: 1;
  padding: 10px;
  overflow-y: auto;
  background-color: #fff;

  scrollbar-width: thin; /* Para navegadores Firefox */
  scrollbar-color: #ccc #fff; /* Para navegadores Firefox */
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


.bot-message-with-options {
  margin-bottom: 15px; /* Ajusta este valor según lo necesario */
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
  position: absolute;
  top: 10px;
  right: 10px;
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
  z-index: 9999;
}

.chatbot-tooltip {
  position: absolute;
  top: 48%;
  right: calc(100% + 10px); /* Ajustar la distancia entre el tooltip y el botón */
  transform: translateY(-50%);
  background-color: #002f5d;
  color: white;
  padding: 4px 17px;
  border-radius: 12px 12px 0px 12px;
  opacity: 0;
  transition: opacity 0.5s ease, transform 0.5s ease;
  white-space: nowrap; /* Para evitar que el texto se divida en varias líneas */
  z-index: 999; /* Asegurarse de que el tooltip esté por encima de otros elementos */
}

.chatbot-tooltip.show {
  opacity: 1;
  transform: translateY(-50%); /* Mantener la posición correcta */
}

/* Estilos para la lista de resultados de autocompletar */
.autocomplete-results {
  list-style: none;
  padding: 0;
  margin: 0;
  border: 1px solid #ccc;
  max-height: 150px; /* Cambia la altura según sea necesario */
  overflow-y: auto;
  background-color: #f6fcd0 ;
  position: absolute;
  width: 100%;
  z-index: 1001;
  bottom: 50px; /* Ajusta esta altura según el diseño de tu chatbot */
  box-shadow: 0 2px 5px rgba(0,0,0,0.1); /* Agrega sombra para mayor visibilidad */
  font-size: 12px; /* Cambia el tamaño del texto */
  color: black;
}

.autocomplete-results::-webkit-scrollbar {
  width: 8px; /* Cambia el ancho del scroll */
}

.autocomplete-results li {
  padding: 8px; /* Reduce el padding para que se vea más compacto */
  cursor: pointer;
}

.autocomplete-results li:hover {
  background-color: #cbe1fe  ;
  color: black;
}


/* nuevo botón para contactos whatsapp */

.whatsapp-button {
  display: inline-flex;
  align-items: center;
  background-color: #58d68d; /* Color verde más claro */
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  text-decoration: none;
  margin: 1px 0px 1px 9px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.whatsapp-button:hover {
  background-color: #45bf7a; /* Color verde más claro en el hover */
}

.whatsapp-icon {
  width: 20px;
  height: 20px;
  margin-right: 5px;
}

.whatsapp-button {
  color: white; /* Asegura que el texto esté en color blanco */
}

.whatsapp-button span {
  color: white; /* Asegura que el texto dentro del botón esté en color blanco */
}

.whatsapp-button a {
  color: white; /* Asegura que los enlaces dentro del botón estén en color blanco */
}

/* mensaje de advertencia */

.temporary-message {
  position: absolute;
  bottom: 60px; /* Ajusta según sea necesario para que aparezca justo encima del input */
  left: 50%;
  transform: translateX(-50%);
  background-color: #ff6666;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  z-index: 10002; /* Asegúrate de que esté por encima del resto del contenido */
  width: 200px;
  text-align: center; 
}

/* boton de maps */

.maps-button {
  display: inline-flex;
  align-items: center;
  background-color: #00aaff; /* Color celeste azulado */
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  text-decoration: none;
  margin: 1px 0px 1px 9px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.maps-button:hover {
  background-color: #0099dd; /* Color celeste azulado más oscuro en el hover */
}

.maps-icon {
  width: 20px;
  height: 20px;
  margin-right: 5px;
}

.maps-button {
  color: white; /* Asegura que el texto esté en color blanco */
}

.maps-button span {
  color: white; /* Asegura que el texto dentro del botón esté en color blanco */
}

.maps-button a {
  color: white; /* Asegura que los enlaces dentro del botón estén en color blanco */
}

/* botón de buscar en ciudades */

.search-button {
  display: inline-flex;
  align-items: center;
  background-color: #fddd25; /* Color amarillo */
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  text-decoration: none;
  /* margin-top: 10px; */
  margin: 1px 0px 1px 9px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

.search-button:hover {
  background-color: #ffbf00; /* Color amarillo más oscuro */
}

.search-icon {
  width: 20px;
  height: 20px;
  margin-right: 5px;
}

.search-button span {
  color: white;
}



/* CSS para el grupo de opciones */
.options-grouped-container {
  display: flex;
  flex-wrap: wrap;
  gap: 3px; /* Espacio entre botones */
  margin-top: 10px; /* Espacio superior */
  margin-bottom:  10px;
}

.option-grouped-button {
  background-color: #25D366;
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
  font-size: 15px; /* Tamaño de texto más pequeño */
  /* flex: 1 1 calc(33.333% - 10px);  */
  text-align: center;
  white-space: nowrap;
}

.option-grouped-button:hover {
  background-color: #34e47a;
}


/* scroll de los navegadores más delgados */

/* Para navegadores basados en Webkit (Chrome, Safari) */
.chatbot-body::-webkit-scrollbar {
  width: 6px; /* Ancho del scrollbar */
}

.chatbot-body::-webkit-scrollbar-track {
  background: #fff; /* Color del track del scrollbar */
}

.chatbot-body::-webkit-scrollbar-thumb {
  background-color: #ccc; /* Color del thumb del scrollbar */
  border-radius: 10px; /* Redondear los bordes del thumb */
}

.chatbot-body::-webkit-scrollbar-thumb:hover {
  background-color: #bbb; /* Color del thumb cuando se pasa el mouse por encima */
}

</style>
