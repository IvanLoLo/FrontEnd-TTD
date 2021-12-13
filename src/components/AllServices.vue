<template>
  <div class="servicesContainer">
    <h1>Todos los pedidos</h1>

    <div class="tablaPedidos">
      <table>
        <tr>
          <th>Fecha Actualizacion</th>
          <th>Emisor</th>
          <th>Receptor</th>
          <th>Origen</th>
          <th>Destino</th>
        </tr>

        <tr
          v-for="pedido in allDeliveries"
          :key="pedido.id"
          v-on:click="detallesServicio(pedido.id, $event)"
        >
          <td v-if="pedido.pickUpDate && !pedido.deliverDate">
            {{
              pedido.pickUpDate
                .substring(0, 10)
                .split("-")
                .reverse()
                .join("/")
            }}
          </td>
          <td v-if="pedido.deliverDate">
            {{
              pedido.deliverDate
                .substring(0, 10)
                .split("-")
                .reverse()
                .join("/")
            }}
          </td>
          <td v-if="!pedido.pickUpDate && !pedido.deliverDate">-</td>
          <td>{{ pedido.usernameEmisor }}</td>
          <td>{{ pedido.usernameReceptor }}</td>
          <td>{{ pedido.ciudadOrigen }}</td>
          <td>{{ pedido.ciudadDestino }}</td>
          <td id="deleteService">
            <img
              src="http://assets.stickpng.com/images/5a5798809538462e5a82d431.png"
              title="Eliminar Servicio"
              alt="Eliminar Servicio"
              style="cursor: pointer;"
              v-on:click="borrarServicio(pedido.id)"
            />
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import gql from "graphql-tag";

export default {
  name: "Services",

  data: function() {
    return {
      username: localStorage.getItem("username") || "",
      date: new Date()
        .toISOString()
        .substring(0, 10)
        .split("-")
        .reverse()
        .join("/"),
      filter: "",
      allDeliveries: {
        id: "",
        usernameEmisor: "",
        usernameReceptor: "",
        ciudadOrigen: "",
        ciudadDestino: "",
        estado: "",
        pickUpDate: "",
        deliverDate: "",
      },
    };
  },

  apollo: {
    allDeliveries: {
      query: gql`
        query {
          allDeliveries {
            id
            usernameEmisor
            usernameReceptor
            ciudadOrigen
            ciudadDestino
            estado
            pickUpDate
            deliverDate
          }
        }
      `,
    },
  },

  methods: {

    permisos: function(emisor){
      return emisor == localStorage.getItem("username");
    },

    borrarServicio: function(id) {
      console.log(this.allDeliveries);
      let confirmar = confirm("¿Está seguro que desea calcelar este servicio?");
      if(!confirmar) return;
      console.log("Eliminando servicio "+id);
      this.$apollo.mutate({
        mutation: gql`
          mutation($deliveryId: String!) {
            deleteDelivery(deliveryId: $deliveryId)
          }
        `,
        variables: {
          deliveryId: id,
        },
      }).then((result) => {
          console.log("Resultado", result);
      });
      alert("Pedido Eliminado");
      window.location.href = window.location.href;
    },

    detallesServicio: function(id, event) {
      //Que no sea el boton de borrar
      if (event.path[0].textContent == "") return;
      console.log("Ver detalles de " + id);
      //Enviar a la pagina de detalles
      this.$emit("loadDetalles", id);
    },
  },
};
</script>

<style>

.servicesContainer h1 {
  margin-top: 50px;
  margin-bottom: 0;
}

.tablaPedidos {
  margin-top: 30px;
  margin-bottom: 10px;
  max-height: 430px;
  overflow-x: hidden;
  overflow-y: scroll;
  border: 3px solid rgb(128, 129, 81);
}
</style>