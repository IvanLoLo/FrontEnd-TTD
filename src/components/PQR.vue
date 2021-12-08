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
      <form v-on:submit.prevent="processpqr">
        <h2>
          Seleccione el tipo de solicitud:
        </h2>
        <select
          name="tipo"
          type="text"
          v-model="editpqr.estado"
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
          v-model="editpqr.pqr"
          placeholder="Comenta tus inquietudes en este espacio..."
        />
        <br />
        <h2>
          Ingresa el número de rastreo del pedido de tu inquietud:
        </h2>
        <input
          type="text"
          v-model="editpqr.id"
          placeholder="Número de rastreo"
        />
        <br />
        <h2>Seleccione el domicilio de la PQRS a realizar:</h2>
        <div class="container-table">
          <table>
            <tr>
              <th>Numero de rastreo</th>
              <th>Fecha de solicitud</th>
              <th>Fecha última novedad</th>
              <th>Usuario Origen</th>
              <th>Usuario Destino</th>
              <th>Descripción</th>
              <th>Estado</th>
              <th>Valor</th>
              <th>PQRS</th>
              <!-- <th></th> -->
            </tr>

            <tr v-for="pqr in deliveriesByUsername" :key="pqr.id">
              <td>{{ pqr.id }}</td>
              <td>{{ pqr.deliverDate.substring(0, 10) }}</td>
              <td>{{ pqr.pickUpDate.substring(0, 10) }}</td>
              <td>{{ pqr.usernameEmisor }}</td>
              <td>{{ pqr.usernameReceptor }}</td>
              <td>{{ pqr.description }}</td>
              <td>{{ pqr.estado }}</td>
              <td>{{ pqr.value }}</td>
              <td>{{ pqr.pqr }}</td>
              <!-- <button class="seleccionar" onClick="seleccionar(pqr.i)">
                Seleccionar
              </button> -->
            </tr>
          </table>
        </div>
        <button id="enviar" type="submit" onClick="guardar()">Enviar</button>
      </form>
    </div>
  </div>
</template>

<script>
import gql from "graphql-tag";

export default {
  name: "PQR",

  data: function() {
    return {
      username: localStorage.getItem("username") || "none",
      deliveriesByUsername: [],

      editpqr: {
        ////////Registro datos PQRS////
        usernameEmisor: localStorage.getItem("username"),
        estado: "",
        pqr: "",
        id: "",
        value: localStorage.getItem("value"),
        usernameReceptor: localStorage.getItem("usernameReceptor"),
        ciudadOrigen: localStorage.getItem("ciudadOrigen"),
        ciudadDestino: localStorage.getItem("ciudadDestino"),
        direccionOrigen: localStorage.getItem("direccionOrigen"),
        direccionDestino: localStorage.getItem("direccionDestino"),
        description: localStorage.getItem("description"),
        pickUpDate: localStorage.getItem("pickUpDate"),
        deliverDate: localStorage.getItem("deliverDate"),
      }, ////////
      /*       fields: [ */
      ////// Boton seleccionar
      /*         { key: "id", label: "id" },
      ], */
    };
  },
  //// PQRS
  methods: {
    processpqr: async function() {
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
            mutation RefreshToken($refresh: String!) {
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
        .mutate(
          localStorage.setItem(
            "usernameReceptor",
            this.deliveriesByUsername[0].usernameReceptor
          ),
          localStorage.setItem(
            "ciudadOrigen",
            this.deliveriesByUsername[0].ciudadOrigen
          ),
          localStorage.setItem(
            "ciudadDestino",
            this.deliveriesByUsername[0].ciudadDestino
          ),
          localStorage.setItem(
            "direccionDestino",
            this.deliveriesByUsername[0].direccionDestino
          ),
          localStorage.setItem(
            "direccionOrigen",
            this.deliveriesByUsername[0].direccionOrigen
          ),
          localStorage.setItem("value", this.deliveriesByUsername[0].value),
          localStorage.setItem(
            "description",
            this.deliveriesByUsername[0].description
          ),
          localStorage.setItem(
            "pickUpDate",
            this.deliveriesByUsername[0].pickUpDate
          ),
          localStorage.setItem(
            "deliverDate",
            this.deliveriesByUsername[0].deliverDate
          ),
          localStorage.setItem("value", this.deliveriesByUsername[0].value),
          console.log(this.editpqr),

          (editpqr = {
            ////////Registro datos PQRS////
            usernameEmisor: localStorage.getItem("username"),
            estado: "",
            pqr: "",
            id: "",
            value: localStorage.getItem("value"),
            usernameReceptor: localStorage.getItem("usernameReceptor"),
            ciudadOrigen: localStorage.getItem("ciudadOrigen"),
            ciudadDestino: localStorage.getItem("ciudadDestino"),
            direccionOrigen: localStorage.getItem("direccionOrigen"),
            direccionDestino: localStorage.getItem("direccionDestino"),
            description: localStorage.getItem("description"),
            pickUpDate: localStorage.getItem("pickUpDate"),
            deliverDate: localStorage.getItem("deliverDate"),
          }),
          {
            mutation: gql`
              mutation Mutation($delivery: DeliveryInput!) {
                editDelivery(delivery: $delivery) {
                  pqr
                  estado
                  deliverDate
                  pickUpDate
                  description
                  value
                  direccionDestino
                  direccionOrigen
                  ciudadDestino
                  ciudadOrigen
                  usernameReceptor
                  usernameEmisor
                  id
                }
              }
            `,
            variables: {
              delivery: this.editpqr,
            },
          }
        )

        .then((result) => {
          alert("Solicitud realizada de manera correcta, revise su historial");
        })
        .catch((error) => {
          alert("Error dato incorrecto");
          console.log(this.editpqr);
          //console.log(this.deliveriesByUsername);
        });
    },

    /*     seleccionar(){
      id = this.username.id;
      console.log(this.id);
    } */
  },
  ////////

  apollo: {
    deliveriesByUsername: {
      query: gql`
        query Query($username: String!) {
          deliveriesByUsername(username: $username) {
            usernameEmisor
            usernameReceptor
            ciudadOrigen
            ciudadDestino
            direccionOrigen
            direccionDestino
            value
            description
            estado
            pickUpDate
            deliverDate
            id
            pqr
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
  guardar() {
    localStorage.setItem(
      "usernameReceptor",
      this.deliveriesByUsername[0].usernameReceptor
    );
    localStorage.setItem(
      "ciudadOrigen",
      this.deliveriesByUsername[0].ciudadOrigen
    );
    localStorage.setItem(
      "ciudadDestino",
      this.deliveriesByUsername[0].ciudadDestino
    );
    localStorage.setItem(
      "direccionDestino",
      this.deliveriesByUsername[0].direccionDestino
    );
    localStorage.setItem(
      "direccionOrigen",
      this.deliveriesByUsername[0].direccionOrigen
    );
    localStorage.setItem("value", this.deliveriesByUsername[0].value);
    localStorage.setItem(
      "description",
      this.deliveriesByUsername[0].description
    );
    localStorage.setItem("pickUpDate", this.deliveriesByUsername[0].pickUpDate);
    localStorage.setItem(
      "deliverDate",
      this.deliveriesByUsername[0].deliverDate
    );
    localStorage.setItem("id", this.deliveriesByUsername[0].id);
    console.log(this.editpqr);
  },

  seleccionar() {
    console.log(this.pqr.id);
  },

  created: function() {
    this.$apollo.queries.deliveriesByUsername.refetch();
  },
};
</script>

<style>
div .main-component {
  background-color: #faffe1;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 1.8vh;
}

div .container {
  display: flex;
  position: absolute;
  top: 6vh;
}

#Historial {
  width: 100%;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  flex-direction: column;
}

#Historial .container-table {
  width: 100vh;

  max-height: 185px;
  overflow-y: scroll;
  overflow-x: hidden;
}

#Historial table {
  width: 100%;
  border-collapse: collapse;
  border: 1px solid #ddd;
}

#Historial table td,
#Historial table th {
  border: 1px solid #ddd;
  padding-left: 5px;
  font-size: 1.8vh;
}

#Historial table tr:nth-child(even) {
  background-color: #f2f3c8;
}

#Historial table tr:hover {
  background-color: #dadb82;
}

#Historial table th {
  padding-top: 8px;
  padding-bottom: 8px;
  text-align: center;
  background-color: #9c9d63;
  color: white;
}

#Historial > h2 {
  color: #4b3019;
  font-size: 25px;
}

#Historial h1 {
  color: #4b3019;
  font-size: 25px;
}

#Historial .container {
  padding: 30px;
  /* border: 3px solid rgba(0, 0, 0, 0.3); */
  border-radius: 20px;
  margin: 9% 0 1% 0;
}

#Historial .container h2 {
  font-size: 25px;
  color: #4b3019;
  font-size: 2vh;
}

#Historial .container span {
  color: crimson;
  font-weight: bold;
}

.logo2 {
  display: flex;
  position: absolute;
  width: 20%;
  left: 0vh;
  margin-left: 0vh;
  top: 14vh;
}

input {
  height: 5vh;
  width: 100%;
  box-sizing: border-box;
  padding: 8px 20px;
  margin: 5px 0;
  border: 1px solid #4b3019;
  font-size: 1.7vh;
}

select {
  height: 5vh;
  width: 50%;
  box-sizing: border-box;
  padding: 8px 20px;
  margin: 5px 0;
  border: 1px solid #4b3019;
  font-size: 1.7vh;
}

button {
  width: 100%;
  height: 35px;

  color: #e5e7e9;
  background: #9e6838;
  border: 1.2px solid #e5e7e9;

  border-radius: 5px;
  padding: 10px 25px;
  margin: 5px 0 25px 1;
}
button:hover {
  color: #e5e7e9;
  background: rgb(46, 199, 115);
  border: 1.2px solid #4b3019;
}
</style>
