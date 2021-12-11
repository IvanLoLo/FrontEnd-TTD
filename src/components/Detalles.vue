<template>
  <div class="informacionH">
    <h1>Detalles del pedido: {{ deliveryById.id }}</h1>
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
        <span v-if="deliveryById.pickUpDate">{{ deliveryById.pickUpDate.substring(0, 10) }}</span>
        <span v-if="!deliveryById.pickUpDate">El pedido aun no se recoge</span>
      </h2>

      <h2>
        Fecha de entrega:
        <span v-if="deliveryById.deliverDate">${{ deliveryById.deliverDate.substring(0, 10) }}</span>
        <span v-if="!deliveryById.pickUpDate">El pedido aun no se entrega</span>
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
</template>

<script>
import gql from "graphql-tag";
import jwt_decode from "jwt-decode";

export default {
  name: "detalles",

  data: function() {
    return {
      userId: jwt_decode(localStorage.getItem("token_refresh")).user_id,
      deliveryById: [],
      most: false,
      most2: false,
      most3: false,
      most4: false,
      most5: false,
    };
  },
  apollo: {
    deliveryById: {
      query: gql`
        query deliveryById($deliveryId: String!) {
          deliveryById(deliveryId: $deliveryId) {
            id
            usernameEmisor
            usernameReceptor
            ciudadOrigen
            direccionOrigen
            ciudadDestino
            direccionDestino
            value
            description
            estado
            pickUpDate
            deliverDate
            pqr
          }
        }
      `,
      variables() {
        console.log(this.deliveryById.estado);
        if (this.deliveryById.estado != undefined) {
          localStorage.setItem("estados", this.deliveryById.estado);
          this.mostrar();
        }
        return {
          deliveryId: window.location.href.split("detalles/")[1],
        };
      },
    },
  },
  methods: {
    mostrar() {
      console.log(this.deliveryById.estado);
      //let estado2 = localStorage.getItem("estados").style;
      let estado2 = this.deliveryById.estado;
      let estadoImg = document.getElementById("i1");
      let estadoImg2 = document.getElementsByClassName("i2");
      let estadoImg3 = document.getElementsByClassName("i3");
      let estadoImg4 = document.getElementsByClassName("i4");
      let estadoImg5 = document.getElementsByClassName("i5");

      /*       document.addEventListener("DOMContentLoaded", function() {
        if (estado2 == "Por recoger") {
          estadoImg2.style.display = "";
          console.log("Estado 1");
        }
      }); */
      if (estado2 == "Por recoger") {
        this.most = true;
        console.log("Estado 1");
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
      } /* else {
        estadoImg.style.display = "none";
        estadoImg2.style.display = "none";
        estadoImg3.style.display = "none";
        estadoImg4.style.display = "none";
        estadoImg5.style.display = "none";
      } */
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
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.informacionH h1 {
  font-size: 3vw;
  color: #283747;
}

.informacion {
  margin: 0;
  padding: 0%;
  width: 94vw;
  height: 18vw;
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
  font-size: 2vw;
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
  font-size: 1.6vw;
  color: #283747;
}

.detalles2 {
  padding: 10px 20px;
  margin: 0 0 0px 0;
  width: 40vw;
  order: 1;
}

.detalles2 h2 {
  font-size: 1.6vw;
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
  width: 5.5vw;
}
.i2 {
  padding: 0;
  margin: 1vw;
  width: 5.5vw;
}
.i3 {
  padding: 0;
  margin: 1vw;
  width: 5.5vw;
}
.i4 {
  padding: 0;
  margin: 1vw;
  width: 5.5vw;
}
.i5 {
  padding: 0;
  margin: 1vw;
  width: 5.5vw;
}
</style>
