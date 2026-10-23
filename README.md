# clasecuatro

## Vista de colchones

La vista está disponible en `/colchones` y consume la API de colchones. Por defecto usa
`http://192.168.1.254:3000`, la URL base indicada en la documentación del API, accesible desde
la red local correspondiente.

Para usar una instancia local del API, inicia el servidor según el
[README de fabricacolchones](https://github.com/solincosta/fabricacolchones#readme)
y crea `.env.local` en la raíz de este proyecto:

```env
VUE_APP_API_URL=http://localhost:3000
```

Reemplaza esa URL si el API está desplegado en otra dirección. Inicia Vue con `npm run serve`
y abre la opción **COLCHONES** en el menú. Si no puedes conectar con la URL documentada,
comprueba que estés en la red local donde está desplegado el API.

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### USE ESTAS LINEAS DE INSTALACION DURANTE EL PROYECTO
```
#Router
vue add router

#Instalar Axios
npm install --save axios vue-axios

#Instalar PrimeVue
npm install primevue @primeuix/themes
```
