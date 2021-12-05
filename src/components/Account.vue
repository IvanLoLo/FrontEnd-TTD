<template>
  <div id="Historial">
    <div>
      <h1>Peticiones, Quejas, Reclamos y Sugerencias (PQRS)</h1>
      <h3>
        Bienvenido
        <span>{{ username }}.</span>
        En este apartado podrás solitiar soporte, queja o reclamo a tus pedidos.
      </h3>
      <img class="logo2" src="../assets/logo2.png" alt="" />
    </div>
    <div class="container">
      <form v-on:submit.prevent="processTransaction">
        <h2>
          Seleccione el tipo de solicitud:
        </h2>
        <!--         <input
          type="text"
          v-model="createTransaction.usernameDestiny"
          placeholder="Usuario Destino"
        /> -->
        <select
          name="tipo"
          type="text"
          v-model="createTransaction.usernameDestiny"
          placeholder="PQRS"
        >
          <option>Pregunta</option>
          <option>Queja</option>
          <option>Reclamo</option>
          <option>Sugerencia</option>
        </select>
        <h2>
          Comentanos tu inquietud:
        </h2>
        <input
          type="text"
          v-model="createTransaction.value"
          placeholder="..."
        />
        <br />

        <!--         <h2>
          Último Movimiento:
          <span>
            {{ accountByUsername.lastChange.substring(0, 10) }}
            {{ accountByUsername.lastChange.substring(11, 19) }}
          </span>
        </h2> -->

        <h2>Seleccione el domicilio realizado:</h2>
        <div class="container-table">
          <table>
            <tr>
              <th>Fecha</th>
              <th>Hora</th>
              <th>Origen</th>
              <th>Destino</th>
              <th>Valor</th>
            </tr>

            <tr
              v-for="transaction in transactionByUsername"
              :key="transaction.id"
            >
              <td>{{ transaction.date.substring(0, 10) }}</td>
              <td>{{ transaction.date.substring(11, 19) }}</td>
              <td>{{ transaction.usernameOrigin }}</td>
              <td>{{ transaction.usernameDestiny }}</td>
              <td>${{ transaction.value }} COP</td>
              <button class="selelccionar">Seleccionar</button>
            </tr>
          </table>
        </div>
        <button id="enviar" type="submit">Enviar</button>
      </form>
    </div>
  </div>
</template>

<script>
import gql from "graphql-tag";

export default {
  name: "Account",

  data: function() {
    return {
      username: localStorage.getItem("username") || "none",
      transactionByUsername: [],
      accountByUsername: {
        balance: "",
        lastChange: "",
      },
      createTransaction: {
        ///Registro datos PQRS
        usernameOrigin: localStorage.getItem("username"),
        usernameDestiny: "",
        value: "",
      }, ///
      fields: [
        ////// Boton seleccionar
        { key: "id_consola", label: "ID Consola" },
        { key: "marca", label: "Marca" },
        { key: "modelo", label: "Modelo" },
        { key: "precio", label: "Precio" },
        { key: "descuento", label: "Descuento" },
        { key: "descripcion", label: "Descripción" },
        { key: "cantidad", label: "Cantidad", variant: "active" },
        { key: "action", label: "" },
      ],
      consolas: [], ////// b seleccionar
    };
  },
  //// PQRS
  methods: {
    processTransaction: async function() {
      if (
        localStorage.getItem("token_access") === null ||
        localStorage.getItem("token_refresh") === null
      ) {
        this.$emit("logOut");
        return;
      }

      localStorage.setItem("token_access", "");

      await this.$apollo
        .mutate({
          mutation: gql`
            mutation($refresh: String!) {
              refreshToken(refresh: $refresh) {
                access
              }
            }
          `,
          variables: {
            refresh: localStorage.getItem("token_refresh"),
          },
        })
        .then((result) => {
          localStorage.setItem("token_access", result.data.refreshToken.access);
        })
        .catch((error) => {
          this.$emit("logOut");
          return;
        });

      await this.$apollo
        .mutate({
          mutation: gql`
            mutation($transaction: TransactionInput!) {
              createTransaction(transaction: $transaction) {
                date
                id
                usernameDestiny
                usernameOrigin
                value
              }
            }
          `,
          variables: {
            transaction: this.createTransaction,
          },
        })
        .then((result) => {
          alert("Transacción Realizada de Manera Correcta, Revise Historial");
        })
        .catch((error) => {
          alert("Saldo Insuficiente o Destino Incorrecto");
        });
    },
    getConsolas() {
      ////////// Botón seleccionar
      const path = "https://mynthic2.herokuapp.com/ver-inventario/";
      axios
        .get(path)
        .then((response) => {
          this.consolas = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    created() {
      this.getConsolas();
    }, ///////////////B seleccionar
  },
  ////////

  apollo: {
    transactionByUsername: {
      query: gql`
        query($username: String!) {
          transactionByUsername(username: $username) {
            id
            usernameOrigin
            usernameDestiny
            value
            date
          }
        }
      `,
      variables() {
        return {
          username: this.username,
        };
      },
    },

    accountByUsername: {
      query: gql`
        query($username: String!) {
          accountByUsername(username: $username) {
            balance
            lastChange
          }
        }
      `,
      variables() {
        return {
          username: this.username,
        };
      },
    },
  },

  created: function() {
    this.$apollo.queries.transactionByUsername.refetch();
    this.$apollo.queries.accountByUsername.refetch();
  },
};
</script>

<style>
div .main-component {
  background-color: #faffe1;
  font-family: Arial, Helvetica, sans-serif;
}

div .container {
  display: flex;
  position: absolute;
  top: 18vh;
}

#Historial {
  width: 100%;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  flex-direction: column;
}

#Historial .container-table {
  width: 800px;

  max-height: 250px;
  overflow-y: scroll;
  overflow-x: hidden;
}

#Historial table {
  width: 100%;
  border-collapse: collapse;
  border: 1px solid rgba(0, 0, 0, 0.3);
}

#Historial table td,
#Historial table th {
  border: 1px solid #ddd;
  padding: 8px;
}

#Historial table tr:nth-child(even) {
  background-color: #f2f2f2;
}

#Historial table tr:hover {
  background-color: #ddd;
}

#Historial table th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #9c9d63;
  color: white;
}

#Historial > h2 {
  color: #283747;
  font-size: 25px;
}

#Historial .container {
  padding: 30px;
  /* border: 3px solid rgba(0, 0, 0, 0.3); */
  border-radius: 20px;
  margin: 5% 0 1% 0;
}

#Historial .container h2 {
  font-size: 25px;
  color: #283747;
}
#Historial .container span {
  color: crimson;
  font-weight: bold;
}

.logo2 {
  display: flex;
  position: absolute;
  width: 40vh;
  left: 0vh;
  margin-left: 0vh;
  top: 14vh;
}

input {
  height: 40px;
  width: 100%;

  box-sizing: border-box;
  padding: 10px 20px;
  margin: 5px 0;

  border: 1px solid #4b3019;
}
select {
  height: 40px;
  width: 100%;

  box-sizing: border-box;
  padding: 10px 20px;
  margin: 5px 0;

  border: 1px solid #4b3019;

  font-size: 15px;
}

button {
  width: 100%;
  height: 40px;

  color: #e5e7e9;
  background: #9e6838;
  border: 1px solid #e5e7e9;

  border-radius: 5px;
  padding: 10px 25px;
  margin: 5px 0 25px 0;
}
button:hover {
  color: #e5e7e9;
  background: rgb(46, 199, 115);
  border: 1px solid #283747;
}
</style>
