<template>

    <div class="formService">
        <div class="container_formService">
            <h2>{{accion}} Servicio</h2>
            <form v-on:submit.prevent="processAction" id="formulario">
                <input type="text" v-model="infoDelivery.usernameReceptor" placeholder="Username Destino" required>
                <br>

                <input type="text" v-model="infoDelivery.ciudadOrigen" placeholder="Ciudad Origen" required>
                <br>

                <input type="text" v-model="infoDelivery.direccionOrigen" placeholder="Dirección Origen" required>
                <br>

                <input type="text" v-model="infoDelivery.ciudadDestino" placeholder="Ciudad Destino" required>
                <br>

                <input type="text" v-model="infoDelivery.direccionDestino" placeholder="Direccion Destino" required>
                <br>

                <textarea name="descripcion" rows="4" v-model="infoDelivery.description" placeholder="Descripción"></textarea>
                <br>

                <input type="number" v-model="infoDelivery.value" min="10000" placeholder="Precio" v-if="userId==1">
                <br>

                <input type="text" v-model="infoDelivery.estado" placeholder="Estado" v-if="userId==1">
                <br>

                <input type="date" id="fecha" v-model="infoDelivery.pickUpDate" v-if="userId==1">
                <br>

                <input type="date" v-model="infoDelivery.deliverDate" v-if="userId==1">
                <br>

                <button id="btnEnviar" type="submit">Enviar</button>
            </form>
        </div>

    </div>

</template>

<script>
import gql from 'graphql-tag'
import jwt_decode from "jwt-decode";

export default {
    name: "formServices",
    data: function(){
        return {
            username: localStorage.getItem("username") || "",
            userId: jwt_decode(localStorage.getItem("token_refresh")).user_id,
            accion: window.location.href.split("/")[4],
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



    methods: {
        processAction: async function(){
            if (
                localStorage.getItem("token_access") === null ||
                localStorage.getItem("token_refresh") === null
            ){
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
                console.log(error);
                this.$emit("logOut");
                return;
            });

            if(this.accion=="Nuevo") await this.newService();
            else await this.updateService();
            console.log(this.userId);
            if(this.userId == 1) window.location.href = window.location.href.split("services/")[0]+"administration/services";
            else window.location.href = window.location.href.split("services/")[0]+"user/services";

        },

        newService: async function(){
            console.log("Creando");

            await this.$apollo
            .mutate({
                mutation: gql`
                    mutation($delivery: DeliveryInput!) {
                        createDelivery(delivery: $delivery) {
                            id
                            usernameEmisor
                            usernameReceptor
                            ciudadOrigen
                            ciudadDestino
                            estado
                            value
                        }
                    }
                `,
                variables: {
                    delivery: this.infoDelivery,
                },
            })
            .then((result) => {
                alert("Servicio creado satisfactoriamente");
            })
            .catch((error) => {
                console.log(error);
                alert("Hay un error en la información. Por favor verifique e intente de nuevo");
            });
        },

        updateService: async function(){
            console.log("Actualizando");

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
                            description
                            estado
                            value
                        }
                    }
                `,
                variables: {
                    delivery: this.infoDelivery,
                },
            })
            .then((result) => {
                alert("Servicio editado satisfactoriamente");
            })
            .catch((error) => {
                console.log(error);
                alert("Hay un error en la información. Por favor verifique e intente de nuevo");
            });
        },

        loadInfo: async function(){
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
                    deliveryId: window.location.href.split("edit/")[1],
                },
            })
            .then((result) => {
                console.log(result);
                console.log(this.username+" vs "+result.data.deliveryById.usernameEmisor);
                console.log(this.userId+" vs 1");
                if(this.username != result.data.deliveryById.usernameEmisor && this.userId != 1){
                    alert("No tienes permisos para editar este pedido");
                    window.location.href = window.location.href.split("services/")[0]+"user/services";
                }
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
                }
                if(this.infoDelivery.pickUpDate) this.infoDelivery.pickUpDate = this.infoDelivery.pickUpDate.substring(0,10);
                if(this.infoDelivery.deliverDate) this.infoDelivery.deliverDate = this.infoDelivery.deliverDate.substring(0,10);
            })
            .catch((error) => {
                console.log(error);
                alert("Error trayendo info");
            });
        }
    },

    created: function(){
        console.log(this.accion);
        console.log(window.location.href.split("/"));
        if(this.accion=="edit"){
            this.loadInfo();
        }
        if(this.accion=="edit") this.accion = "Editar";
        else this.accion = "Nuevo";
    }
}
</script>

<style>

.formService {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}

.container_formService {
    text-align: center;
    width: 30%;
}

.container_formService h2 {
    margin-top: 30px;
    margin-bottom: 15px;
}

#formulario textarea{
    width: 98%;
}

</style>