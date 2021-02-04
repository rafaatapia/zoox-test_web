<template>
  <div>
    <v-data-table
      :headers="headers"
      :items="estados"
      class="elevation-1"
      hide-default-footer
      :loading="loading"
      loading-text="Buscando informações..."
    >
      <template #top>
        <v-toolbar flat>
          <v-toolbar-title>Estados</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template #activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                Cadastrar
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{
                  estado.id ? 'Alterar estado' : 'Cadastrar estado'
                }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        v-model="estado.nome"
                        label="Nome"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        v-model="estado.abreviacao"
                        hint="ex: PR"
                        label="Abreviação"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red darken-1" text @click="cancel">
                  Cancelar
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  :loading="loading"
                  :disabled="loading"
                  text
                  @click="createOrUpdateEstado()"
                >
                  {{ estado.id ? 'Alterar' : 'Cadastrar' }}
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="headline text-center"
                >Tem certeza que deseja remover o estado
                {{ estado.nome }}?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="cancel"
                  >Cancelar</v-btn
                >
                <v-btn color="red darken-1" text @click="deleteEstado()"
                  >SIM</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template #[`item.created_at`]="{ item }">
        {{ formatDate(item.created_at) }}
      </template>
      <template #[`item.updated_at`]="{ item }">
        {{ formatDate(item.updated_at) }}
      </template>
      <template #[`item.actions`]="{ item }">
        <v-icon color="blue" small class="mr-2" @click="openUpdateEstado(item)">
          mdi-pencil
        </v-icon>
        <v-icon color="red" small @click="openDeleteEstado(item)">
          mdi-delete
        </v-icon>
      </template>
    </v-data-table>
    <v-snackbar
      :color="sncakbarColor"
      :timeout="3000"
      :value="snackbar"
      absolute
      right
      shaped
      bottom
    >
      {{ snackbarText }}.
    </v-snackbar>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loading: false,
      estados: [],
      snackbar: false,
      sncakbarColor: '',
      snackbarText: '',
      dialog: false,
      dialogDelete: false,
      headers: [
        {
          text: 'Nome',
          sortable: false,
          value: 'nome',
        },
        { text: 'Abreviação', value: 'abreviacao' },
        { text: 'Criado', value: 'created_at' },
        { text: 'Atualizado', value: 'updated_at' },
        { text: 'Ações', value: 'actions', sortable: false },
      ],
      estado: {
        nome: '',
        abreviacao: '',
      },
    };
  },
  created() {
    this.fetchEstados();
  },
  methods: {
    async fetchEstados() {
      this.loading = true;
      try {
        const data = await this.$axios.$get('estados');
        this.estados = data;
      } catch (err) {
        let msg =
          'Houve uma falha ao carregar os registros. Por favor, tente novamente';
        if (err.response?.data?.message) {
          msg = err.response.data.message;
        }
        this.showSnack(msg, 'red');
      }
      this.loading = false;
    },
    async updateEstado() {
      this.loading = true;
      try {
        await this.$axios.$put(`estados/${this.estado.id}`, {
          abreviacao: this.estado.abreviacao,
          nome: this.estado.nome,
        });
        this.fetchEstados();
        this.dialog = false;
        this.estado = {};
        this.showSnack('Estado alterado com sucesso', 'green');
      } catch (err) {
        let msg =
          'Houve uma falha ao alterar o estado. Por favor, tente novamente';
        if (err.response?.data?.message) {
          msg = err.response.data.message;
        }
        this.showSnack(msg, 'red');
      }
      this.loading = false;
    },
    async createEstado() {
      this.loading = true;
      try {
        await this.$axios.$post('estados', this.estado);
        this.fetchEstados();
        this.dialog = false;
        this.estado = {};
        this.showSnack('Estado cadastrado com sucesso', 'green');
      } catch (err) {
        let msg =
          'Houve uma falha ao cadastrar o estado. Por favor, tente novamente';
        if (err.response?.data?.message) {
          msg = err.response.data.message;
        }
        this.showSnack(msg, 'red');
      }
      this.loading = false;
    },
    async deleteEstado() {
      this.loading = true;
      try {
        await this.$axios.$delete(`estados/${this.estado.id}`);
        this.fetchEstados();
        this.dialogDelete = false;
        this.estado = {};
        this.showSnack('Estado removido com sucesso', 'green');
      } catch (err) {
        let msg =
          'Houve uma falha ao remover o estado. Por favor, tente novamente';
        if (err.response?.data?.message) {
          msg = err.response.data.message;
        }
        this.showSnack(msg, 'red');
      }
      this.loading = false;
    },
    showSnack(text, color) {
      this.snackbarText = text;
      this.sncakbarColor = color;
      this.snackbar = true;
    },
    createOrUpdateEstado() {
      if (this.estado.id) {
        this.updateEstado();
      } else {
        this.createEstado();
      }
    },
    openUpdateEstado(estado) {
      this.estado = estado;
      this.dialog = true;
    },
    openDeleteEstado(estado) {
      this.estado = estado;
      this.dialogDelete = true;
    },
    formatDate(date) {
      const data = new Date(date);
      return ` ${data.toLocaleDateString()}  ${data.toLocaleTimeString()}`;
    },
    cancel() {
      this.estado = {};
      this.dialog = false;
      this.dialogDelete = false;
    },
  },
};
</script>

<style></style>
