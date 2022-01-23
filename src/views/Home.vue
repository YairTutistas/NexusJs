<template lang="">
  <div>
  <!-- Alerta CRUD-->   
    <h1 align="center" class="mt-5">Lista de empleados</h1>
    <v-alert width="300px" type="success" @click="alertaCreacion = false" :value="alertaCreacion" transition="scale-transition">
      {{notificacion}}
    </v-alert>
    <!-- Fin alerta CRUD-->
    <!-- Alerta de validacion-->
    <v-alert width="300px" type="error"  @click="alertaValidacion = false" :value="alertaValidacion" transition="scale-transition"
    v-for="vals in val">
      {{vals}}
    </v-alert>   
    <!-- Fin alerta de validacion --> 
    <v-container>
    <!-- Modal formulario-->
    <template>
      <v-row>
        <v-dialog
          v-model="dialog"
          persistent
          max-width="700px"
        >
        <!-- Boton abrir modal-->
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              v-bind="attrs"
              v-on="on"
            >
              <v-icon>mdi-account-plus</v-icon>
              Crear
            </v-btn>
          </template>
        <!-- Fin boton-->
          <v-card>
          <!-- Titulo formulario-->
            <v-card-title>
              <span class="text-h5">{{titulos}}</span>
            </v-card-title>
          <!-- Fin titulo formulario-->
          <!-- Contenido modal-->
            <v-card-text>    
              <form>
                <v-container>
                  <v-text-field
                    v-model="nombre"
                    name="nombre"
                    label="Nombre completo"
                    :rules="rules"
                  ></v-text-field>
                  <v-text-field
                    v-model="correo"
                    name="correo"
                    label="Correo"
                    type="email"
                    :rules="rules"
                  ></v-text-field>        
                  <v-radio-group v-model="sexo">Sexo
                    <v-radio label="Hombre" value="M"></v-radio>
                    <v-radio label="Mujer" value="F"></v-radio>
                  </v-radio-group>
                  <v-select
                    :items="areas"
                    item-text="nombre"
                    item-value="id"
                    v-model="area"
                    item-color="success"
                    label="Areas"
                  ></v-select>
                  <v-textarea
                  v-model="descripcion"
                  name="descripcion"
                  label="Descripcion"
                  :rules="rules"
                  ></v-textarea>
                  <v-checkbox label="Deseo recibir boletin informativo" v-model="boletin"></v-checkbox>
                  <v-col>
                  <span>Roles</span>
                     <v-combobox
                        item-text="nombre"
                        item-value="id"
                        v-model="select"
                        :items="roles"
                        label="Roles"
                        multiple
                        outlined
                        dense></v-combobox>
                  </v-col>
                </v-container>
              </form>
            </v-card-text>
          <!-- Fin contenido-->
          <!-- Acciones modal-->
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue darken-1"
                text
                @click="dialog = false"
              >
                Close
              </v-btn>
              <v-btn
                color="blue darken-1"
                text
                @click="submit"
              >
                Save
              </v-btn>
            </v-card-actions>
          <!-- Fin acciones-->
          </v-card>
        </v-dialog>
      </v-row>
    </template>
    <!-- Fin modal formulario-->
    <!-- Tabla empleados-->
      <template>
      <!-- Tabla-->
        <v-simple-table>
        <!-- Conteido tabla-->
          <template>
            <thead>
              <tr>
                <th class="text-left">
                  ID
                </th>
                <th class="text-left">
                  Nombre
                </th>
                <th class="text-left">
                  Email
                </th>
                <th class="text-left">Sexo</th>
                <th class="text-left">Area</th>
                <th class="text-left">Boletin</th>
                <th class="text-left">Modificar/Eliminar</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="(item, index) in empleado"
                :key="index"
              >
                <td>{{ index }}</td>
                <td>{{ item.nombre }}</td>
                <td>{{ item.email }}</td>
                <td>{{ item.sexo }}</td>
                <td>{{ item.area }}</td>
                <td v-if="item.boletin > 0">SI</td>
                <td v-else="item.boletin > 0">NO</td>
                <td>
                <v-btn color="yellow darken-4"
                text
                small
                @click="edit(item.id)"
                fab><v-icon>mdi-account-edit</v-icon></v-btn>
                <v-btn color="red darken-4"
                text
                small
                @click="delet(item.id)"
                fab><v-icon>mdi-delete</v-icon></v-btn>
                </td>
              </tr>
            </tbody>
          </template>
        <!-- Contenido tabla-->
        </v-simple-table>
      <!-- Tabla-->
      </template>
    <!-- Fin tabla empleados-->
    </v-container>

  </div>
</template>
<script>
import  axios  from 'axios'
  export default {
    data () {
      return {
        dialog: false,
        titulo: -1,
        id: '',
        empleado: [],
        areas: [],
        nombre: '',
        boletin: false,
        correo: '',
        sexo: '',
        descripcion: '',
        area: '',
        roles: [],
        select: '',
        alertaCreacion: false,
        notificacion: '',
        alertaValidacion: false,
        val: [],
        rules: [
          value => !!value || 'Required.'
        ]
      }
    },
    computed:
    {
      titulos()
      {
        return this.titulo === -1 ? 'Crear empleado' : 'Actualizar empleado'
      }
    },
    mounted () {
      this.getEmpleados()
      this.getAreas()
      this.getRoles()
    },
    methods: {
      // Metodo traer emppleado y mostrarlo en variable empleado
      async getEmpleados ()
      {
        let empleados = await axios.get('http://127.0.0.1:8001/api/empleados')
        empleados.data.empleados.forEach(element =>this.empleado.push(element));
      },
      async getRoles ()
      {
        let roles = await axios.get('http://127.0.0.1:8001/api/roles')
        roles.data.rol.forEach(element => this.roles.push(element));
      },
      async getAreas ()
      {
        let areas = await axios.get('http://127.0.0.1:8001/api/areas')
        areas.data.areas.forEach(element => this.areas.push(element));
      },
      // Metodo guardar y actualizar
      submit()
      {
        // Condicion si es guardar o actualizar
        if(this.titulo === -1) {
          var data = {
            area: this.area,
            nombre: this.nombre,
            correo: this.correo,
            sexo: this.sexo,
            boletin: this.boletin === true ? 1 : 0,
            descripcion: this.descripcion,
            roles: this.select,
        }
          axios.post('http://127.0.0.1:8001/api/crearEmpleado', data)
          .then((resp) => {
            if(resp.data.msg != null){
              this.empleado = [];
              this.getEmpleados();
              this.notificacion = resp.data.msg;
              this.alertaCreacion = true;
              this.clear();
            } else if (resp.data.error != null){
              this.val = [];
              resp.data.errors.forEach(element => this.val.push(element));
              this.alertaValidacion = true;
              this.dialog = true;
            }

          })
          .catch((err) => {
          })
          this.dialog = false;
        // Condicion sino es crear para actualizar
        }else {
          var data = {
            id: this.id,
            area: this.area,
            nombre: this.nombre,
            correo: this.correo,
            sexo: this.sexo,
            boletin: this.boletin === true ? 1 : 0,
            descripcion: this.descripcion,
            roles: this.select,
        }
          axios.post('http://127.0.0.1:8001/api/actualizarEmpleado', data)
          .then((resp) => {
            this.empleado = [];
            this.getEmpleados();
            this.notificacion = resp.data.msg;
            this.alertaCreacion = true;
          })
          .catch((err) => {
          })
          this.dialog = false;
        }
        // Fin condicion

      },
      // Metodo editar
      edit (id)
      {
        // Asigno el id del empleado al Titulo para saber que voy actualizar en el metodo Submit
        this.titulo = id;
        var id = {empleadoId: id}
        axios.post('http://127.0.0.1:8001/api/editarEmpleado', id)
        .then((resp) => {
          // Pinto en los inputs la respuesta del backend
          this.id = resp.data.empleado[0].id;
          this.sexo = resp.data.empleado[0].sexo;
          this.correo = resp.data.empleado[0].email;
          this.nombre = resp.data.empleado[0].nombre;
          this.descripcion = resp.data.empleado[0].descripcion;
          this.boletin = resp.data.empleado[0].boletin > 0 ? true : false;
        })
        .catch((err) => {
        })
        this.dialog = true;
      },
      // Metodo eliminar segun el id
      delet (id)
      {
        var id = {empleadoId: id}
        axios.post('http://127.0.0.1:8001/api/eliminarEmpleado', id)
        .then((resp) => {
          this.empleado = [];
          this.getEmpleados();
          this.notificacion = resp.data.msg;
          this.alertaCreacion = true;
        })
        .catch((err) => {
        })
      },
    },
    // Metodo para limpiar inputs
    clear ()
    {
      this.area = '',
      this.nombre = '',
      this.correo = '',
      this.sexo = '',
      this.boletin = false,
      this.descripcion = ''
    }
  }
</script>
<style lang="">
  
</style>