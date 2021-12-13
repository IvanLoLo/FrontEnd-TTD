<template>
  <div id="app" class="app">
    <header class="header">
      <h1>Transport To Door</h1>
      <img src="./assets/logo.png" alt="Logo-TTDOOR" />
      <nav>
        <button v-if="is_auth" v-on:click="loadHome">Cuenta</button>
        <button v-if="is_auth" v-on:click="loadServices">Pedidos</button>
        <button v-if="is_auth && admin()" v-on:click="loadAllServices">Admin</button>
        <button v-if="is_auth" v-on:click="loadPQR">PQRS</button>
        <button v-if="is_auth" v-on:click="logOut">Cerrar Sesión</button>
        <button v-if="!is_auth" v-on:click="loadLogIn">Iniciar Sesión</button>
        <button v-if="!is_auth" v-on:click="loadSignUp">Registrarse</button>
      </nav>
    </header>

    <div class="main-component">
      <router-view
        v-on:completedLogIn="completedLogIn"
        v-on:completedSignUp="completedSignUp"
        v-on:logOut="logOut"
        v-on:loadFormNew="loadFormNew"
        v-on:loadFormEdit="loadFormEdit"
        v-on:loadDetalles="loadDetalles"
      >
      </router-view>
    </div>

    <footer class="footer">
      <h2>TTDOOR - Misión TIC 2022. Copyright <svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-copyright" width="44" height="30" viewBox="0 0 24 24" stroke-width="1.5" stroke="#fdfefe" fill="none" stroke-linecap="round" stroke-linejoin="round">
  <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
  <circle cx="12" cy="12" r="9" />
  <path d="M14.5 9a3.5 4 0 1 0 0 6" />
</svg></h2>
    </footer>
  </div>
</template>

<script>
import jwt_decode from "jwt-decode";

export default {
  name: "App",

  computed: {
    is_auth: {
      get: function() {
        return this.$route.meta.requiresAuth;
      },
      set: function() {},
    },
  },

  methods: {

    admin: function(){
      return jwt_decode(localStorage.getItem("token_refresh")).user_id == 1;
    },

    loadLogIn: function() {
      this.$router.push({ name: "logIn" });
    },

    loadSignUp: function() {
      this.$router.push({ name: "signUp" });
    },

    completedLogIn: function(data) {
      localStorage.setItem("username", data.username);
      localStorage.setItem("token_access", data.token_access);
      localStorage.setItem("token_refresh", data.token_refresh);
      alert("Autenticación Exitosa");
      this.loadHome();
    },

    completedSignUp: function(data) {
      alert("Registro Exitoso");
      this.completedLogIn(data);
    },

    loadHome: function() {
      this.$router.push({ name: "home" });
    },

    loadServices: function() {
      this.$router.push({ name: "services" });
    },

    loadAllServices: function() {
      this.$router.push({ name: "allServices" });
    },

    loadFormNew: function() {
      this.$router.push({ name: "formServicesNew" });
    },

    loadFormEdit: function(id) {
      this.$router.push({ name: "formServicesEdit", params: { deliver: id} });
    },

    loadPQR: function() {
      this.$router.push({ name: "pqr" });
    },

    loadDetalles: function(id) {
      this.$router.push({ name: "detalles", params: { deliver: id} });
    },

    loadAccount: function() {
      this.$router.push({ name: "account" });
    },

    loadTransaction: function() {
      this.$router.push({ name: "transaction" });
    },

    logOut: function() {
      localStorage.clear();
      alert("Sesión Cerrada");
      this.loadLogIn();
    },
  },
};
</script>

<style>
div .main-component{
  height: 75vh;
}

body {
  margin: 0 0 0 0;
}

.header {
  margin: 0%;
  padding: 0;
  width: 100vw;
  height: 6vh;
  min-height: 100px;

  background-color: #9c9d63;
  color: #e5e7e9;

  display: flex;
  justify-content: space-between;
  align-items: center;
  font-family:Arial, Helvetica, sans-serif;
}

.header h1 {
  width: 20%;
  text-align: center;
}

.header nav {
  height: 6vw;
  width: 30vw;
  margin-right: 2vw;

  display: flex;
  justify-content: space-around;
  align-items: center;
}

.header nav button {
  color: #e5e7e9;
  background: #4b3019;
  border: 1px solid #e5e7e9;

  border-radius: 5px;
  padding: 0;
  margin-right: 1vw;

  font-size: 0.8vw;
}

.header nav button:hover {
  color: #ffffff;
  background: #135700;
  border: 1px solid #e5e7e9;
}

.main-component {
  height: 75vh;
  margin: 0%;
  padding: 0%;

  background: #fdfefe;
}

.footer {
  margin-top: 2.4vw;
  padding: 0;
  width: 100vw;
  height: 5vw;
  background-color: #9c9d63;
  color: #e5e7e9;
  
}

.footer h2 {
  width: 100vw;
  height: 4vw;

  display: flex;
  justify-content: center;
  align-items: center;
  font-family: Arial, Helvetica, sans-serif;
}

img {
  width: 90px;
  margin-left: 11vw;
}
</style>
