<template>
  <div class="informacionH">
    <h1>Detalles del pedido: {{ deliveryById.id }}</h1>
    <form v-on:submit.prevent="processLogInUser">
      <input
        type="text"
        v-model="idCon"
        placeholder="#ID"
        style="margin: 10px 20px 0px 20px;"
      />
      <button
        type="submit"
        v-on:click="consultaId(idCon)"
        style="margin-top: 10px; height: 46px;"
      >
        Consultar
      </button>
    </form>
  </div>
  <div class="informacion">
    <div class="detalles">
      <h2>
        Usuario Emisor:
        <span>{{ deliveryById.usernameEmisor }}</span>
      </h2>

      <h2>
        Usuario Receptor:
        <span>{{ deliveryById.usernameReceptor }}</span>
      </h2>

      <h2>
        Dirección de Origen del paquete:
        <span
          >{{ deliveryById.ciudadOrigen }} -
          {{ deliveryById.direccionOrigen }}</span
        >
      </h2>

      <h2>
        Dirección de Destido del paquete:
        <span
          >{{ deliveryById.ciudadDestino }} -
          {{ deliveryById.direccionDestino }}</span
        >
      </h2>

      <h2>
        Valor del envío:
        <span>${{ deliveryById.value }}</span>
      </h2>
    </div>

    <div class="detalles2">
      <h2>
        Descripción del paquete:
        <span>{{ deliveryById.description }}</span>
      </h2>

      <h2>
        Estado de envío:
        <span>{{ deliveryById.estado }}</span>
      </h2>

      <h2>
        Fecha de recogida:
        <span v-if="deliveryById.pickUpDate">{{
          deliveryById.pickUpDate
            .substring(0, 10)
            .split("-")
            .reverse()
            .join("/")
        }}</span>
        <span v-if="!deliveryById.pickUpDate && most"
          >El pedido aún no se recoge</span
        >
      </h2>

      <h2>
        Fecha de entrega:
        <span v-if="deliveryById.deliverDate">{{
          deliveryById.deliverDate
            .substring(0, 10)
            .split("-")
            .reverse()
            .join("/")
        }}</span>
        <span v-if="!deliveryById.pickUpDate && most"
          >El pedido aún no se entrega</span
        >
      </h2>

      <h2>
        PQRS Realizados:
        <span>{{ deliveryById.pqr }}</span>
      </h2>
    </div>
  </div>
  <div class="informacionImg">
    <h1>Estado del pedido:</h1>
  </div>
  <div class="image">
    <img src="../assets/recogida.png" alt="Recogida" class="i1" v-show="most" />
    <p>---------</p>
    <img
      src="../assets/despacho.png"
      alt="Despachado"
      class="i2"
      v-show="most2"
    />
    <p>---------</p>
    <img src="../assets/bodega.png" alt="En Bodega" class="i3" v-show="most3" />
    <p>---------</p>
    <img
      src="../assets/reparto.png"
      alt="En Reparto"
      class="i4"
      v-show="most4"
    />
    <p>---------</p>
    <img
      src="../assets/entregado.png"
      alt="Entregado"
      class="i5"
      v-show="most5"
    />
  </div>
  <div class="cajaB">
    <button
      class="btnE"
      v-if="permisos()"
      v-show="mostButton"
      v-on:click="editService()"
    >
      Editar pedido
    </button>
    <button
      class="btnC"
      v-if="permisos()"
      v-show="mostButton"
      v-on:click="deleteService()"
    >
      Cancelar pedido
    </button>
  </div>
</template>

<script>
import gql from "graphql-tag";
import jwt_decode from "jwt-decode";

export default {
  name: "ServiceId",

  data: function() {
    return {
      userId: -1,
      deliveryById: {
        id: null,
        usernameEmisor: "",
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
      most: false,
      most2: false,
      most3: false,
      most4: false,
      most5: false,
      mostButton: false,
      idCon: "",
    };
  },

  methods: {
    consultaId: async function(id) {
      this.most = false;
      (this.most2 = false),
        (this.most3 = false),
        (this.most4 = false),
        (this.most5 = false),
        (this.mostButton = false),
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
              deliveryId: id,
            },
          })
          .then((result) => {
            this.deliveryById = {
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
            if (this.deliveryById.pickUpDate)
              this.deliveryById.pickUpDate = this.deliveryById.pickUpDate.substring(
                0,
                10
              );
            if (this.deliveryById.deliverDate)
              this.deliveryById.deliverDate = this.deliveryById.deliverDate.substring(
                0,
                10
              );
            this.mostrar();
          })
          .catch((error) => {
            console.log(error);
            if (this.idCon == "" || this.idCon == null)
              alert("Por favor ingrese un codigo de identificación de pedido");
            else
              alert(
                "No se encontro un pedido con el codifo de indentificación: " +
                  this.idCon
              );
            this.idCon = "";
          });
    },

    permisos: function() {
      console.log("Permisos");
      if (localStorage.getItem("username"))
        this.userId = jwt_decode(localStorage.getItem("token_refresh")).user_id;
      return (
        this.deliveryById.usernameEmisor == localStorage.getItem("username") ||
        this.userId == 1
      );
    },

    editService: function(id) {
      console.log("Editar servicio");
      this.$router.push({
        name: "formServicesEdit",
        params: { deliver: this.idCon },
      });
    },

    deleteService: function(id) {
      let confirmar = confirm("¿Está seguro que desea eliminar este servicio?");
      if (!confirmar) return;
      console.log("Eliminando servicio");
      this.$apollo.mutate({
        mutation: gql`
          mutation($deliveryId: String!) {
            deleteDelivery(deliveryId: $deliveryId)
          }
        `,
        variables: {
          deliveryId: window.location.href.split("detalles/")[1],
        },
      });
      alert("Pedido Eliminado");
      if (this.userId == 1)
        window.location.href =
          window.location.href.split("user/")[0] + "administration/services";
      else
        window.location.href =
          window.location.href.split("user/")[0] + "user/services";
    },

    mostrar() {
      console.log(this.deliveryById.estado);
      let estado2 = this.deliveryById.estado;
      if (estado2 == "Por Recoger" || this.userId == 1) {
        this.most = true;
        console.log("Estado 1");
        this.mostButton = true;
      }
      if (estado2 == "En Despacho") {
        this.most = true;
        this.most2 = true;
        console.log("Estado 2");
      }
      if (estado2 == "En Bodega") {
        this.most = true;
        this.most2 = true;
        this.most3 = true;
        console.log("Estado 3");
      }
      if (estado2 == "En Reparto") {
        this.most = true;
        this.most2 = true;
        this.most3 = true;
        this.most4 = true;
        console.log("Estado 4");
      }
      if (estado2 == "Entregado") {
        this.most = true;
        this.most2 = true;
        this.most3 = true;
        this.most4 = true;
        this.most5 = true;
        console.log("Estado 5");
      }
    },
  },
};
</script>

<style>
.informacionH {
  margin: 1vw;
  padding: 0%;
  width: 100%;
  height: 10%;

  display: flex;
  flex-direction: row;
  margin-top: 30px;
  margin-bottom: 30px;
}

.informacionH h1 {
  font-size: 2.5vw;
  color: #283747;
}

.informacionH form {
  display: flex;
  flex-direction: row;
  margin-top: 20px;
}

.informacion {
  margin: 0;
  padding: 0%;
  width: 94vw;
  height: 15vw;
  margin-left: 2.5%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  flex-direction: row;

  border: 3px solid rgba(0, 0, 0, 0.3);
  border-radius: 20px;
}

.informacion h2 {
  font-size: 1.8vw;
  color: #283747;
}

.informacion span {
  color: crimson;
  font-weight: bold;
}

.detalles {
  padding: 10px 20px;
  margin: 0 0 0 0;
  width: 50vw;
  order: 0;
}

.detalles h2 {
  font-size: 1.4vw;
  color: #283747;
}

.detalles2 {
  padding: 10px 20px;
  margin: 0 0 0px 0;
  width: 40vw;
  order: 1;
}

.detalles2 h2 {
  font-size: 1.4vw;
  color: #283747;
}

/* Imágenes */

.informacionImg {
  padding: 0;
  margin-left: 40vw;
  width: 30vw;
  color: #283747;
}
.image {
  margin: 1vw;
  padding: 0%;
  width: 94vw;
  height: 7vw;
  margin-left: 2.5%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  flex-direction: row;
}
.image p {
  font-size: 2.5vw;
  padding: 0;
}

.i1 {
  padding: 0;
  margin: 1vw;
  width: 5vw;
}
.i2 {
  padding: 0;
  margin: 1vw;
  width: 4.7vw;
}
.i3 {
  padding: 0;
  margin: 1vw;
  width: 5vw;
}
.i4 {
  padding: 0;
  margin: 1vw;
  width: 5vw;
}
.i5 {
  padding: 0;
  margin: 1vw;
  width: 5.3vw;
}

/* Botones */

.cajaB {
  display: flex;
  margin-right: 5vw;
  justify-content: right;
}
.btnE {
  width: 110px;
  height: 40px;
  padding: 0;
  margin-top: 0;
}
.btnC {
  width: 120px;
  height: 40px;
  padding: 0;
  margin-top: 0;
  background-color: brown;
}

.btnC:hover {
  color: #ffffff;
  background: #e90000;
  border: 1px solid #1f1700;
}
</style>
