<template>
  <div class="hello">
    <div class="container-fluid">
      <div class="row">
        <div class="col">
          <button type="button" class="btn btn-primary" @click="modalToggle">
            Agregar Nuevo Usuario
          </button>
        </div>
      </div>
      <div class="row">
        <div class="col">
          <table class="table table-striped">
            <thead>
              <tr>
                <th scope="col">ID</th>
                <th scope="col">Nombre</th>
                <th scope="col">Edad</th>
                <th scope="col">Email</th>
                <th scope="col">Acciones</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="user in users" :key="user.id">
                <td>{{ user.id }}</td>
                <td>{{ `${user.first_name} ${user.last_name}` }}</td>
                <td>
                  {{ user?.birthday ? calcularEdad(user?.birthday) : "" }}
                </td>
                <td>{{ user.email }}</td>
                <td>
                  <button
                    type="button"
                    class="btn btn-warning"
                    @click="modificar(user)"
                  >
                    Modificar
                  </button>
                  <button
                    type="button"
                    class="btn btn-danger"
                    @click="eliminar(user)"
                  >
                    Eliminar
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- modal -->
    <div>
      <div>
        <div
          ref="modal"
          class="modal fade"
          :class="{ show: active, 'd-block': active }"
          tabindex="-1"
          role="dialog"
        >
          <div class="modal-dialog" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title">Datos del Usuario</h5>
                <button
                  type="button"
                  class="close"
                  data-dismiss="modal"
                  aria-label="Close"
                  @click="modalToggle"
                >
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <div class="modal-body">
                <form
                  class="row g-3 needs-validation"
                  @submit.prevent="onSubmit"
                >
                  <div class="col-md-6">
                    <label for="validationCustom01" class="form-label"
                      >Nombres</label
                    >
                    <input
                      type="text"
                      class="form-control"
                      v-model="user.first_name"
                      required
                    />
                  </div>
                  <div class="col-md-6">
                    <label for="validationCustom02" class="form-label"
                      >Apellidos</label
                    >
                    <input
                      type="text"
                      class="form-control"
                      id="validationCustom02"
                      v-model="user.last_name"
                      required
                    />
                  </div>
                  <div class="col-md-7">
                    <label for="validationCustomUsername" class="form-label"
                      >Email</label
                    >
                    <div class="input-group has-validation">
                      <span class="input-group-text" id="inputGroupPrepend"
                        >@</span
                      >
                      <input
                        type="email"
                        class="form-control"
                        id="validationCustomUsername"
                        aria-describedby="inputGroupPrepend"
                        v-model="user.email"
                        required
                      />
                    </div>
                  </div>
                  <div class="col-md-5">
                    <label for="validationCustom03" class="form-label"
                      >Fecha Nacimiento</label
                    >
                    <input
                      type="date"
                      class="form-control"
                      id="validationCustom03"
                      v-model="user.birthday"
                      required
                    />
                  </div>
                  <div class="col-12">
                    <button class="btn btn-primary" type="submit">
                      Guardar
                    </button>
                  </div>
                </form>
              </div>
            </div>
          </div>
        </div>
        <div v-if="active" class="modal-backdrop fade show"></div>
      </div>
    </div>
    <!-- modal -->
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "Usuarios",
  props: {},
  data() {
    return {
      active: false,
      users: [],
      user: {
        id: null,
        first_name: "",
        last_name: "",
        birthday: "",
        email: "",
        avatar: "",
      },
    };
  },
  mounted() {
    axios.get("https://reqres.in/api/users?page=1").then((response) => {
      if (localStorage.getItem("users")) {
        this.users = JSON.parse(localStorage.getItem("users"));
      } else if (response.status === 200) {
        this.users = response.data.data;
        localStorage.setItem("users", JSON.stringify(this.users));
      }
    });
  },
  methods: {
    modificar(user) {
      this.user = user;
      this.modalToggle();
    },
    eliminar(user) {
      const index = this.users.findIndex((usr) => usr.id === user.id);
      this.$swal({
        title: `¿Seguro de eliminar a ${user.first_name} ${user.last_name}?`,
        showCancelButton: true,
        confirmButtonText: "Eliminar",
      }).then((result) => {
        if (result.isConfirmed) {
          this.users.splice(index, 1);
          localStorage.setItem("users", JSON.stringify(this.users));
        }
      });
    },
    modalToggle() {
      const body = document.querySelector("body");
      this.active = !this.active;
      this.active
        ? body.classList.add("modal-open")
        : body.classList.remove("modal-open");
    },
    onSubmit() {
      this.$swal({
        title: `¿Seguro de guardar datos?`,
        showCancelButton: true,
        confirmButtonText: "Guardar",
      }).then((result) => {
        if (result.isConfirmed) {
          if (this.user.id) {
            const index = this.users.findIndex(
              (user) => user.id === this.user.id
            );
            this.users[index] = this.user;
          } else {
            this.users.sort((a, b) => {
              let rs = 0;
              if (a.id > b.id) rs = 1;
              if (a.id < b.id) rs = -1;
              return rs;
            });
            const lastUser = this.users[this.users.length - 1];
            console.log({ lastUser });
            this.user.id = lastUser.id + 1;
            this.users.push(this.user);
          }
          localStorage.setItem("users", JSON.stringify(this.users));
          this.user = {
            id: null,
            first_name: "",
            last_name: "",
            birthday: "",
            email: "",
            avatar: "",
          };
          this.modalToggle();
        }
      });
    },
    calcularEdad(fecha) {
      let hoy = new Date();
      let cumpleanos = new Date(fecha);
      let edad = hoy.getFullYear() - cumpleanos.getFullYear();
      let m = hoy.getMonth() - cumpleanos.getMonth();

      if (m < 0 || (m === 0 && hoy.getDate() < cumpleanos.getDate())) {
        edad--;
      }

      return edad;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
