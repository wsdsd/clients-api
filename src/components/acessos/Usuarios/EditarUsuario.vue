<template>
  <v-container
    id="user-profile"
    fluid
    tag="section"
  >

    <v-dialog
      v-model="submited"
      max-width="500"
    >
      <v-card>
        <v-card-title>
          <div class="success rounded">
            {{ $t('Cliente') }} {{ $t('editado_sucesso') }}
          </div>


          <v-spacer/>

          <v-icon
            aria-label="Close"
            @click="submited = false"
          >
            mdi-close
          </v-icon>
        </v-card-title>

        <v-card-text class="pb-6 pt-12 text-center">
          <v-card-subtitle>{{ $t('fazer_mais') }}</v-card-subtitle>

          <v-btn
            class="mr-3"
            text
            to="/usuarios"
          >
            {{ $t('nao') }}
          </v-btn>

          <v-btn
            color="success"
            text
            @click="submited = false; this.getRecurso"
          >
            {{ $t('sim') }}
          </v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>

    <div v-for="error in errors">
      <base-material-alert
        color="error"
        dark
        dismissible
      >
        <span>{{ error.fieldName }}</span> — {{ error.message }}"
      </base-material-alert>
    </div>


    <v-row justify="center">
      <v-col
        cols="12"
        md="12"
      >

        <v-form @submit.prevent="submitForm" v-if="!submited">
          <v-container class="py-0">
            <v-row>
              <v-col cols="12">
                <base-material-card
                  icon="mdi-clipboard-text"
                  :title="$t('Editar')"
                  class="px-5 py-3"
                >
                  <v-row>
                    <v-col
                      cols="12"
                    >
                      <v-text-field
                        label="Email"
                        class="purple-input"
                        v-model="recurso.email"
                      />
                    </v-col>

                    <v-col cols="12">
                      <v-combobox
                        v-model="perfisSelecionados"
                        :items="todosPerfisDescricao"
                        :label="$t('Perfis_do_usuario')"
                        multiple
                        chips
                      ></v-combobox>
                    </v-col>

                  </v-row>
                </base-material-card>
              </v-col>


              <v-col
                cols="12"
                class="text-right"
              >
                <v-btn
                  color="success"
                  class="mr-0"
                  type="submit"
                >
                  {{ $t('Enviar') }}
                </v-btn>
              </v-col>
            </v-row>
          </v-container>
        </v-form>

      </v-col>
    </v-row>
  </v-container>
</template>

<script>

import UsuarioDataService from "@/services/acessos/UsuarioDataService";
const DataService = UsuarioDataService

export default {
  name: 'Cadastro',
  data() {
    return {
      submited: false,
      recurso: {
        email: '',
      },
      errors: [],
      todosPerfisDescricao: [],
      perfisSelecionados:[],
      todosPerfis: [],
    }
  },
  methods: {
    submitForm(event) {
      this.ConvertePerfis();
      DataService.update(this.recurso)
        .then((response) => {
          this.errors = [];
          this.submited = true
        })
        .catch(error => {
          console.log(error)
          this.errors = error.response.data.errors
          window.scrollTo({top: 0, left: 0, behavior: 'smooth'})

        })

    },

    getRecurso() {

      DataService.find(this.$route.params.id)
        .then((response)=>{
          this.recurso = response.data
          const perfis= response.data.perfis

          perfis.forEach((value)=> {
            this.perfisSelecionados.push(value.descricao)
          })

        })
        .catch(error => {
          window.scrollTo({top: 0, left: 0, behavior: 'smooth'})
          console.log(error)
          this.errors.push({field: 'ID', message: 'Usuário não encontrado'})
        })
    },

    findPerfis(){

      DataService.findPerfis().then((response)=>{
        //Pego so as descrições de cada permissão
        //Ao enviar form envia os obj perfil que tem descrição entre as selecionadas
        const perfis = response.data
        this.todosPerfis = response.data;

        perfis.forEach((value)=> {
          this.todosPerfisDescricao.push(value.descricao)
        })

      })
        .catch(error => {
          window.scrollTo({top: 0, left: 0, behavior: 'smooth'})
          console.log(error)

        })
    },

    ConvertePerfis(){

      this.todosPerfis.forEach((perfil => {
        this.todosPerfisDescricao.forEach((descricao) => {

          if (descricao === perfil.descricao){
            this.recurso.perfis.push(perfil)
          }
        })
      }))

    }
  },
  mounted() {
    this.getRecurso()
    this.findPerfis()
  }

}
</script>

<style scoped>

</style>
