<template>
   <div>
       <md-table v-model="searched" md-sort="name" md-sort-order="asc" md-card md-fixed-header>
      <md-table-toolbar>
        <div class="md-toolbar-section-start">
          <h1 class="md-title">Clientes</h1>
        </div>
        <md-field md-clearable class="md-toolbar-section-end">
          <md-input placeholder="Buscar por nombre..." v-model="search" @input="searchOnTable" />
        </md-field>
      </md-table-toolbar>

      <md-table-empty-state
        md-label="Usuario no encontrado"
        :md-description="`Nombre '${search}' no encontrado. Intenta con otro nombre o crea un nuevo usuario.`">
        <md-button class="md-primary md-raised" @click="showDialog = true">Crear nuevo usuario</md-button>
      </md-table-empty-state>
      <md-table-row slot="md-table-row" slot-scope="{ item }" >
        <md-table-cell md-label="Nombre">{{ item.name }}</md-table-cell>
        <md-table-cell md-label="Opción">
            <md-button class="md-primary md-icon-button"  @click="openDialogShop(item.id)">
                <md-icon>add_shopping_cart</md-icon>
            </md-button>
            <md-button class="md-icon-button" style="background-color:#00E676; color:white;" @click="openDialogVentas(item.id)">
                <md-icon>monetization_on</md-icon>
            </md-button>
        </md-table-cell>
      </md-table-row>
    </md-table>
     <!-- dialog register -->
      <md-dialog :md-active.sync="showDialog">
      <md-dialog-title>Nuevo Cliente</md-dialog-title>
      <form v-on:submit.prevent="register()">
      <md-dialog-content>
        <md-field>
            <label>Nombre</label>
            <md-input v-model="nombreCliente" required></md-input>
        </md-field>
      </md-dialog-content>
      <md-dialog-actions>
        <md-button class="md-primary" @click="showDialog = false">Cerrar</md-button>
        <md-button type="submit" class="md-primary">Guardar</md-button>
      </md-dialog-actions>
      </form>
    </md-dialog>
    <!-- dialog register end -->
    <!-- dialog shop -->
    <md-dialog :md-active.sync="showDialogShop">
      <md-dialog-title>Realizar Venta</md-dialog-title>
      <form v-on:submit.prevent="saveShop()">
      <md-dialog-content >
        <md-content class="md-scrollbar">
        <md-steppers :md-active-step.sync="active" md-linear>
        <md-step id="first" md-label="Seleccionar Sucursal" :md-done.sync="first">
 
        <md-field>
          <label for="movie">Sucursal</label>
          <md-select v-model="selectSucursal" name="sucursal" id="sucursal">
              
            <md-option v-for="(sucursal, index) in sucursales" :key="index" :value="index">{{sucursal.name}}</md-option>
            
          </md-select>
        </md-field>
            <div style="text-align:right;">
            <md-button class="md-raised" @click="cancelarCompra">Cancelar</md-button>
            <md-button class="md-raised md-primary" @click="setDone('first', 'second')">Continuar</md-button>
            </div>
        </md-step>

        <md-step id="second" md-label="Productos" :md-done.sync="second">
             <md-field>
                <label for="movies">Productos</label>
                <md-select v-model="selectedProductos" name="productos" id="productos" multiple>
                <md-option v-for="(producto, index) in productos" :key="index" :value="producto.title">{{producto.title}}</md-option>
                </md-select>
            </md-field>
            <table style="width:100%">
            <tr>
                <th width="25%"></th>
                <th width="25%"></th> 
                <th width="25%"></th>
                <th width="25%"></th>
            </tr>
            <tr  v-for="(producto, index) in selectedProductos" :key="index">
                <td>
                    <md-field>
                        <label>Nombre</label>
                        
                        <md-input disabled :value="producto"></md-input>
                    </md-field>
                </td>
                <td>
                    <md-field>
                        <label>Cantidad</label>
                        <md-input v-model="productosVenta[index].quantity" @keyup="calcularTotal"></md-input>
                    </md-field>
                </td> 
                <td>
                    <md-field>
                        <label>Precio</label>
                        <md-input v-model="productosVenta[index].price" @keyup="calcularTotal"></md-input>
                        
                    </md-field>
                </td>
                <td>
                    <md-field>
                        <label>Subtotal: {{productosVenta[index].quantity * productosVenta[index].price}}</label>
                    </md-field>
                </td>
            </tr>
            <tr v-if="showTotal">
                <td colspan="3" style="text-align:right;"><h4>TOTAL:</h4></td>
                <td><md-field>
                        <label style="color:black;">{{totalVenta}} {{moneda}}</label>
                    </md-field></td>
            </tr>
            </table>
            
            <div style="text-align:right;">
            <md-button class="md-raised" @click="cancelarCompra">Cancelar</md-button>
            <md-button class="md-raised md-primary" @click="saveShop">Confirmar Venta</md-button>
            </div>
        </md-step>
        </md-steppers>
        </md-content>
      </md-dialog-content>
      
      </form>
    </md-dialog>
    <!--dialog shop end -->
    <!-- dialog ventas -->
      <md-dialog :md-active.sync="showDialogVentasCliente">
      <md-dialog-title>Ventas</md-dialog-title>
      <form >
      <md-dialog-content>
        <table style="width:100%">
            <tr>
                <th width="25%">Cantida Productos</th>
                <th width="25%">Total</th> 
                <th width="25%"></th>
                <th width="25%"></th>
            </tr>
            <tr  v-for="(venta, index) in clienteVentas.ventas" :key="index">
                <td>
                    {{venta.productos.length}}
                </td>
                <td>
                    {{ totalMostrar(venta) }}
                </td> 
                <td>
                    
                </td>
                <td>
                    
                </td>
            </tr>
            </table>
      </md-dialog-content>
      <md-dialog-actions>
        <md-button class="md-primary" @click="showDialogVentasCliente = false">Cerrar</md-button>
        
      </md-dialog-actions>
      </form>
    </md-dialog>
    <!-- dialog ventas end -->
    <md-snackbar class="snackbarColor" md-position="center" :md-duration="4000" :md-active.sync="showSnackbar" md-persistent>
      <span>{{textSnack}}</span>
    </md-snackbar>
   </div>
</template>

<script>
import axios from 'axios'
    const toLower = text => {
        return text.toString().toLowerCase()
    }
    const searchByName = (items, term) => {
    if (term) {
      return items.filter(item => toLower(item.name).includes(toLower(term)))
    }
        return items
    }
    export default {
        data () {
            return {
                sucursales: [],
                clientes: [],
                searched: [],
                productos: [],
                showSnackbar: false,
                showRegister: false,
                showDialog: false,
                showTotal: false,
                showDialogShop: false,
                search: '',
                selectedClient: null,
                nombreCliente: '',
                active: 'first',
                first: false,
                second: false,
                selectSucursal: '',
                producto: {
                    name: '',
                    quantity: 0,
                    price: 0,
                    subtotal: 0,
                },
                selectedProductos: [],
                productosVenta: [],
                totalVenta: 0,
                selectClient: 0,
                textSnack: '',
                showDialogVentasCliente: false,
                clienteVentas: {},
                moneda: ''
            }
        },
        created () {
            this.getClientes()
            this.getSucursales()
            this.getProducts()
        },
        watch:{
            selectedProductos (val)
            {
                
                this.productosVenta = this.selectedProductos.map(function(e){return {title: e, price: 0, quantity:0}})
                if(this.productosVenta !== null)
                {
                    this.showTotal = true;
                }else{
                    this.showTotal = false
                }
                
            },
            selectSucursal (val)
            {
                // console.log(this.sucursales.findIndex(val))
                console.log(this.sucursales.find(x => x.id === val))
                this.moneda = this.sucursales.find(x => x.id === val).code
                
            }

        },
        methods: {
            getProducts ()
            {
                axios.get('http://fakerestapi.azurewebsites.net/api/books').then((response) => {
                    this.productos = response.data.map(function (e){ return {title: e.Title, quantity: 0, price: 0}})
                })
            },
            getSucursales()
            {
                axios.get('https://restcountries.eu/rest/v2/region/americas')
                    .then((response) => {
                        console.log(response.data)
                        for (let i = 0; i < response.data.length; i++) {
                            this.sucursales.push({id: i, name: response.data[i].name, code: response.data[i].currencies[0].code }) 
                        }
                    }).catch((error)=>{
                        console.log(error)
                    })
            },
            getClientes()
            {
                axios.get('https://jsonplaceholder.typicode.com/users')
                    .then((response) => {
                        for (let i = 0; i < response.data.length; i++) {
                            this.searched.push({id: i,name: response.data[i].name})
                            this.clientes.push({id: i,name: response.data[i].name, ventas:[]})
                        }
                    }).catch((error)=>{
                        console.log(error)
                    })
            },
            register()
            {
                //console.log(this.nombreCliente)
                //this.showRegister = true;
                this.clientes.push({name: this.nombreCliente})
                this.search = this.nombreCliente
                this.showDialog = false;
                this.textSnack = 'Se ha registrado con éxito'
                this.showSnackbar = true;
                //alert('Se ha agregado con exito')
            },
            searchOnTable () {
                this.searched = searchByName(this.clientes, this.search)
            },
            openDialogShop (id)
            {
                this.showDialogShop = true
                this.selectClient = id
                
            },
            saveShop()
            {
                console.log(this.clientes[this.selectClient].ventas)
                var numeroVenta = this.clientes[this.selectClient].ventas.length + 1;
                this.clientes[this.selectClient].ventas.push({id: numeroVenta, productos: this.productosVenta})
                this.textSnack = 'Se ha registrado con éxito la venta'
                this.showSnackbar = true;
                this.showDialogShop = false;
            },
            setDone (id, index) {
                this[id] = true

                if (index) {
                this.active = index
                }
            },
            prueba ()
            {
                console.log(this.selectedProductos)
            },
            calcularTotal ()
            {
                var sumaTotal = 0;
                for (let i = 0; i < this.productosVenta.length; i++) {
                    sumaTotal += this.productosVenta[i].price * this.productosVenta[i].quantity
                }
                this.totalVenta = sumaTotal
            },
            openDialogVentas (id)
            {
                console.log(this.clienteVentas = this.clientes.find(x => x.id === id))
                
                this.showDialogVentasCliente = true
            },
            totalMostrar(venta)
            {
                var totalventa = 0
                for (let i = 0; i < venta.productos.length; i++) {
                    totalventa += venta.productos[i].price *  venta.productos[i].quantity 
                    console.log(totalventa)
                }
                
                return totalventa
            },
            cancelarCompra ()
            {
                this.selectSucursal = ''
                this.selectedProductos =  []
                this.productosVenta = []
                this.active = 'first'
                this.first = false
                this.second = false
                this.showDialogShop = false
            }

        }
    }
</script>

<style lang="scss" scoped>
.snackbarColor {
    background-color:green !important;
    color:white !important;
}
  .md-content {
    
    max-height: 500px;
    overflow-y: auto;
  }
  .md-layout-item {
    width: 25% !important;
  }

</style>

