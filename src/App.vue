<template>
  <section>
    <div class="container-fluid">
      <div class="row align-items-center full">
        <div class="col">
          <div class="row justify-content-center pt-5">
            <div class="col-auto">
              <h1 class="h2">Seleccione su archivo:</h1>
            </div>
          </div>

          <div class="row justify-content-center">
            <div class="col-auto form-group mt-2">
              <input
                type="file"
                class="form-control-file"
                id="fileInputId"
                @change="handleFileSelect"
                accept=".xlsx,.xls"
              />
            </div>
          </div>

          <div class="row justify-content-center">
            <div class="col-lg-3 col-md-4 col-sm-5 col-6 pb-5 pt-3">
              <button
                class="btn btn-block btn-primary"
                v-if="excel"
                @click="enviar()"
              >
                Enviar
              </button>
              <button class="btn btn-block btn-primary disabled" v-else>
                Enviar
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import axios from "axios";
import Swal from "sweetalert2";

export default {
  data() {
    return {
      excel: "",
      name: "",
      file: "",
    };
  },
  methods: {
    enviar() {
      Swal.fire({
        title: "¿Estasa seguro de quere enviar este archivo?",
        icon: "question",
        showCancelButton: true,
        cancelButtonText: "Cancelar",
        cancelButtonColor: "#d33",
        confirmButtonText: "Confirmar",
        showLoaderOnConfirm: true,
        preConfirm: () => {
          let data = {
            name: this.name,
            // name: "dv_20200825_test01",
            content: this.excel,
          };

          return axios
            .post(
              "https://lxud3te6vg.execute-api.us-east-1.amazonaws.com/v1/file-base64",
              data
            )
            .then((res) => {
              this.excel = "";
              this.file = "";
              this.name = "";
              document.getElementById("fileInputId").value = null;
              Swal.fire({
                icon: "success",
                title: "¡Éxito!",
                text:
                  "Se ha enviao correctamente el archivo." + res.data.body.name,
              });
            })
            .catch((err) => {
              Swal.fire({
                icon: "error",
                title: "Oops...",
                text: err.response.data.err,
              });
            });
        },
        allowOutsideClick: () => !Swal.isLoading(),
      });
    },
    async handleFileSelect(evt) {
      const toBase64 = (file) =>
        new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.readAsDataURL(file);
          reader.onload = () => resolve(reader.result);
          reader.onerror = (error) => reject(error);
        });

      this.file = evt.target.files[0]; // FileList object
      let string = await toBase64(this.file);
      let desde = string.search("base64,");
      let palabra = string.substr(0, desde + 7);
      this.excel = string.replace(palabra, "");
      this.name = this.file.name;
      //google-chrome-stable --disable-web-security --user-data-dir="[some directory here]"
    },
  },
};
</script>

<style>
section {
  background-color: rgba(50, 146, 166, 0.8);
  color: white;
}
.full {
  height: 100vh;
}
.full > .col {
  background-color: #333333;
}
</style>
