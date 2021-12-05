<template>
  <div class="signUp_user">
    <div class="container_signUp_user">
      <h2>Registro de Usuario</h2>

      <form v-on:submit.prevent="processSignUp">
        <input type="text" v-model="user.username" placeholder="Username" />
        <br />

        <input type="password" v-model="user.password" placeholder="Password" />
        <br />

        <input type="text" v-model="user.name" placeholder="Nombre" />
        <br />

        <input type="text" v-model="user.lastname" placeholder="Apellido" />
        <br />

        <input type="email" v-model="user.email" placeholder="Correo" />
        <br />

        <input type="text" v-model="user.address" placeholder="Dirección" />
        <br />

<!--         <input type="number" v-model="user.balance" placeholder="Saldo inicial"/>
        <br /> -->

        <button type="submit">Registrarse</button>
      </form>
    </div>
  </div>
</template>

<script>
import gql from "graphql-tag";

export default {
  name: "SignUp",

  data: function() {
    return {
      user: {
        username: "",
        password: "",
        name: "",
        lastname: "",
        email: "",
        address: "",
      },
    };
  },

  methods: {
    processSignUp: async function() {
      await this.$apollo
        .mutate({
          mutation: gql`
            mutation($userInput: SignUpInput!) {
              signUpUser(userInput: $userInput) {
                refresh
                access
              }
            }
          `,
          variables: {
            userInput: this.user,
          },
        })
        .then((result) => {
          let dataLogIn = {
            username: this.user.username,
            token_access: result.data.signUpUser.access,
            token_refresh: result.data.signUpUser.refresh,
          };
          this.$emit("completedSignUp", dataLogIn);
        })
        .catch((error) => {
          alert("ERROR: Fallo en el registro.");
        });
    },
  },
};
</script>

<style>
html{
  background-color: #faffe1;
}
.signUp_user {
  margin: 0;
  padding: 0%;
  height: 100%;
  width: 100%;

  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #faffe1;
}

.container_signUp_user {
  border: 3px solid #faffe1;
  border-radius: 10px;
  width: 25%;
  height: 90%;

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  font-family: Arial, Helvetica, sans-serif;
}

.signUp_user h2 {
  color: #4b3019;
}

.signUp_user form {
  width: 70%;
}

.signUp_user input {
  height: 40px;
  width: 100%;

  box-sizing: border-box;
  padding: 10px 20px;
  margin: 5px 0;

  border: 1px solid #4b3019;
}

.signUp_user button {
  width: 100%;
  height: 40px;

  color: #e5e7e9;
  background: #4b3019;
  border: 1px solid #e5e7e9;

  border-radius: 5px;
  padding: 10px 25px;
  margin: 5px 0 25px 0;
}

.signUp_user button:hover {
  color: #e5e7e9;
  background: rgb(16, 126, 65);
  border: 1px solid #283747;
}
</style>
