<template>
    <div class="login">
        <input v-model="usuario.documento" type="text" placeholder="Digite usuario">
        <input v-model="usuario.clave" type="password" placeholder="Digite contraseña">
        <button @click="iniciarSesion">Iniciar sesión</button>
    </div>
</template>

<script>
export default {
    data() {
        return {
            usuario: {
                documento: "",
                clave: ""
            }
        }
    },

    methods: {
        iniciarSesion: async function () {
            await this.axios
                .post('https://solincosta.com/apiv2.php?action=iniciarsesion',
                    {
                        documento: this.usuario.documento,
                        clave: this.usuario.clave
                    }
                )
                .then(response => {
                    if (response.data.status) {
                        localStorage.setItem("token", response.data.token);
                        this.$router.push("/destinos");
                    } else {
                        alert("ERROR");
                    }
                })
        },



    }
}
</script>

<style scope>
.login {
    width: 45%;
    display: flex;
    flex-direction: column;
    margin: 5rem auto;
    border: 2px solid #000;
    padding: 1rem;
    gap: 1px;
}

input,
button {
    height: 2rem;
    font-size: 1.5rem;
}
</style>