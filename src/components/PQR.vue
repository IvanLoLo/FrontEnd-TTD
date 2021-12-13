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
      <form v-on:submit.prevent="procesar($event)">
        <h2>
          Seleccione el tipo de solicitud:
        </h2>
        <select
          name="tipo"
          type="text"
          v-model="editpqr.tipo"
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
        <h2>Seleccione el domicilio de la PQRS a realizar:</h2>
        <div class="container-table">
          <table>
            <tr>
              <th>Numero de rastreo</th>
              <th>Usuario Origen</th>
              <th>Usuario Destino</th>
              <th>Ciudad Origen</th>
              <th>Ciudad Destino</th>
              <th>Estado</th>
              <th>PQRS</th>
            </tr>

            <tr v-for="pqr in deliveriesByUsername" :key="pqr.id">
              <td>{{ pqr.id }}</td>
              <td>{{ pqr.usernameEmisor }}</td>
              <td>{{ pqr.usernameReceptor }}</td>
              <td>{{ pqr.ciudadOrigen }}</td>
              <td>{{ pqr.ciudadDestino }}</td>
              <td>{{ pqr.estado }}</td>
              <td>{{ pqr.pqr }}</td>
              <button class="seleccionar" v-on:click="seleccionar(pqr.id)">
                Seleccionar
              </button>
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
  name: "PQR",

  data: function() {
    return {
      username: localStorage.getItem("username") || "none",
      deliveriesByUsername: [],
      editpqr: {
        pqr: "",
        tipo: "",
        delivery: "",
      },
      infoDelivery: {
        id: null,
        usernameEmisor: localStorage.getItem("username"),
        usernameReceptor: "",
        ciudadOrigen: "",
        ciudadDestino: "",
        direccionOrigen: "",
        direccionDestino: "",
        value: null,
        estado: null,
        description: null,
        pickUpDate: null,
        deliverDate: null,
        pqr: null,
      },
    };
  },
  //// PQRS
  methods: {
    procesar: function(event) {
      if (event.submitter.outerText == "Seleccionar") return;
      this.processpqr();
    },

    seleccionar: function(id) {
      console.log("Seleccionado");
      console.log(id);
      console.log(this.editpqr.tipo);
      this.editpqr.delivery = id;
    },

    processpqr: async function() {
      //Revisar que haya una queja y un tipo
      if (this.editpqr.pqr.trim() == "" || this.editpqr.pqr == null) {
        alert("Por favor ingrese una queja");
        return;
      }

      if (this.editpqr.tipo.trim() == "" || this.editpqr.tipo == null) {
        alert("Por favor ingrese un tipo de queja");
        return;
      }

      //Revisar que se haya escogido un pedido editpqr.delivery != ""
      if (this.editpqr.delivery.trim() == "" || this.editpqr.delivery == null) {
        alert(
          "Por favor seleccione el pedido sobre el cual desea realizar el comentario"
        );
        return;
      }

      //Traer la informacion de ese delivery

      await this.$apollo
        .query({
          query: gql`
            query($deliveryId: String!) {
              deliveryById(deliveryId: $deliveryId) {
                id
                usernameEmisor
                usernameReceptor
                ciudadOrigen
                ciudadDestino
                direccionOrigen
                direccionDestino
                description
                estado
                value
                pickUpDate
                deliverDate
                pqr
              }
            }
          `,
          variables: {
            deliveryId: this.editpqr.delivery,
          },
        })
        .then((result) => {
          console.log(result);
          this.infoDelivery = {
            id: result.data.deliveryById.id,
            usernameEmisor: result.data.deliveryById.usernameEmisor,
            usernameReceptor: result.data.deliveryById.usernameReceptor,
            ciudadOrigen: result.data.deliveryById.ciudadOrigen,
            ciudadDestino: result.data.deliveryById.ciudadDestino,
            direccionOrigen: result.data.deliveryById.direccionOrigen,
            direccionDestino: result.data.deliveryById.direccionDestino,
            value: result.data.deliveryById.value,
            estado: result.data.deliveryById.estado,
            description: result.data.deliveryById.description,
            pickUpDate: result.data.deliveryById.pickUpDate,
            deliverDate: result.data.deliveryById.deliverDate,
            pqr: result.data.deliveryById.pqr,
          };
          console.log(this.infoDelivery);
          if (this.infoDelivery.pqr == null) this.infoDelivery.pqr = "";
          this.infoDelivery.pqr +=
            this.editpqr.tipo + ": " + this.editpqr.pqr + ". ";
        })
        .catch((error) => {
          console.log(error);
          alert("Error trayendo info en PQR");
        });

      //Editar el delivery con la info de la PQR

      await this.$apollo
        .mutate({
          mutation: gql`
            mutation($delivery: DeliveryInput!) {
              editDelivery(delivery: $delivery) {
                id
                usernameEmisor
                usernameReceptor
                ciudadOrigen
                ciudadDestino
                pqr
              }
            }
          `,
          variables: {
            delivery: this.infoDelivery,
          },
        })
        .then((result) => {
          console.log(result);
          alert("Servicio editado satisfactoriamente");
        })
        .catch((error) => {
          console.log(error);
          alert(
            "Hay un error en la información. Por favor verifique e intente de nuevo"
          );
        });
    },
  },

  apollo: {
    deliveriesByUsername: {
      query: gql`
        query Query($username: String!) {
          deliveriesByUsername(username: $username) {
            usernameEmisor
            usernameReceptor
            ciudadOrigen
            ciudadDestino
            id
            estado
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
  top: 5vh;
}

#Historial {
  width: 100vw;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  flex-direction: column;
}

#Historial .container-table {
  width: 55vw;
  max-height: 13vw;
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
  padding-top: 0.2vw;
  padding-bottom: 0.2vw;
  text-align: center;
  background-color: #9c9d63;
  color: white;
}

#Historial > h2 {
  color: #4b3019;
  font-size: 1vw;
}

#Historial h3 {
  color: #2e1d0f;
  font-size: 1vw;
}

#Historial h1 {
  color: #4b3019;
  font-size: 2.2vw;
}

#Historial .container {
  padding: 0vw;
  /* border: 3px solid rgba(0, 0, 0, 0.3); */
  border-radius: 20px;
  margin: 10vw 0 1vw 0;
}

#Historial .container h2 {
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
  width: 19vw;
  left: 0vh;
  margin-left: 0;
  top: 10vw;
}

input {
  height: 3vw;
  width: 100%;
  box-sizing: border-box;
  padding: 8px 20px;
  margin: 5px 0;
  border: 1px solid #4b3019;
  font-size: 0.9vw;
}

select {
  height: 3vw;
  width: 20vw;
  box-sizing: border-box;
  padding: 8px 20px;
  margin: 5px 0;
  border: 1px solid #4b3019;
  font-size: 0.9vw;
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
table button:focus {
  color: #e5e7e9;
  background: rgb(46, 199, 115);
  border: 1.2px solid #4b3019;
}

@media (max-width: 1189px) {
  .logo2 {
    display: none;
  }

  #Historial .container-table {
    width: 85vw;
  }
}
@media (max-width: 1300px) {
  #Historial .container {
    padding: 0vw;
    border-radius: 20px;
    margin: 12vw 0 1vw 0;
  }
}
</style>
