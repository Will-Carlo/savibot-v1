<template>
  <div class="chatbot">
    <div class="chatbot-header">
      <h2>SaviBot</h2>
    </div>
    <div class="chatbot-body">
      <div v-for="(message, index) in messages" :key="index" :class="message.sender">
        <p>{{ message.text }}</p>
      </div>
    </div>
    <div class="chatbot-footer">
      <input v-model="userMessage" @keyup.enter="sendMessage" placeholder="Type your message..." />
      <button @click="sendMessage">Send</button>
    </div>
  </div>
</template>


  <script>
  export default {
    data() {
      return {
        messages: [],
        userMessage: '',
        botName: 'SaviBot',
        contactList: [
          { id: 3, city_id: 2, order: 10, title: 'TIENDA EL ALTO', description: 'EL CEIBO', whatsapp: 0 },
          { id: 4, city_id: 2, order: 11, title: 'TIENDA EL ALTO', description: 'SATELITE', whatsapp: 0 },
          { id: 6, city_id: 1, order: 1, title: 'TIENDA LA PAZ', description: 'CALLE LOAYZA', whatsapp: 0 },
          { id: 7, city_id: 1, order: 2, title: 'TIENDA LA PAZ', description: 'C.ZAPATA - UMSA', whatsapp: 0 },
          { id: 8, city_id: 5, order: 1, title: 'TIENDA COCHABAMBA', description: 'CALLE SUCRE', whatsapp: 0 },
          { id: 9, city_id: 8, order: 1, title: 'TIENDA SUCRE', description: 'CALLE REGIMIENTO', whatsapp: 0 },
          { id: 10, city_id: 7, order: 1, title: 'TIENDA TARIJA', description: 'CALLE A. DEL CARPIO', whatsapp: 0 },
          { id: 11, city_id: 6, order: 1, title: 'TIENDA ORURO', description: 'CALLE MONTECINO', whatsapp: 0 },
          { id: 12, city_id: 9, order: 1, title: 'TIENDA POTOSI', description: 'AV. PRADO SAN CLEMENTE', whatsapp: 0 },
          { id: 13, city_id: 1, order: 100, title: 'AREA DE COMPUTACION', description: '', whatsapp: 74040348 },
          { id: 14, city_id: 1, order: 200, title: 'AREA IMPRESORAS 3D', description: '', whatsapp: 68068883 },
        ],
        listCity: ["La Paz", "El Alto", "Cochabamba", "Santa Cruz", "Tarija", "Sucre", "Oruro", "Potosí"],
        listArea: ["Impresoras 3D", "Fotocopiadoras", "Sublimación", "Cortadora láser", "Computadoras", "Bioseguridad", "Impresoras", "Papel", "Novedades", "Otros"],
        listMenu: ["📦 Productos y precios", "🗺️ Dirección", "💰 Promociones y ofertas", "📖 Ver catálogos", "🙋🏻‍♂️ Chatear con un asesor"],
        listAreaSupport: ["🙋🏻‍♂️ Impresoras 3D", "🙋🏻‍♂️ Máquinas láser", "🙋🏻‍♂️ Computadoras", "🙋🏻‍♂️ Sublimación", "🙋🏻‍♂️ Atención general"],
        listCitySupport: ["➡️ La Paz", "➡️ El Alto", "➡️ Cochabamba", "➡️ Santa Cruz", "➡️ Tarija", "➡️ Sucre", "➡️ Oruro", "➡️ Potosí"],
        listAreaCatalog: ["📰 Impresoras 3D", "📰 Fotocopiadoras", "📰 Sublimación", "📰 Cortadora láser", "📰 Computadoras", "📰 Bioseguridad", "📰 Impresoras", "📰 Papel", "📰 Novedades", "📰 Otros"]
      };
    },
    methods: {
      sendMessage() {
        if (this.userMessage.trim() !== '') {
          this.messages.push({ id: Date.now(), text: this.userMessage, sender: 'user' });
          this.handleBotResponse(this.userMessage);
          this.userMessage = '';
        }
      },
      handleBotResponse(userMessage) {
        // Add logic to handle user messages and respond accordingly
        let botMessage = '';
  
        switch (userMessage.toLowerCase()) {
          case '🤖 Empezar':
            botMessage = '¡Hola! Soy SaviBot, tu asistente virtual. ¿En qué puedo ayudarte?';
            break;
          case 'hola':
            botMessage = '¡Hola! Soy SaviBot, tu asistente virtual. ¿En qué puedo ayudarte?';
            break;
          case '📅 horarios':
            botMessage = 'Nuestros horarios de atención en tiendas son:\n' +
                         '✅ Lunes a Viernes\n' +
                         '    ➡ de 8:30am a 12:30pm\n' +
                         '    ➡ de 2:30pm a 7:00pm\n' +
                         '✅ Sábados\n' +
                         '    ➡ de 9:00am a 1:00pm';
            break;
          case '📦 productos y precios':
            botMessage = 'Escoge el área del producto que buscas:\n' +
                         this.listArea.join('\n');
            break;
          case '🗺️ dirección':
            botMessage = 'Por favor, indícame de qué ciudad me escribes:\n' +
                         this.listCity.join('\n');
            break;
          case '💰 promociones y ofertas':
            botMessage = 'Mantente al pendiente de nuestras ofertas 😉';
            break;
          case '📖 ver catálogos':
            botMessage = 'Excelente, escoge el área del catálogo que quieres ver:\n' +
                         this.listAreaCatalog.join('\n');
            break;
          case '🙋🏻‍♂️ chatear con un asesor':
            botMessage = 'Muy bien ¿En qué área necesitas la atención al cliente? 🤔\n' +
                         this.listAreaSupport.join('\n');
            break;
          default:
            if (this.listCity.includes(userMessage)) {
              botMessage = `En ${userMessage} atendemos en: \n` +
                           this.getAddressByCity(userMessage).join('\n');
            } else if (this.listArea.includes(userMessage)) {
              botMessage = `¿Qué producto estás buscando en el área ${userMessage}?`;
            } else if (this.listAreaCatalog.includes(userMessage)) {
              botMessage = `Lo siento, esta área esta en construcción 🛠️`;
            } else if (this.listAreaSupport.includes(userMessage)) {
              botMessage = `Nuestro asesor en el área de ${userMessage} te atenderá con gusto 😊`;
            } else {
              botMessage = 'Lo siento, no entendí tu mensaje.';
            }
            break;
        }
  
        this.messages.push({ id: Date.now(), text: botMessage, sender: 'bot' });
      },
      getAddressByCity(city) {
        const addresses = {
          'La Paz': [
            "🏢 Calle Loayza # 349, local 3 (Frente a la facultad de Derecho UMSA)",
            "📲 72030101",
            "📌 https://maps.app.goo.gl/tNsAqrArK2NfGnM47",
            "🏢 Calle Zapata # 141 (frente Monoblock UMSA)",
            "📲 72030107",
            "📌 https://maps.app.goo.gl/vnP2W9hk2oJZMSwx5",
            "🏢 Calle 2 de obrajes entre Av. Hernando Siles y Av. 14 de Septiembre (Frente Universidad Catolica)",
            "📲 71545171",
            "📌 https://maps.app.goo.gl/Vvw4BjAnpP6MFnwa8"
          ],
          'El Alto': [
            "🏢 Av. Juan Pablo II Edif. EI Ceibo Local A-15 (Final Autopista casi esq. Rene Dorado)",
            "📲 72029023",
            "📌 https://maps.app.goo.gl/vTUrQCpyNQmC24hH6",
            "🏢 Avenida Satélite # 668 (Cerca al Banco Mercantil Santa Cruz)",
            "📲 71543980",
            "📌 https://maps.app.goo.gl/bagfMNGR4GSmpom9A"
          ],
          'Cochabamba': [
            "🏢 Calle Sucre # 882 (Casi esquina Oquendo)",
            "📲 72030102",
            "📌 https://maps.app.goo.gl/6MfeLnrtaiAk9p6y9"
          ],
          'Santa Cruz': [
            "🏢 Avenida Centenario # 113 casi esquina Palermo (entre primer y segundo anillo)",
            "📲 72030103",
            "📌 https://maps.app.goo.gl/1xw1r9zfBwv1pQJK6"
          ],
          'Tarija': [
            "🏢 Calle Alejandro del Carpio # 258 entre Suipacha y Méndez (Zona Las Panosas)",
            "📲 72030105",
            "📌 https://maps.app.goo.gl/rHxKVwKALUQev44QA"
          ],
          'Sucre': [
            "🏢 Calle Regimiento Campos # 174 Esquina Ricardo Andrade (Frente a la Facultad Técnica)",
            "📲 72030104",
            "📌 https://maps.app.goo.gl/bcK8XhSmjCk9daXt7"
          ],
          'Oruro': [
            "🏢 Calle Potosí # 5507 Esquina Montecinos (Diagonal al Col. Juan Misael Saracho)",
            "📲 72030106",
            "📌 https://maps.app.goo.gl/5ARt9qRxZoRzadc89"
          ],
          'Potosí': [
            "🏢 Avenida Prado San Clemente # 29 entre las calles Camargo y 13 de Mayo",
            "📲 68868684",
            "📌 https://maps.app.goo.gl/mzG5tcuqNpD9NcLDA"
          ]
        };
        return addresses[city] || [];
      }
    }
  };
  </script>
  
  <style scoped>
  .chatbot {
    border: 1px solid #ccc;
    width: 300px;
    height: 400px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }
  .chatbot-header {
    background-color: #007bff;
    color: white;
    padding: 10px;
  }
  .chatbot-body {
    flex: 1;
    padding: 10px;
    overflow-y: auto;
  }
  .user {
    text-align: right;
  }
  .bot {
    text-align: left;
  }
  .chatbot-footer {
    padding: 10px;
    border-top: 1px solid #ccc;
  }
  .chatbot-footer input {
    width: calc(100% - 70px);
  }
  .chatbot-footer button {
    width: 60px;
  }
  </style>


  
<!-- <style src="@/assets/css/savin-v4.css"></style> -->