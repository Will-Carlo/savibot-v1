

<template>
  <div class="chatbot-container">
    <div class="chatbot-button-wrapper">
      <div class="chatbot-button" @click="handleChatbotButtonClick">
        <div class="chatbot-tooltip" :class="{ show: showTooltip }">
          <!-- ¿Necesitas ayuda? <br> <strong>Conversa con nosotros</strong> -->
          <img v-if="showTooltip" src="https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/flecha%20AZUL.png?v=1719610915886" alt="Flecha" id="flechaAnimada">
        </div>
      </div>
    </div>

    <div v-if="showChat" class="chatbot" @mouseleave="keepChatOpen">
      <div class="chatbot-header">
        <h3>Hola Bienvenido 👋🏻</h3>
        <p>Ciudad seleccionada: <strong style="font-size: 21px;">{{ nameCity }}</strong></p>
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
          <div v-else-if="message.sender === 'groupedOptionsYellow'" class="options-grouped-container-yellow">
            <button v-for="(option, idx) in message.options" :key="idx" class="option-grouped-button-yellow" @click="sendMessage(option)">
              {{ option }}
            </button>
          </div>
          <div v-else-if="message.sender === 'option-principal-menu'" class="option-principal-menu-button" @click="sendMessage(message.text)">
            {{ message.text }}
          </div>
          <div v-else-if="message.sender === 'simple-option'" class="simple-option-container" v-html="message.text"></div>
          <div v-else class="message-bubble" v-html="message.text" @click="message.sender === 'option' && !message.disabled ? sendMessage(message.text) : null"></div>
        </div>
      </div>
      <div class="menu-toggle" @click="toggleMenu">{{ menuButtonText }}</div>
      <div v-if="showMenu" class="menu-options show">
        <button v-for="(option, idx) in listMenu" :key="idx" class="menu-option-button" @click="handleMenuOption(option)">
          {{ option }}
        </button>
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
import axios from 'axios';
// import moment from 'moment-timezone';


export default {
  data() {
    return {
      messages: [],
      userMessage: '',
      showChat: false,
      showTooltip: true,
      botName: 'Conversa con nosotros',
      isInputEnabled: true,
      inputPlaceholder: 'Buscar producto...',
      idCiudad: parseInt(sessionStorage.getItem("coddepto")) || 1,
      // idCiudad: 1,
      listCity: ["La Paz", "El Alto", "Cochabamba", "Santa Cruz", "Tarija", "Sucre", "Oruro", "Potosí"],
      listArea: ["Impresoras 3D", "Fotocopiadoras", "Sublimación", "Cortadora láser", "Computadoras", "Bioseguridad", "Impresoras", "Papel", "Novedades", "Otros"],
      listMenu: [
                  "VER TIENDAS EN TU CIUDAD 🏬",
                  "HORARIOS DE ATENCIÓN 🕒",
                  "ÁREA DE COMPUTACIÓN 💻",
                  "ÁREA 3D 🚀",
                  "ÁREA CORTADORAS LÁSER 🔦"
                ],
      listAreaSupport: ["ÁREA DE COMPUTACIÓN 💻", "ÁREA 3D 🚀", "ÁREA CORTADORAS LÁSER 🔦", "🙋🏻‍♂️ Sublimación", "🙋🏻‍♂️ Atención general"],
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
      nameCity: '',
      optionsArea: {
        "computación": {
          "CPU": "https://www.savin.com.bo/computacion/marcas.html?tkn=MTIsMjcsNzAsVEVDTk9MT0dJQS1DT01QVVRBQ0lPTixDUFU=",
          "ACCESORIOS": "https://www.savin.com.bo/computacion/marcasns.html?tkn=MTQsMCw3MSxURUNOT0xPR0lBLUNPTVBVVEFDSU9OLEFDQ0VTT1JJT1M=",
          "ZONA GAMER": "https://www.savin.com.bo/computacion/marcasns.html?tkn=NTMsMCw3MSxURUNOT0xPR0lBLUNPTVBVVEFDSU9OLFpPTkEgR0FNRVI=",
          "REDES": "https://www.savin.com.bo/computacion/marcasns.html?tkn=NjIsMCw3MSxURUNOT0xPR0lBLUNPTVBVVEFDSU9OLFJFREVT",
          "CELULARES": "https://www.savin.com.bo/computacion/marcasns.html?tkn=NjMsMCw3MSxURUNOT0xPR0lBLUNPTVBVVEFDSU9OLENFTFVMQVJFUw==",
          "LAPTOP": "https://www.savin.com.bo/computacion/marcas.html?tkn=NzQsMjcsNzAsVEVDTk9MT0dJQS1DT01QVVRBQ0lPTixMQVBUT1A=",
          "ALL IN ONE": "https://www.savin.com.bo/computacion/marcas.html?tkn=NzcsMjcsNzAsVEVDTk9MT0dJQS1DT01QVVRBQ0lPTixBTEwgSU4gT05F"
        },
        "cortadores láser": {
          "MÁQUINAS LASER": "https://www.savin.com.bo/cortadoras-laser/marcasns.html?tkn=NzgsMCw3MSxDT1JUQURPUkFTIExBU0VSLE1BUVVJTkFTIExBU0VS",
          "REPUESTOS": "https://www.savin.com.bo/cortadoras-laser/marcasns.html?tkn=NzksMCw3MSxDT1JUQURPUkFTIExBU0VSLFJFUFVFU1RPUyA=",
          "ACCESORIOS LÁSER": "https://www.savin.com.bo/cortadoras-laser/marcasns.html?tkn=ODIsMCw3MSxDT1JUQURPUkFTIExBU0VSLEFDQ0VTT1JJT1MgTEFTRVI="
        },
        "3d": {
          "INSUMOS 3D": "https://www.savin.com.bo/savin3d/modelodetalle.html?tkn=MTAsSU1QUkVTT1JBUyAzRCxJTlNVTU9TIDNELG51bGwsTW9uIEp1biAxMCAyMDI0IDE4OjE0OjI4IEdNVC0wNDAwIChCb2xpdmlhIFRpbWUp",
          "MAQ. IMPRESORAS 3D": "https://www.savin.com.bo/savin3d/marcasns.html?tkn=MTEsMCw3MSxJTVBSRVNPUkFTIDNELE1BUS4gSU1QUkVTT1JBUyAzRCxNb24gSnVuIDEwIDIwMjQgMTg6MTQ6MzcgR01ULTA0MDAgKEJvbGl2aWEgVGltZSk=",
          "REPUESTOS 3D": "https://www.savin.com.bo/savin3d/marcasns.html?tkn=NzIsMCw3MSxJTVBSRVNPUkFTIDNELFJFUFVFU1RPUyAzRCxNb24gSnVuIDEwIDIwMjQgMTg6MTQ6NDMgR01ULTA0MDAgKEJvbGl2aWEgVGltZSk=",
          "SCANER 3D": "https://www.savin.com.bo/savin3d/marcasns.html?tkn=NzYsMCw3MSxJTVBSRVNPUkFTIDNELFNDQU5FUiAzRCxNb24gSnVuIDEwIDIwMjQgMTg6MTQ6NDkgR01ULTA0MDAgKEJvbGl2aWEgVGltZSk=",
          "ACCESORIOS 3D": "https://www.savin.com.bo/savin3d/marcasns.html?tkn=ODMsMCw3MSxJTVBSRVNPUkFTIDNELFDQ0VTT1JJT1MgM0QsTW9uIEp1biAxMCAyMDI0IDE4OjE1OjAyIEdNVC0wNDAwIChCb2xpdmlhIFRpbWUp"
        }
      },
      areaSelected: '',
      showMenu: false,
      menuButtonText: '⬆ SELECCIONAR UNA OPCIÓN',
      hasTooltipBeenShown: false,
      // VARIABLES PARA EL ENVIO DE DATOS DE SESION
      apiFechaInicioSesion: null,
      apiFechaFinalSesion: null,
      apiDuracionSesion: null,
      apiUrlReferente: document.referrer,
      apiBrowserUserAgent: navigator.userAgent,
      apiTipoDeDispositivo: this.detectDeviceType(),
      apiIpAddress: null,  // Asumiremos que obtendremos la IP desde la API
      apiPais: null,       // Asumiremos que obtendremos el país desde la API
      apiCiudad: this.idCiudad,
      apiBuscarTiendas: 0,
      apiHorarios: 0,
      apiAreaComp: 0,
      apiArea3d: 0,
      apiAreaCort: 0
    };
  },
  methods: {
    getCurrentDateTimeLaPaz() {
      const options = { timeZone: 'America/La_Paz', hour12: false };
      const now = new Date();
      const dateTimeLaPaz = now.toLocaleString('en-US', options);
      
      // Convertir a objeto Date en el huso horario de La Paz
      const dateLaPaz = new Date(dateTimeLaPaz);

      // Obtener componentes de fecha y hora
      const year = dateLaPaz.getFullYear();
      const month = String(dateLaPaz.getMonth() + 1).padStart(2, '0'); // Meses comienzan en 0
      const day = String(dateLaPaz.getDate()).padStart(2, '0');
      const hours = String(dateLaPaz.getHours()).padStart(2, '0');
      const minutes = String(dateLaPaz.getMinutes()).padStart(2, '0');
      const seconds = String(dateLaPaz.getSeconds()).padStart(2, '0');

      // Formatear en "YYYY-MM-DD HH:MM:SS"
      const formattedDateTime = `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;

      return formattedDateTime;
    },
    async toggleMenu() {
      const menuElement = this.$el.querySelector('.menu-options');
      if (this.showMenu) {
        if (menuElement) {  // Verifica que el elemento exista
          menuElement.classList.remove('show');
        }
        await this.delay(300); // Espera 300ms antes de ocultar el menú
        this.showMenu = false;
      } else {
        this.showMenu = true;
        await this.delay(10); // Espera 10ms para permitir la transición
        this.$nextTick(() => {
          if (menuElement) {  // Verifica que el elemento exista
            menuElement.classList.add('show');
          }
        });
      }
      this.menuButtonText = this.showMenu ? '⬇ SELECCIONAR UNA OPCIÓN' : '⬆ SELECCIONAR UNA OPCIÓN';
    },
    handleMenuOption(option) {
      
      // this.showMenu = false;
      this.toggleMenu();
      this.sendMessage(option);

      switch (option) {
        case 'VER TIENDAS EN TU CIUDAD 🏬':
          this.apiBuscarTiendas++;
          break;
        case 'HORARIOS DE ATENCIÓN 🕒':
          this.apiHorarios++;
          break;
        case 'ÁREA DE COMPUTACIÓN 💻':
          this.apiAreaComp++;
          break;
        case 'ÁREA 3D 🚀':
          this.apiArea3d++;
          break;
        case 'ÁREA CORTADORAS LÁSER 🔦':
          this.apiAreaCort++;
          break;
      }

    },
    async toggleChat() {
      const chatbotElement = this.$el.querySelector('.chatbot');
      if (this.showChat) {
        if (chatbotElement) {  // Verifica que el elemento exista
          chatbotElement.classList.add('close');
        }
        await this.delay(300); // Espera 300ms antes de ocultar el chat
        this.showChat = false;
        // this.apiFechaFinalSesion = moment().tz("America/La_Paz").toISOString();
        // this.apiFechaFinalSesion = moment().tz("America/La_Paz").subtract(4, 'hours').toISOString();
        this.apiFechaFinalSesion = this.getCurrentDateTimeLaPaz();
        console.log("apiFechaFinalSesion", this.apiFechaFinalSesion);
        // this.apiDuracionSesion = Math.floor((moment(this.apiFechaFinalSesion).tz("America/La_Paz").toDate() - moment(this.apiFechaInicioSesion).tz("America/La_Paz").toDate()) / 1000);
        this.apiDuracionSesion = Math.floor((new Date(this.apiFechaFinalSesion) - new Date(this.apiFechaInicioSesion)) / 1000);
        this.sendSessionData();
        console.log("cerrando el chatbot");
      } else {
        this.showChat = true;
        this.hasTooltipBeenShown = true; // Actualiza la variable
        this.$nextTick(() => {
          if (chatbotElement) {  // Verifica que el elemento exista
            chatbotElement.classList.remove('close');
          }
        });
        // this.apiFechaInicioSesion = moment().tz("America/La_Paz").toISOString();
        // this.apiFechaInicioSesion = moment().tz("America/La_Paz").subtract(4, 'hours').toISOString();
        this.apiFechaInicioSesion = this.getCurrentDateTimeLaPaz();
        console.log("apiFechaInicioSesion", this.apiFechaInicioSesion);
        this.startConversation();
        console.log("abriendo el chatbot");
      }
      
      this.showTooltip = false;
      this.toggleMenu();
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
      if (this.hasTooltipBeenShown) return; // Si el tooltip ya ha sido mostrado, no hacer nada

      // this.showTooltip = true;
      setInterval(() => {
        const button = this.$el.querySelector('.chatbot-button');
        button.classList.add('vibrate');
        // this.showTooltip = true; // Mostrar el tooltip
        setTimeout(() => {
          button.classList.remove('vibrate');
        }, 200); // Duración de la vibración
      }, 15000); // Intervalo de 15 segundos
    },

    handleOutsideClick(event) {
      if (!this.$el.contains(event.target) && this.showChat && this.showMenu) {
        // this.clearMessages();
        this.clearAutocomplete();
        this.disableInput();
        this.showChat = false;
        // this.showTooltip = true;
      }
    },
    clearMessages() {
      this.messages = []; // Vaciar el array de mensajes
    },
    clearAutocomplete() {
      this.autocompleteResults = [];
    },
    disableInput() {
      // this.isInputEnabled = false;
    },
    startConversation() {
      this.loadMessagesFromSession();
      if (this.messages.length === 0) {
        // console.log("Valor de idCiudad:", this.idCiudad);
        if (this.idCiudad === 0) {
          this.welcome();
          this.registerCity();
        } else {
          this.nameCity = this.getCityNameById(this.idCiudad);
          console.log(this.nameCity, this.idCiudad);
          // this.welcome();
          // this.menu();
          this.sendBotMessage(`¿En qué podemos ayudarte? 👀`);
        }
      }
      this.scrollToBottom();
    },
    sendMessage(userMessage) {
      if (this.isInputEnabled) {
        this.disableInput();
        this.messages.push({ text: userMessage, sender: 'user' });
        this.handleBotResponse(userMessage);
        this.autocompleteResults = [];  
        this.userMessage = '';

        // this.showTemporaryMessage = true;
        // setTimeout(() => {
        //   this.showTemporaryMessage = false;
        // }, 2000);
        this.scrollToBottom();
        return;
      }

      if (userMessage.trim() !== '') {
        this.messages.push({ text: userMessage, sender: 'user' });
        // this.disablePreviousOptions();
        this.handleBotResponse(userMessage);                                                                                                                                                                                                                                                                                                                                                                                                                                 
        this.userMessage = '';
        this.disableInput();
        this.scrollToBottom();
      }

      // console.log(this.messages);
    },
    checkEnter(event) {
      // if (event.key === 'Enter' && this.isInputEnabled) {
      if (event.key === 'Enter') {
        this.sendDataGeneral(this.userMessage, this.idCiudad);
        this.sendMessage(this.userMessage);
        this.sendBotMessage(`Buscando producto... 👀`);

        // this.showTemporaryMessage = true;
        // setTimeout(() => {
        //   this.showTemporaryMessage = false;
        // }, 2000);
      }
    },
    handleChatbotButtonClick(event) {
    // Verifica si el clic no fue en la flecha animada
      if (event.target.id !== 'flechaAnimada') {
        this.toggleChat();
        if (!this.apiFechaInicioSesion) {
          this.apiFechaInicioSesion = new Date().toISOString();
        }
      }
    },


    saveMessagesToSession() {
      const messagesJSON = JSON.stringify(this.messages);
      // console.log(messagesJSON);
      sessionStorage.setItem('listaChatbot', messagesJSON);
    },
    loadMessagesFromSession() {
      const savedMessages = sessionStorage.getItem('listaChatbot');
      if (savedMessages) {
        this.messages = JSON.parse(savedMessages);
        
        this.nameCity = this.getCityNameById(this.idCiudad);
        // this.scrollToBottom();
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
        // switch (userMessage) {
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
          case 'horarios de atención 🕒':
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
          case 'ver tiendas en tu ciudad 🏬':
            this.addressSelected();
            break;
          case '7. liquidaciones':
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
            } else if (this.areaSelected && this.getOptionsList(this.areaSelected).includes(userMessage)) {
              this.openAreaSelected(this.getLinkRubroForArea(this.areaSelected, userMessage))
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
    async sendBotOptionsPrincipalMenu(options) {
      await this.delay(this.getRandomResponseTimeChatBotOptions());
      for (const option of options) {
        this.messages.push({ text: option, sender: 'option-principal-menu', disabled: false });
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
    async sendBotOptionsGroupedYellow(options) {
      await this.delay(this.getRandomResponseTimeChatBotOptions());
      this.messages.push({ options: options, sender: 'groupedOptionsYellow' });
      this.scrollToBottom();
    },
    async sendBotMessageSimpleOption(buttonHtml) {
      await this.delay(this.getRandomResponseTimeChat());
      this.messages.push({ text: buttonHtml, sender: 'simple-option' });
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
      // this.inputPlaceholder = 'Escriba el nombre del producto...';
      this.$nextTick(() => {
        this.$refs.userInput.focus();
        this.scrollToBottom();
      });
    },
    async handleInput(event) {
      const query = event.target.value;
      if (query) {
        const results = await this.getDataAutocomplete(query, this.idCiudad);
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

    // método que busca items cuando el usuario preciona [enter]
    sendDataGeneral(userInput, cityId) {
      console.log(`Texto del usuario: ${userInput}, Ciudad: ${cityId}`);
    },

  // ..............
    // DESARROLLO
    selectAutocompleteResult(result) {
      this.sendDataAutocomplete(result, this.idCiudad);
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
      // this.sendBotOptions(this.listMenu);
      this.sendBotOptionsPrincipalMenu(this.listMenu);
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
      // this.menuPreEnd();
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
      // console.log("Valor de id:", id);
      const city = this.listaContactos.find(contact => contact.id_ciudad === id);
      // console.log("Valor de city:", city);
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
    openAreaSelected(link) {
      this.redirectWithMessage(link);
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
            address: "https://maps.app.goo.gl/Jaeyw86zfhZybJag6",
            description: "Calle Loayza # 349, local 3 <b>(Frente a la facultad de Derecho UMSA)</b>",
            phone: "72030101"
          },
          {
            address: "https://maps.app.goo.gl/S3H7hnDYPxdsdkzj7",
            description: "Calle Zapata # 141 <b>(frente Monoblock UMSA)</b>",
            phone: "72030107"
          }
        ],
        'El Alto': [
          {
            address: "https://maps.app.goo.gl/8aic1TmbHBULVWTV8",
            description: "Av. Juan Pablo II Edif. EI Ceibo Local A-15 <b>(Final Autopista casi esq. Rene Dorado)</b>",
            phone: "72029023"
          },
          {
            address: "https://maps.app.goo.gl/3b2wvZZ3rFan5nXLA",
            description: "Avenida Satélite # 668 <b>(Cerca al Banco Mercantil Santa Cruz)</b>",
            phone: "71543980"
          }
        ],
        'Cochabamba': [
          {
            address: "https://maps.app.goo.gl/tCEMbEkSAc5EKHKD6",
            description: "Calle Sucre # 882 <b>(Casi esquina Oquendo)</b>",
            phone: "72030102"
          }
        ],
        'Santa Cruz': [
          {
            address: "https://maps.app.goo.gl/RLxbcr5wNug8RAgU6",
            description: "Avenida Centenario # 113 casi esquina Palermo <b>(entre primer y segundo anillo)</b>",
            phone: "72030103"
          }
        ],
        'Tarija': [
          {
            address: "https://maps.app.goo.gl/SENXLifigjXYbzLs8",
            description: "Calle Alejandro del Carpio # 258 entre Suipacha y Méndez <b>(Zona Las Panosas)</b>",
            phone: "72030105"
          }
        ],
        'Sucre': [
          {
            address: "https://maps.app.goo.gl/WLtqbiQt3ETXsHwG9",
            description: "Calle Regimiento Campos # 174 Esquina Ricardo Andrade <b>(Frente a la Facultad Técnica)</b>",
            phone: "72030104"
          }
        ],
        'Oruro': [
          {
            address: "https://maps.app.goo.gl/CZPpa5dKrd7tf8QF7",
            description: "Calle Potosí # 5507 Esquina Montecinos <b>(Diagonal al Col. Juan Misael Saracho)</b>",
            phone: "72030106"
          }
        ],
        'Potosí': [
          {
            address: "https://maps.app.goo.gl/cnXZSHvwshURQ4Uq7",
            description: "Avenida Prado San Clemente # 29 <b>(Entre las calles Camargo y 13 de Mayo)</b>",
            phone: "68868684"
          }
        ]
      };
      // await this.sendBotMessage(`Nuestra ubicación y contacto 👇🏻`);

      await this.sendBotMessage(`Tiendas en <b>${this.nameCity}</b>`);


      for (const contact of addresses[city]) {
        await this.sendBotMessage(`
          <div class='address-container'>
            <span class='address-icon'>📌</span>${contact.description}, 
            ${this.generateMapsButton(contact.address)}
            <br>
            ${this.generateWhatsAppButton(contact.phone)}
          </div>
        `);
      }


      
      await this.sendBotMessage("También estamos en otras ciudades");
      await this.sendBotOptionsGrouped(this.filterCities(this.nameCity));

      // this.menuPreEnd();
    },
    generateWhatsAppButton(phoneNumber) {
      const whatsappBaseUrl = "https://api.whatsapp.com/send/?phone=591";
      return `
        <a href='${whatsappBaseUrl}${phoneNumber}&text&type=phone_number&app_absent=0' target='_blank' class='button-full-width whatsapp-button'>
          <img src='https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/whatsapp_logo_icon_229310.png?v=1717776398635' alt='WhatsApp' class='whatsapp-icon' />
          <span>WhatsApp <b>(Sólo texto)</b></span>
        </a>
      `;
    },
    generateMapsButton(googleMapsUrl) {
      return `
        <a href='${googleMapsUrl}' target='_blank' class='button-full-width maps-button'>
          <img src='https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/google-maps-icon-free-png.webp?v=1717799314299' alt='Google Maps' class='maps-icon' />
          <span>Abrir ubicación</span>
        </a>
      `;
    },
    generateSearchButton() {
      return `
        <a href="#" class="button-full-width search-button search-button-custom">
          <img src="https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/binoculars_78394.png?v=1718033434301" alt="Buscar" class="search-icon" />
          <span>Buscar un producto</span>
        </a>
      `;
    },
    generatePDFDownloadButton(link, buttonText) {
    return `
        <a href="${link}" target="_blank" download class="pdf-download-button">
          <img src="https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/free-download-1767976-1502312.webp?v=1718216961750" alt="Download" class="pdf-download-icon" />
          <span>${buttonText}</span>
        </a>
      `;
    },  
    getOptionsList(text) {
      this.areaSelected = text;
      if (this.optionsArea[text]) {
        return Object.keys(this.optionsArea[text]);
      }
      return [];
    },
    getLinkRubroForArea(area, rubro) {
      if (this.optionsArea[area] && this.optionsArea[area][rubro]) {
        return this.optionsArea[area][rubro];
      }
      return null;
    },

    async whatsAppLinks(area) {
      
      
      const linksWhatsApp = {
        'ÁREA DE COMPUTACIÓN 💻': [
          "Iván", "74040348", "computación", "https://www.savin.com.bo/catalogo/catalogo_junio_computacion.pdf"
        ],
        'ÁREA 3D 🚀': [
          "Rodri", "68068883", "3d", "https://www.savin.com.bo/catalogo/catalogo_junio_impresora3d.pdf"
        ],
        'ÁREA CORTADORAS LÁSER 🔦': [
          "Rodri", "68068883", "cortadores láser", "https://www.savin.com.bo/catalogo/catalogo_junio_cortadora_grabadora.pdf"
          ]
      };
          
      const advisor = linksWhatsApp[area];
      
      await this.sendBotMessage(`Rubros disponibles del área de ${advisor[2]}`);
      await this.sendBotOptionsGroupedYellow(this.getOptionsList(advisor[2]));

      await this.sendBotMessageSimpleOption(this.generatePDFDownloadButton(advisor[3], 'Descarga nuestro CATÁLOGO DEL MES'));
      
      if (advisor) {
        await this.sendBotMessage(`
          <div class='address-container'>
            ¿Desea una atención personalizada?                                                                           <br>  
            ${this.generateWhatsAppButton(advisor[1])}
          </div>
        `);
      }

            
      // this.menuPreEnd();
    },

    catalog() {
      this.buildArea();
      // this.menuPreEnd();
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
      // this.menuPreEnd();
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
      // this.menuPreEnd();

    },
    buildArea() {
      this.sendBotMessage("Lo siento, esta área está en construcción 🛠️");
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const chatBody = this.$refs.chatBody;
        chatBody.scrollTop = chatBody.scrollHeight;
      });
      this.saveMessagesToSession();
    },
    setCity(city) {
      const selectedContact = this.listaContactos.find(contact => contact.title === city);
      this.idCiudad = selectedContact.id_ciudad;
      sessionStorage.setItem("coddepto", this.idCiudad);
      this.menu();
      this.scrollToBottom();
    },

   // Método para detectar el tipo de dispositivo
   detectDeviceType() {
      const ua = navigator.userAgent;
      if (/mobile/i.test(ua)) return 'mobile';
      if (/tablet/i.test(ua)) return 'tablet';
      return 'desktop';
    },

     // Método para enviar datos de la sesión a la API
    async sendSessionData() {
      // Obtén la IP y el país del usuario si es necesario
      if (!this.apiIpAddress || !this.apiPais) {
        const res = await axios.get('https://api.ipify.org?format=json');
        this.apiIpAddress = res.data.ip;
        // Aquí podrías usar otro servicio para obtener el país basado en la IP
      }

      this.apiCiudad = this.idCiudad;

      const sessionData = {
        apiFechaInicioSesion: this.apiFechaInicioSesion,
        apiFechaFinalSesion: this.apiFechaFinalSesion,
        apiDuracionSesion: this.apiDuracionSesion,
        apiUrlReferente: this.apiUrlReferente,
        // apiBrowserUserAgent: this.apiBrowserUserAgent,
        apiBrowserUserAgent: escapeSlashes(this.apiBrowserUserAgent),
        apiTipoDeDispositivo: this.apiTipoDeDispositivo,
        apiIpAddress: this.apiIpAddress,
        apiPais: this.apiPais,
        apiCiudad: this.apiCiudad,
        apiBuscarTiendas: this.apiBuscarTiendas,
        apiHorarios: this.apiHorarios,
        apiAreaComp: this.apiAreaComp,
        apiArea3d: this.apiArea3d,
        apiAreaCort: this.apiAreaCort,
      };

      // console.log('Enviando datos de sesión a la API:', sessionData);

      // Enviar los datos a la API
      try {
        // const response = await axios.post('https://www.savin.com.bo/savinphp/'+ 'chatbot-registrar-sesion/', sessionData, {
        const response = await axios.post('http://localhost:8082/'+ 'chatbot-registrar-sesion/', sessionData, {
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
          },
        }); 
        console.log('Respuesta de la API:', response.data);
      } catch (error) {
        console.error('Error al enviar los datos de sesión a la API:', error);
      }
    },
  // Método para detectar el cierre de la ventana o navegación fuera de la página
    handleBeforeUnload() {
      if (this.showChat) {
        this.apiFechaFinalSesion = new Date().toISOString();
        this.apiDuracionSesion = Math.floor((new Date(this.apiFechaFinalSesion) - new Date(this.apiFechaInicioSesion)) / 1000);
        this.sendSessionData();
      }
    },
    async getUserCountry() {
      try {
        const response = await axios.get('https://natural-happy-chord.glitch.me/geoip');
        this.apiPais = response.data.country;
        console.log('País del usuario:', this.apiPais);
      } catch (error) {
        console.error('Error al obtener el país del usuario:', error);
      }
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
    
    // Detectar el cierre de la ventana o navegación fuera de la página
    window.addEventListener('beforeunload', this.handleBeforeUnload);

    // Llamar al método para obtener el país del usuario
    // this.getUserCountry();
  },
  beforeDestroy() {
    document.removeEventListener('click', this.handleOutsideClick);
    window.removeEventListener('beforeunload', this.handleBeforeUnload);
  }
};

function escapeSlashes(str) {
      return str.replace(/\//g, '-'); // Reemplaza '/' con '-'
}
</script>




<style>

@keyframes vibrate {
  0%, 100% { transform: translateX(0); }
  20%, 60% { transform: translateX(-1px); }
  40%, 80% { transform: translateX(1px); }
}

@keyframes expand {
  0%, 100% {
    box-shadow: 0 0 0 0 rgba(37, 49, 211, 0.7);
  }
  50% {
    box-shadow: 0 0 15px 15px rgba(37, 211, 102, 0);
  }
}

.chatbot-button {
  position: relative;
  width: 60px;
  height: 60px;
  background-color: #0000ff;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  background-image: url('https://cdn.glitch.global/fb88d943-304b-4312-be00-868b389c37cf/mensaje.png?v=1718742560046');
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
  bottom: 15px;
  right: 15px;
  width: 400px;
  /* height: 700px; */
  height: 78%;
  background: #f0f0f0;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  z-index: 9999; 
  border-radius: 10px;

  transition: transform 0.3s ease-in-out, opacity 0.3s ease-in-out;
  transform: translateY(0);
  opacity: 1;
}


.chatbot.close {
  transform: translateY(100%);
  opacity: 0;
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
  padding: 5px 0 0px 10px;
  display: flex;
  align-items: center;
  justify-content: flex-start; /* Alinea el título al inicio (izquierda) */
  width: 100%;
}

.chatbot-header p {
  margin: 0;
  padding-left: 10px;
  padding-bottom: 5px;
  display: flex;
  align-items: center;
  justify-content: flex-start; /* Alinea el texto de la ciudad al inicio (izquierda) */
  width: 100%;
  font-weight: bold;
}

/* boton para cerrar el chat */
.chatbot-header .close-btn {
  background-color: rgb(216, 43, 43); /* Fondo rojo */
  color: white; /* Texto blanco */
  border: none; /* Sin borde */
  font-size: 20px; /* Tamaño del texto */
  cursor: pointer; /* Cambia el cursor al pasar sobre el botón */
  border-radius: 50%; /* Hace que el botón sea redondo */
  width: 30px; /* Ancho del botón */
  height: 30px; /* Altura del botón */
  display: flex;
  align-items: center;
  justify-content: center;
  position: absolute; /* Asegúrate de que se posicione correctamente */
  top: 10px; /* Ajusta la posición según sea necesario */
  right: 10px; /* Ajusta la posición según sea necesario */
}

.chatbot-header .close-btn:hover {
  background-color: rgb(240, 140, 140); /* Fondo rojo oscuro al pasar el mouse */
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


/* .bot-message-with-options {
  margin-bottom: 7px; 
} */


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
  font-size: 19px;
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
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 32px;

  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  font-size: 21px;
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
  margin: 9px 3px 0px 0px;
  padding: 5px 15px;
  background-color: #06ce4f;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.two-option-button:hover {
  background-color: #52f08c;
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
  font-size: 11px;
}

/* 
.two-option-content:hover {
  font-size: 13px;
} */

/* tool tip */

.chatbot-container {
  position: relative;
  /* height: 100vh; */
}

.chatbot-button-wrapper {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 1000;
}

.chatbot-tooltip {
  position: absolute;
  top: 48%;
  right: calc(100% + 1px); /* Ajustar la distancia entre el tooltip y el botón */
  transform: translateY(-50%);
  /* background-color: #002f5d; */
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



/* animación de la flecha */

#flechaAnimada {
  width: 200px;
  height: 75px;
  animation: bounce 3s infinite;
  
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-30px);
  }
  60% {
    transform: translateY(-15px);
  }
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
  width: 100%;
  min-width: 245px; 
  box-sizing: border-box;
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
  white-space: nowrap;
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

/* botones de rubros agrupados */

.options-grouped-container-yellow {
  display: flex;
  flex-wrap: wrap;
  gap: 3px;
  /* justify-content: center; */
  margin: 3px 0px;
}

.option-grouped-button-yellow {
  background-color: #fdd835; /* Color amarillo */
  color: black;
  padding: 4px 9px;
  border: none;
  border-radius: 50px; /* Para que los bordes sean redondeados */
  cursor: pointer;
  font-weight: bold;
  text-align: center;
  white-space: nowrap;
  font-size: 10px; 
} 

.option-grouped-button-yellow:hover {
  background-color: #ffeb3b; /* Color amarillo más claro en el hover */
}

/* botón del PDF */

.pdf-download-button {
  display: inline-flex;
  align-items: center;
  background-color: #ECECEC; /* Fondo blanco */
  color: #333; /* Color del texto */
  padding: 5px 10px;
  border: 1px solid #ccc; /* Borde gris claro */
  border-radius: 5px;
  text-decoration: none;
  margin: 5px 0;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

.pdf-download-button:hover {
  background-color: #ffffff; /* Fondo gris claro en hover */
}

.pdf-download-icon {
  width: 20px;
  height: 20px;
  margin-right: 5px;
}

.pdf-download-button span {
  font-weight: bold; /* Texto en negrita */
  font-size: 13px; /* Ajusta el tamaño del texto aquí */
}

/* nuevo estilo de menú principal */

.option-principal-menu-button {
  display: block; /* Hacer que el botón sea un elemento de bloque */
  width: calc(100% - 20px); /* Asegurar que el botón ocupe todo el ancho disponible con algo de margen */
  margin: -2px 13px; /* Espaciado automático a los lados y margen vertical */
  padding: 9px;
  border: 1px solid #ccc;
  border-radius: 0px;
  background-color: #fff;
  cursor: pointer;
  font-size: 18px; /* Tamaño del texto */
  color: #1da74f; /* Color del texto */
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease;
  text-align: center; /* Alinear el texto a la izquierda */
  font-weight: bold;
  font-size: 15px;
}

.option-principal-menu-button:hover {
  background-color: #c1ffd8; /* Fondo gris claro en hover */
}

/* enviar los botones de forma simple */
.simple-option-container {
  margin: 1px 0; /* Ajusta el margen superior e inferior */
}

/* mismo ancho para los botones whatsapp maps y buscador */
.button-full-width {
  /* display: inline-flex;
  align-items: center; */
  /* background-color: #00aaff; */
  /* color: white;
  padding: 5px 10px;
  border-radius: 5px;
  text-decoration: none;
  margin: 1px 0px 1px 9px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1); */
  width: calc(100% - 18px); /* Asegura que el botón ocupe el ancho completo menos el margen */
  box-sizing: border-box; /* Incluye el padding y el borde en el cálculo del ancho */
}

/* .button-full-width img {
  width: 20px;
  height: 20px;
  margin-right: 5px;
} */

.button-full-width span {
  color: white; /* Asegura que el texto dentro del botón esté en color blanco */
}

.button-full-width:hover {
  text-decoration: none;
}


/* tamaño de los botones */

/* .option .message-bubble, .two-option-content, .option-grouped-button, .option-principal-menu-button {
  font-size: 18px;
}

.option-grouped-button-yellow{
  font-size: 13px;
}

.whatsapp-button span, .maps-button span, .search-button span {
  font-size: 18px; 
}
.chatbot-header h3 {
  font-size: 30px; 
}

.chatbot-header p {
  font-size: 24px; 
} */

/* nuevo menú parte inferior */

.menu-toggle {
  width: 100%; /* Ajusta el ancho para que considere el padding y margen del contenedor */
  background-color: #25D366;
  color: white;
  padding: 10px 0 10px 0;
  text-align: center;
  cursor: pointer;
  border-radius: 15px 15px 0 0;
  margin-bottom: 10px 0 10px 0;
}

.menu-options {
  width: 100%;
  display: flex;
  flex-direction: column;
  margin-bottom: 5px;
  transition: max-height 0.3s ease-in-out, transform 0.3s ease-in-out;
  max-height: 0;
  overflow: hidden;
  /* transform: translateY(-100%); */
  
  background-color: #ffffff;
}

.menu-options.show {
  max-height: 500px; /* Ajusta según el contenido */
  transform: translateY(0);
}


.menu-option-button {
  background-color: #f0f0f0f0;
  border: none;
  padding: 10px;
  text-align: center;
  cursor: pointer;
  margin: 5px 8px;
  border-radius: 5px;
  color: #095a27;
  font-weight: bold;
}

.menu-option-button:hover {
  background-color: #bbf7d2;
  color: #095a27;
}



/* Estilos responsivos para el chatbot */
@media (max-width: 768px) {
  .chatbot {
    width: 90%;
    height: 80%; /* Ajusta la altura según sea necesario */
    bottom: 0;
    right: 0;
    border-radius: 0;
  }

  .chatbot-header {
    padding: 10px;
    font-size: 16px;
  }

  .chatbot-body {
    padding: 10px;
  }

  .chatbot-footer input {
    padding: 10px;
    font-size: 16px;
  }

  .menu-toggle, .menu-option-button {
    padding: 10px;
    font-size: 14px;
  }

  .two-option-button, .option-grouped-button, .option-principal-menu-button, .option-grouped-button-yellow, .simple-option-container, .button-full-width, .whatsapp-button, .maps-button, .search-button {
    font-size: 14px;
    padding: 10px;
  }

  .whatsapp-button, .maps-button, .search-button {
    width: 100%;
  }

  .message-bubble {
    font-size: 14px;
  }

  .temporary-message {
    bottom: 100px; /* Ajusta según sea necesario para que aparezca justo encima del input */
    font-size: 12px;
    width: 90%; /* Ajusta el ancho según sea necesario */
  }

  .autocomplete-results {
    font-size: 12px;
  }
}



/* Añade un scroll a toda la página */
/* body {
  height: 200vh;
} */



</style>
