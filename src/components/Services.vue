<template>
  <div class="servicesContainer">
    <h1 style="margin-top: 20px">
      Tus pedidos, <span>{{ username }}</span>
    </h1>
    <h2 style="margin-bottom: 5px;">Ordenar por:</h2>
    <select name="Filtro" v-model="filter">
      <option value="">Todos</option>
      <option value="Emisor">Emisor</option>
      <option value="Receptor">Receptor</option>
    </select>

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
          v-for="pedido in deliveriesByUsername"
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
              v-if="pedido.estado == 'Por Recoger' && permisos(pedido.usernameEmisor)"
              v-on:click="borrarServicio(pedido.id)"
            />
            <img
              src="http://assets.stickpng.com/images/5a5798809538462e5a82d431.png"
              title="No es posible eliminar este servicio"
              alt="Eliminar Servicio"
              style="filter: grayscale(1); cursor: not-allowed;"
              v-if="pedido.estado != 'Por Recoger' || !permisos(pedido.usernameEmisor)"
            />
          </td>
        </tr>
      </table>
    </div>

    <div class="newService">
      <h2>Crear nuevo Servicio</h2>
      <img
        src="https://cdn-icons-png.flaticon.com/512/117/117885.png"
        style="width: 4vw;"
        v-on:click="crearServicio()"
        alt="Imagen Nuevo"
      />
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
      deliveriesByUsername: {
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
    deliveriesByUsername: {
      query: gql`
        query($username: String!, $filter: String) {
          deliveriesByUsername(username: $username, filter: $filter) {
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
      variables() {
        return {
          username: this.username,
          filter: this.filter,
        };
      },
    },
  },

  methods: {

    permisos: function(emisor){
      return emisor == localStorage.getItem("username");
    },

    borrarServicio: function(id) {
      let confirmar = confirm("¿Está seguro que desea cancelar este servicio?");
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
      });
      alert("Pedido Eliminado");
      window.location.href = window.location.href;
    },

    crearServicio: function() {
      console.log("Toca crear un domicilio");
      this.$emit("loadFormNew");
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
.servicesContainer {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.servicesContainer select {
  width: 13%;
}

.tablaPedidos {
  margin-top: 30px;
  margin-bottom: 10px;
  max-height: 230px;
  overflow-x: hidden;
  overflow-y: scroll;
  border: 3px solid rgb(128, 129, 81);
}

.tablaPedidos table {
  text-align: center;
}

.tablaPedidos table td,
.tablaPedidos table th {
  border: 1px solid #ddd;
  padding: 8px;
}

.tablaPedidos table th {
  background-color: #9c9d63;
}

.tablaPedidos table th {
  cursor: default;
}

.tablaPedidos table tr {
  cursor: pointer;
}

.tablaPedidos table tr:nth-child(even) {
  background-color: #9c9d6323;
}

.tablaPedidos table tr:hover {
  background-color: #9c9d6359;
}

#deleteService {
  padding: 0;
  padding-right: 3px;
  padding-left: 3px;
}

#deleteService img {
  width: 28px;
  margin: 0;
}

.newService {
  text-align: center;
}

.newService img {
  margin: -12px;
  cursor: pointer;
}
</style>
